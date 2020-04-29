在现代的开发流程中随处可见 Docker 的身影，Docker 提供了环境隔离、应用打包等功能让服务部署变得特别简单，本文将会浅析 Docker 背后所使用的技术，阅读完后，你可以搞清楚如下问题：

- 1. 容器与虚拟机之间的差别
- 2. Docker 资源隔离的原理
- 3. Docker 资源限制的原理
- 4. Docker 分层结构的原理

## **容器 vs 虚拟机**

虚拟机（VM）是计算机系统的仿真器，通过软件模拟具有完整硬件系统功能的、运行在一个完全隔离环境中的完整计算机系统，能提供物理计算机的功能。

虚拟机通过在当前的真实操作系统上通过 Hypervisor 技术进行虚拟机运行环境与体系的建立并通过该技术进行资源控制，一个性能较好的物理机通常可以承载多个虚拟机，每个虚拟机都会有自己操作系统，如图 1.1 所示。

![图1.1](https://user-gold-cdn.xitu.io/2020/4/28/171c14a1f5d79e55?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)	图 1.1

从图中可以看出，虚拟机提供了物理机硬件级别的操作系统隔离，这让不同虚拟机之间的隔离很彻底，但也需要消耗更多资源，而有时不需要这么彻底的隔离，而更希望不消耗那么多资源，此时就可以使用容器技术。

容器可以提供操作系统级别的进程隔离，以 Docker 为例，当我们运行 Docker 容器时，此时容器本身只是操作系统中的一个进程，只是利用操作系统提供的各种功能实现了进程间网络、空间、权限等隔离，让多个 Docker 容器进程相互不知道彼此的存在，如图 1.2 所示。

![图1.2](https://user-gold-cdn.xitu.io/2020/4/28/171c14a1f69dd3b0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)图 1.2

虚拟机技术与容器技术的最大区别在于：**多个虚拟机使用多个操作系统内核，而多个容器共享宿主机操作系统内核**。

## **Docker 资源隔离：Linux Namespace**

Linux Namespace（Linux 命名空间）是 Linux 内核（Kernel）提供的功能，它可以隔离一系列的系统资源，如 PID（进程 ID，Process ID）、User ID、Network、文件系统等。

如果你熟悉 Linux，你可能会联想到 linux 中的 chroot 命令，该命令允许将当前目录修改成根目录（即根目录 / 的挂载点切换了），相当于文件系统被隔离了，Namespace 也具有相似的功能，但更加强大。

目前 Linux 主要提供 6 中不同类型的 Namespace，如下表所示。

|  Namespace 类型   |            描述             | 系统调用 flags（标记） |
| :---------------: | :-------------------------: | :--------------------: |
|  Mount Namespace  |     隔离文件系统挂载点      |       CLONE_NEWS       |
|   UTS Namespace   | 隔离 HostName 与 DomianName |      CLONE_NEWUTS      |
|   IPC Namespace   |       隔离进程间通信        |      CLONE_NEWIPC      |
|   PID Namespace   |         隔离进程 ID         |      CLONE_NEWPID      |
| Network Namespace |          隔离网络           |      CLONE_NEWNET      |
|  User Namespace   |      隔离用户和用户组       |     CLONE_NEWUSER      |

以一个具体的例子来解释 Namespace 的作用，假设你有一台性能非常好的计算机，你向用户出售自己的计算机的资源，每个用户买到一个 ssh 实例，为了避免不同客户之间相互干扰，你可能会对不同用户进行权限限制，让用户只能访问自己 ssh 实例下的资源。

但有些操作需要 root 权限，而我们不能将 root 权限提供给用户，此时就可以使用 Namespae 了，通过 User Namespace 对 UID 进行隔离，具体而言，UID 为 x 的用户在该 Namespace 中具有 root 权限，但在真实物理机中，他依旧是 UID 为 x 的用户，这就解决了用户间隔离的问题。

此外还可以通过 PID Namespace 对 PID 进行隔离，从该 Namespace 中的用户角度看，Namespace 中就像一台新的 Linux，有自己的 init 进程（初始进程，PID 为 1），其他进程的 PID 在 init 进程 PID 上递增，如图 1.3 所示。

![图1.3](https://user-gold-cdn.xitu.io/2020/4/28/171c14a1f68ad3c4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)图 1.3

图中，进程 3 在父命名空就中 PID 为 3，而在子命名空间中，其 PID 为 1，用户在该子命名空间中内看进程 3 就像 init 进程一样。

Linux 提供了 3 个系统 API 方便我们使用 Namespace：

- clone () 创建新进程，根据系统调用 flags 来决定哪种类型 Namespace 将会被创建，而该进程的子进程也会包含这些 Namespace。
- setns () 将进程加入到已存在的 Namespace 中。
- unshare () 将进程移出某个 Namespace

**Docker 利用 Linux Namespace 功能实现多个 Docker 容器相互隔离，具有独立环境的功能**，Go 语言对 Namespce API 进行了相应的封装，从 Docker 源码中可以看到相关的实现。

![img](https://user-gold-cdn.xitu.io/2020/4/28/171c14a210c7be7e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## **Docker 资源限制：Linux Cgroups**

Docker 通过 Linux Namespace 帮进程隔离出自己单独的空间 / 资源，那 Docker 如何限制进程对这些资源的使用呢？

Docker 容器本质依旧是一个进程，多个 Docker 容器运行时，如果其中一个 Docker 进程占用大量 CPU 和内存就会导致其他 Docker 进程响应缓慢，为了避免这种情况，可以通过 Linux Cgroups 技术对资源进行限制。

Linux Cgroups（Linux Contorl Groups，简称 Cgroups）可以对一组进程及这些进程的子进程进行资源限制、控制和统计的能力，其中包括 CPU、内存、存储、网络、设备访问权限等，通过 Cgroups 可以很轻松的限制某个进程的资源占用并且统计该进程的实时使用情况。

Cgroups 由 3 个组件构成，分别是 cgroup（控制组）、subsystem（子系统）以及 hierarchy（层级树），3 者相互协同作用。

- cgroup 是对进程分组管理的一种机制，一个 cgroup 通常包含一组（多个）进程，Cgroups 中的资源控制都以 cgroup 为单位实现。
- subsystem 是一组（多个）资源控制的模块，每个 subsystem 会管理到某个 cgroup 上，对该 cgroup 中的进程做出相应的限制和控制。
- hierarchy 会将一组（多个）cgroup 构建成一个树状结构，Cgropus 可以利用该结构实现继承等功能

3 者具体如何相互协同作用？

Cgroups 会将系统进程分组（cgroup）然后通过 hierachy 构建成独立的树，树的节点就是 cgroup（进程组），每颗树都可以与一个或多个 subsystem 关联，subsystem 会对树中对应的组进行操作。

有个几个规则需要注意。

\1. 一个 subsystem 只能附加到一个 hierarchy，而一个 hierarchy 可以附加多个 subsystem 2. 一个进程可以作为多个 cgroup 的成员，但这些 cgroup 只能在不同的 hierarchy 中 3. 一个进程 fork 出子进程，此时子进程与父进程默认是在同一个 cgroup 中，可以根据需要移动到其他 cgroup

![img](https://user-gold-cdn.xitu.io/2020/4/28/171c14a1fafe5667?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## **Docker 分层结构：Union File System**

我们都知道 Docker 镜像是一种分层结构，每一层构建在其他层之上，从而实现增量增加内容的功能，这是如何实现的？

要理解这个问题，首先需要理解 Union File System（简称，UnionFS），它是为 Linux 系统设计的将其他文件系统联合到一个联合挂载点的文件系统服务。UnionFS 使用 branch（分支）将不同文件系统的文件和目录透明地叠加覆盖，形成一个单一一致的文件系统，此外 UnionFS 使用写时复制（Copy on Write，简称，CoW）技术来提高合并后文件系统的资源利用。（后续的文章会介绍 CoW 技术）

Docker 使用的第一种存储驱动为 AUFS（Advanced Multi-layered unification filesytem），AUFS 完全重写了早期的 UnionFS，目的是提高其性能与可靠性，此外还引入了如 branch 负载均衡等新功能。

与 UnionFS 类似，AUFS 可以在基础的文件系统上增量的增加新的文件系统，通过叠加覆盖的形式最终形成一个文件系统。通常 AUFS 最上层是可读可写层，而其他层只是只读层，每一层都只是一个普通的文件系统。

Docker 镜像分层、增量增加等功能正是通过利用 AUFS 的分层文件系统结构、增量增加等功能实现，这也导致了运行 Docker 容器如果没有指定 volume（数据卷）或 bind mount，则 Docker 容器结束后，运行时产生的数据便丢失了。

Docker 存储驱动除了 AUFS 外，还有 OverlayFS、Devicemapper、Btrfs、ZFS 等，本文不多讨论。

## **总结**

至此，我们知道了 Docker 核心功能的基本原理，Docker 利用 Linux Namespace 进行网络、用户、进程等不同资源的隔离，使用 Linux Cgroups 技术对资源的使用进行限制与监控，通过 AUFS 等存储驱动实现分层结构与增量更新等功能。

现实世界中的 Docker 还使用了很多其他技术，但最核心且最基本的就是 Linux Namespace、Linux Cgrpus 与 AUFS。

Docker 在当前的开发流程中已成必备工具，容器带来的优势解放了运维人员也避免了开发人员遇到开发环境与线上环境不一致时导致问题的情况。目前容器编排技术（K8s）快速发展，Docker 容器技术在未来也将会进一步发展，它值得我们花时间与精力去了解其本质。
