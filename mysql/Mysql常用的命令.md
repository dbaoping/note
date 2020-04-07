# 启动mysql服务
/usr/local/mysql/bin/mysqld_safe --user=mysql &
# 用root用户登录
/usr/local/mysql/bin/mysql -u root -proot
# 显示所有数据库
show databases;
# 选择数据库
use <数据库名>；
# 显示所有的表
show tables;
# 显示表结构
desc <表名>
# 查看系统变量及其值
show variables;
# 查看某一个系统变量值  支持模糊查询  例如'slow_%'
show variables like '<变量名>';
# 设置系统变量的值
set global <变量名>=<值>
# 查看查询的执行计划
explain <查询的sql句>

SHOW TABLES FROM mysql LIKE 'help%';


