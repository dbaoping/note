### binlog 和 redolog 区别
1. 适用对象不同：
    - bin log 是 MySQL 的 Server 层实现的，所有引擎都可以使用
    - redo log 是 InnoDB 引擎特有的
2. 写入内容不同：
    - bin log 是逻辑日志，记录的是这个语句的原始逻辑，比如 “给 id = 1 这一行的 age 字段加 1”
    - redo log 是物理日志，记录的是 “在某个数据页上做了什么修改”
3. 写入方式不同：
    - bin log 是可以追加写入的。“追加写” 是指 bin log 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志
    - redo log 是循环写的，空间固定会被用完

### 开启binlog
mysql配置添加
```
# 这个参数表示启用 binlog 功能，并指定 binlog 的存储目录 
log-bin=javaboy_logbin  
# 设置一个 binlog 文件的最大字节 设置最大 100MB 
max_binlog_size=104857600  
# 设置了 binlog 文件的有效期（单位：天） 
expire_logs_days = 7  
# binlog 日志只记录指定库的更新（配置主从复制的时候会用到） 
#binlog-do-db=javaboy_db  
# binlog 日志不记录指定库的更新（配置主从复制的时候会用到） 
#binlog-ignore-db=javaboy_no_db  
# 写缓存多少次，刷一次磁盘，默认 0 表示这个操作由操作系统根据自身负载自行决定多久写一次磁盘 
# 1 表示每一条事务提交都会立即写磁盘，n 则表示 n 个事务提交才会写磁盘 
sync_binlog=0  
# 为当前服务取一个唯一的 id（MySQL5.7 之后需要配置） 
server-id=1
# binlog 模式 ROW | MIXED | STATEMENT
binlog_format="MIXED"
```

### 常用命令
- 产看binlog配置
```sql
SHOW VARIABLES LIKE 'log_bin%'
```
- 查看binlog的模式
```sql
SHOW VARIABLES LIKE "binlog_format";
```
- 查看 binlog 日志列表：
```sql
show master logs;
```
Log_name | File_size
---|---
mysql-bin.002975 | 524288413
mysql-bin.002976 | 524288106
mysql-bin.002977 | 524289807
mysql-bin.002978 | 524288852
mysql-bin.002979 | 524288397
mysql-bin.002980 | 106655857
- 查看 master 状态
```sql
show master status
```
- 手动刷新 binlog
```sql
flush logs
```
手动刷新 binlog 之后，就会产生一个新的 binlog 日志文件，接下来所有的 binlog 日志都将记录到新的文件中

- 查看 binlog
```sql
show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];

show binlog events in 'mysql-bin.002980' LIMIT 20;
```

参数 | 说明
---|---
log_name | 可以指定要查看的 binlog 日志文件名，如果不指定的话，表示查看最早的 binlog 文件。
pos | 从哪个 pos 点开始查看，凡是 binlog 记录下来的操作都有一个 pos 点，这个其实就是相当于我们可以指定从哪个操作开始查看日志，如果不指定的话，就是从该 binlog 的开头开始查看。
offset | 这是是偏移量，不指定默认就是 0。
row_count | 查看多少行记录，不指定就是查看所有。
    
### 备份恢复

#### 数据库备份

```sql
mysqldump -uroot -p --flush-logs --lock-tables -B javaboy>/root/javaboy.bak.sql
```
- flush-logs：这个表示在导出之前先刷新 binlog，刷新 binlog 之后将会产生新的 binlog 文件，后续的操作都存在新的 binlog 中。
- lock-tables：这个表示开始导出前，锁定所有表。需要注意的是当导出多个数据库时，--lock-tables分别为每个数据库锁定表，因此这个选项不能保证导出文件中的表在数据库之间的逻辑一致性，不同数据库表的导出状态可以完全不同。
- B：这个表示指定导出的数据库名称，如果使用 --all-databases 或者 -A 代替 -B 表示导出所有的数据库。
 
#### binlog恢复
```
mysqlbinlog /var/lib/mysql/mysql-bin.002980 --stop-position=764 --database=javaboy | mysql -uroot -p
```
- stop-position=764 表示恢复到 764 这个 Pos，不指定的话就把按整个文件恢复了，如果按当前文件恢复的话，由于这个 binlog 文件中有删除数据库的语句，那么就会导致执行完该 binlog 之后，javaboy 库又被删除了。
- database=javaboy 表示恢复 javaboy 这个库。