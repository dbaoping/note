# MySQL 索引及查询优化

## 一、简单的测试对比

### 1. 创建一张表

```sql
CREATE TABLE `user` (
  `name` varchar(255) NOT NULL,
  `id` int(16) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 2. 插入数据

```sql
insert into user VALUE ("张三", 1);
```

### 3. 测试

#### 未使用索引

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdeb613847b74ced9274913cac95855b~tplv-k3u1fbpfcp-watermark.image)

在上图中，type=ALL，key=Null，rows=11982。该sql未使用索引，是一个效率非常低的全表扫描。如果加上联合查询和其他一些约束条件，数据库会疯狂的消耗内存，并且会影响前端程序的执行。

#### 使用索引

**添加索引**

```
ALTER TABLE user ADD INDEX index_title (name);
```

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa346ede2a1d4f8b916cc9e3a8b7a414~tplv-k3u1fbpfcp-watermark.image)

在上图中，type=ref，key=索引名（index_title），rows=1。该sql使用了索引index_title，且是一个常数扫描，根据索引只扫描了一行。

加了索引后，查询效率明显提升。

## 二、索引

通过上面的对比测试可以看出，索引是快速搜索的关键。MySQL索引的建立对于MySQL的高效运行是很重要的。对于少量的数据，没有合适的索引影响不是很大，但是，当随着数据量的增加，性能会急剧下降。如果对多列进行索引(组合索引)，列的顺序非常重要，MySQL仅能对索引最左边的前缀进行有效的查找。

索引就好比一本书的目录，它会让你更快的找到内容，目录（索引）并不是越多越好，假如这本书1000页，有500是目录，它效率就会变低，目录是要占纸张的,而索引是要占磁盘空间的。

**索引是一种数据结构，可以快速访问数据库表提取信息。**

### 常见索引

#### (1) 主键索引 PRIMARY KEY

> 它是一种特殊的唯一索引，不允许有空值。一般是在建表的时候同时创建主键索引。
>
> ![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1ab4d15c6424e548e3cf07d9a797da1~tplv-k3u1fbpfcp-watermark.image)
>
> 当然也可以用 ALTER 命令。

**记住：一个表只能有一个主键。**

#### (2) 唯一索引 UNIQUE

> 唯一索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。可以在创建表的时候指定，也可以修改表结构，如：
>
> ```sql
> ALTER TABLE table_name ADD UNIQUE (column)
> ```

#### (3) 普通索引 INDEX

> 这是最基本的索引，它没有任何限制。可以在创建表的时候指定，也可以修改表结构，如：
>
> ```sql
> ALTER TABLE table_name ADD INDEX index_name (column)
> ```

#### (4) 组合索引 INDEX

> 组合索引，即一个索引包含多个列。可以在创建表的时候指定，也可以修改表结构，如：
>
> ```sql
> ALTER TABLE table_name ADD INDEX index_name(column1, column2, column3)
> ```

#### (5) 全文索引 FULLTEXT

> 全文索引（也称全文检索）是目前搜索引擎使用的一种关键技术。它能够利用分词技术等多种算法智能分析出文本文字中关键字词的频率及重要性，然后按照一定的算法规则智能地筛选出我们想要的搜索结果。
>
> 可以在创建表的时候指定，也可以修改表结构，如：
>
> ```sql
> ALTER TABLE table_name ADD FULLTEXT (column)
> ```

## 三、索引原理

### 二叉排序树

对于一个节点，它的左子树的孩子节点值都要小于它本身，它的右子树的孩子节点值都要大于它本身，如果所有节点都满足这个条件，那么它就是二叉排序树。

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/896a2646fb454d41868bbdfc3b36fa76~tplv-k3u1fbpfcp-watermark.image)

```
上图是一颗二叉排序树，你可以尝试利用它的特点，体验查找9的过程（值相等）：

9比10小，去它的左子树（节点3）查找
9比3大，去节点3的右子树（节点4）查找
9比4大，去节点4的右子树（节点9）查找
节点9与9相等，查找成功
一共比较了4次，那你有没有想过上述结构可以优化？
```

### AVL（自平衡二叉查找树）

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdf46fd061b74432b472816fa0331c7f~tplv-k3u1fbpfcp-watermark.image)

上图是AVL树，节点个数和值均和二叉排序树一摸一样

```
再来看一下查找9的过程：

9比4大，去它的右子树查找
9比10小，去它的左子树查找
节点9与9相等，查找成功

一共比较了3次，同样的数据量比二叉排序树少了一次，为什么呢？因为AVL树高度要比二叉排序树小，
高度越高意味着比较的次数越多；不要小看优化的这一次，假如是海量数据，比较次数会明显地增长。
```

### B树（Balanced Tree）多路平衡查找树

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fddbaad1fa0047859ec2cc9c3e9f1a07~tplv-k3u1fbpfcp-watermark.image)

```
再来看一下查找9的过程：

9比8大，去它的右子树查找
9与节点中的9相等，查找成功

一共比较了2次；你可能会说，这才到B树就已经提升了那么多，为什么还要引入B+树？再来详细解释一下B树，
B树的节点中存储了可以取出关键值的指针，每次取出关键值比较都要进行IO操作，即从磁盘中取出值和查找值匹配，
查找到9我们比较了2次，就要进行2次IO操作；
```

### B+树

B+树的非叶子节点并不直接存储可以从磁盘中取出关键值的指针，而是存储关键值的索引，关键值只存储在叶子节点中，并且叶子节点组成了一个有序链表。这可能比较难理解。

结合MySql数据库来说一下，图中非叶子节点暂且理解成数据的索引，同样来看查找索引为9的过程：

> 下图的非叶子节点虽然存储的依然是关键值，但此处可以理解为关键值的id，因为关键值不一定是单个数据0009，更多场景下，数据包含多种属性：id，name等等。

查找id为9的关键值数据，id一次比较，查找成功；由于节点并不存储关键值的数据指针，所以本次比较不需要IO操作，顺着指针找到叶子节点，并根据叶子节点的数据指针进行一次IO操作，取出完整数据，查找完成。

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/483ab4c728364a9ca673c080ebe26710~tplv-k3u1fbpfcp-watermark.image)

**B+树的优点：**

比较的次数均衡，减少了IO次数，提高了查找速度，查找也更稳定。

1. B+树的磁盘读写代价更低
2. B+树的查询效率更加稳定

以上是索引的数据结构演化，下面我们再来讲一些关键概念。

要知道的是，你每次创建表，系统会为你自动创建一个基于ID的聚集索引（上述B+树），存储全部数据；你每次增加索引，数据库就会为你创建一个附加索引（上述B+树），索引选取的字段个数就是每个节点存储数据索引的个数，注意该索引并不存储全部数据。

### 回表

比如你创建了name, age索引 ng_index，查询数据时使用了

```sql
select * from table where name ='bill' and age = 21;
```

由于附加索引中只有name 和 age，因此命中索引后，数据库还必须回去聚集索引中查找其他数据，这就是回表，这也是你背的那条：少用select * 的原因。

### 索引覆盖

结合回表会更好理解，比如上述ng_index索引，有查询

```sql
select name, age from table where name ='bill' and age = 21;
```

此时select的字段name，age在索引ng_index中都能获取到，所以不需要回表，满足索引覆盖，效率较高。

### 最左匹配

B+树的节点存储索引顺序是从左向右存储，在匹配的时候自然也要满足从左向右匹配；

比如索引ng_index，下列sql都可以命中ng_index；

```sql
select name from table where name = 'bill';
select name, age from table where name = 'bill' and age = 18；
```

### 索引下推

还是索引ng_index，有如下sql

```sql
select * from table where name like 'B%' and age > 20
```

该语句有两种执行可能：

1、命中ng_index联合索引，查询所有满足name以B开头的数据， 然后回表查询所有满足的行。 

2、命中ng_index联合索引，查询所有满足name以B开头的数据，然后直接筛出age>20的索引，再回表查询全行数据，

 显然第2种方式回表查询的行数较少，IO次数也会减少，这就是索引下推。所以不是所有like都不会命中索引。

## 四、建索引的几大原则

（1） 最左匹配原则

> 对于多列索引，总是从索引的最前面字段开始，接着往后，中间不能跳过。比如创建了多列索引(name,age,sex)，会先匹配name字段，再匹配age字段，再匹配sex字段的，中间不能跳过。mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配。
>
> 一般，在创建多列索引时，where子句中使用最频繁的一列放在最左边。
>
> ```
> 举例：
> ```
>
> 表user，建立索引（name, sex）
>
> **不符合最左前缀匹配原则的sql语句：**
>
> ```sql
> EXPLAIN select * from user where sex = 0;
> ```
> 
>![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7dba32beffe4f07830a4b5dee991fb9~tplv-k3u1fbpfcp-watermark.image)
> 
>该查询跳过了第一个索引name，直接根据第二个索引sex查询，不符合最左前缀匹配原则
> 
>未使用索引，是一个低效的全表扫描。
> 
>**符合最左前缀匹配原则的sql语句：**
> 
>```
> EXPLAIN select * from user where name = "1" and sex = 0;
> 复制代码
> ```
> 
>![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ceff8108452f4c38a511d78f3361d1ef~tplv-k3u1fbpfcp-watermark.image)
> 
>该sql先使用了索引的第一个字段name，再使用索引的第二个字段sex，中间没有跳过，符合最左前缀匹配原则。
> 
>该sql使用了索引，仅扫描了一行。
> 
>对比可知，符合最左前缀匹配原则的sql语句比不符合该原则的sql语句效率有极大提高，从全表扫描上升到了常数扫描。

（2） 尽量选择区分度高的列作为索引

> 比如，我们会选择身份证号做索引，而不会选择年龄来做索引。

（3） =和in可以乱序

> 比如a = 1 and b = 2 and c = 3，建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。

（4） 索引列不能参与计算，保持列“干净”

> 比如：age + 1 > 20。原因很简单，假如索引列参与计算的话，那每次检索时，都会先将索引计算一次，再做比较，显然成本太大。

（5） 尽量的扩展索引，不要新建索引

> 比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

## 五、索引的缺点

虽然索引可以提高查询效率，但索引也有自己的不足之处。

索引的额外开销：

(1) 空间：索引需要占用空间；

(2) 时间：查询索引需要时间；

(3) 维护：索引须要维护（数据变更时）；

不建议使用索引的情况：

(1) 数据量很小的表

(2) 空间紧张

## 六、SQL优化

### 1、有索引但未被用到的情况（不建议）

##### (1) Like的参数以通配符开头时

尽量避免Like的参数以通配符开头，否则数据库引擎会放弃使用索引而进行全表扫描。

以通配符开头的sql语句，例如：

```sql
select * from where name like '%1';
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44dc5a37df9241ab93eb6f5c41c234bf~tplv-k3u1fbpfcp-watermark.image)

这是全表扫描，没有使用到索引，不建议使用。

不以通配符开头的sql语句，例如：

```sql
select * from user where name like '1%'
```
![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1095059d1a8343738da213bfbcaa3462~tplv-k3u1fbpfcp-watermark.image)

很明显，这使用到了索引，是有范围的查找了，比以通配符开头的sql语句效率提高不少。

##### (2) where条件不符合最左前缀原则时

例子已在最左前缀匹配原则的内容中有举例。

##### (3) 使用！= 或 <> 操作符时

尽量避免使用！= 或 <>操作符，否则数据库引擎会放弃使用索引而进行全表扫描。使用`>`或`<`会比较高效。

```sql
select * from user where id != '1';
```

##### (4) 索引列参与计算

应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。

```sql
select * from user where id +1 > '1';
```

##### (5) 对字段进行null值判断

应尽量避免在where子句中对字段进行null值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：

低效：

```sql
select * from user where sex is null ;
```

可以在`sex`上设置默认值0，确保表中`sex`列没有null值，然后这样查询：

高效：

```sql
select * from user where sex = 0;
```

##### (6) 使用or来连接条件

应尽量避免在where子句中使用or来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：

低效：

```sql
select * from user where name = '1' or sex = '0';
```

可以用下面这样的查询代替上面的or查询：

高效：

```sql
select from user where name = '1' union all select from user where name = '2';
```

#### 2、避免select *

在解析的过程中，会将`*`依次转换成所有的列名，这个工作是通过查询数据字典完成的，这意味着将耗费更多的时间。

所以，应该养成一个需要什么就取什么的好习惯。

#### 3、order by 语句优化

任何在`order by`语句的非索引项或者有计算表达式都将降低查询速度。

方法：

1. 重写order by语句以使用索引；
2. 为所使用的列建立另外一个索引
3. 绝对避免在order by子句中使用表达式。

#### 4、GROUP BY语句优化

提高`GROUP BY`语句的效率, 可以通过将不需要的记录在`GROUP BY`之前过滤掉

低效:

```sql
SELECT name, AVG(age) FROM user GROUP BY name HAVING name = '1' OR name = '2'
```

高效:

```sql
SELECT name, AVG(age) FROM user WHERE name = ‘1' OR name = ‘2' GROUP by name
```

#### 5、用 exists 代替 in

很多时候用`exists`代替`in`是一个好的选择：

```sql
select num from a where num in(select num from b)
```

用下面的语句替换：

```sql
select num from a where exists(select 1 from b where num=a.num)
```

#### 6、使用 varchar/nvarchar 代替 char/nchar

尽可能的使用 `varchar/nvarchar` 代替 `char/nchar` ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

#### 7、能用DISTINCT的就不用GROUP BY

```sql
SELECT order_id FROM Details WHERE price > 10 GROUP BY order_id
```

可改为：

```sql
SELECT DISTINCT order_id FROM Details WHERE price > 10
```

#### 8、能用UNION ALL就不要用UNION

UNION ALL不执行SELECT DISTINCT函数，这样就会减少很多不必要的资源。

#### 9、在Join表的时候使用相当类型的例，并将其索引

如果应用程序有很多JOIN 查询，你应该确认两个表中Join的字段是被建过索引的。这样，MySQL内部会启动为你优化Join的SQL语句的机制。

而且，这些被用来Join的字段，应该是相同的类型的。例如：如果你要把 DECIMAL 字段和一个 INT 字段Join在一起，MySQL就无法使用它们的索引。对于那些STRING类型，还需要有相同的字符集才行。（两个表的字符集有可能不一样）