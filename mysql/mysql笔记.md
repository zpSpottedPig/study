## MySql笔记

### 目录

+ [简介](#简介)
+ [MySQL安装](#MySQL安装)
+ [MySQL基础](#MySQL基础)
  + [数据类型](#数据类型)
+ [MySQL命令](#MySQL命令)
  + [用户管理](#用户管理)
  + [数据库管理](#数据库管理)
  + [表管理](#表管理)
  + [desc](#desc)
  + [use](#use)
  + [select](#select)
  + [explain](#explain)
  + [show](#show)
+ [Mysql函数](#Mysql函数)
  + [字符串函数](#字符串函数)
  + [日期函数](日期函数)
  + [常用](#常用)

### 简介

**SQL语言分类**

* DQL（Data Query Language）：数据查询语言 select 相关语句

* DML（Data Manipulate Language）：数据操作语言 insert 、update、delete 语句

* DDL（Data Define Languge）：数据定义语言 create、drop、alter 语句
* DCL （Data Control Language）：数据控制语句 grant：允许用户访问数据库的权限。revoke：撤销用户访问数据库的权限。

* TCL（Transaction Control Language）：事务控制语言 set autocommit=0、start transaction、savepoint、commit、rollback

### MySql安装

window安装 ：		教程参考(<https://www.cnblogs.com/honeynan/p/12408119.html>)

```window
mysql服务启动
-- 查看MySQL版本
mysql -V
mysql --version

方式1：
-- 打开服务
services.msc

方式2：
-- 关闭服务
net stop mysql (mysql服务名称)
-- 启动服务
net start mysql
-- mysql连接
mysql -P3306 -uroot -proot
```

### MySQL基础

##### 数据类型

主要包括以下五大类：

+ 整数类型：bit( 0/1 )、tinyint、smallint、mediumint、int、bigint

+ 浮点数类型：float、double、decimal

+ 字符串类型：char、varchar、tinyblob、blob、mediumblob、longblob、tinytext、text、mediumtext、longtext

+ 日期类型：Date、DateTime、TimeStamp、Time、Year

###### 整数类型

| 类型                      | 字节数 | 有符号值范围                       | 无符号值范围         |
| ------------------------- | ------ | ---------------------------------- | -------------------- |
| tinyint[(n)] [unsigned]   | 1      | [-2<sup>7</sup>,2<sup>7</sup>-1]   | [0,2<sup>8</sup>-1]  |
| smallint[(n)] [unsigned]  | 2      | [-2<sup>15</sup>,2<sup>15</sup>-1] | [0,2<sup>16</sup>-1] |
| mediumint[(n)] [unsigned] | 3      | [-2<sup>23</sup>,2<sup>23</sup>-1] | [0,2<sup>24</sup>-1] |
| int[(n)] [unsigned]       | 4      | [-2<sup>31</sup>,2<sup>31</sup>-1] | [0,2<sup>32</sup>-1] |
| bigint[(n)] [unsigned]    | 8      | [-2<sup>63</sup>,2<sup>63</sup>-1] | [0,2<sup>64</sup>-1] |

注：1. []包含的内容是可选的，默认是有符号类型的，无符号的需要在类型后面跟上unsigned。

​		2. n表示的是显示宽度，不足的用0补足，超过的无视长度而直接显示整个数字，但这要整型设置了unsigned zerofill才有效

​		3.无符号类型的，插入了负数会报错。

​		4. int(5)输出宽度不满5时，前面用0来进行填充，int(n)中的n省略的时候，**宽度为对应类型无符号最大值的十进制的长度**

​		

### MySQL命令

##### 用户管理

```mysql
use mysql; -- 切换数据库
select user,host,authentication_string from user; -- 查看用户列表
-- 创建用户 语法：create user 用户名[@主机名] [identified by '密码'];
create user test5; -- 无密码的用户，可以从任何机器登录到mysql中
create user 'test2'@'localhost' identified by '123'; -- 只能登陆本机的mysql
create user 'test3'@% identified by '123'; -- 可以从任何机器连接到mysql服务器
create user 'test4'@'192.168.11.%' identified by '123'; -- 可以从192.168.11段的机器连接mysql

-- 修改密码 注：通过表的方式修改之后，需要执行flush privileges; 刷新权限信息 ，才能对用户生效。
SET PASSWORD FOR 'test1'@'%' = PASSWORD('123456');
update user set authentication_string = password('321') where user = 'test5' and host = '%';
ALTER USER 'test2'@'localhost' IDENTIFIED BY '123456';

-- 给用户授权  语法：grant privileges ON database.table TO 'username'[@'host'] [with grant option]
-- grant命令说明：
--     priveleges (权限列表)，可以是all，表示所有权限，也可以是select、update等权限，多个权限之间用逗号分开。
--     ON 用来指定权限针对哪些库和表，格式为数据库.表名 ，点号前面用来指定数据库名，点号后面用来指定表名，*.* 表示所有数据库所有表。
--     TO 表示将权限赋予某个用户, 格式为username@host，@前面为用户名，@后面接限制的主机，可以是IP、IP段、域名以及%，%表示任何地方。
--     WITH GRANT OPTION 这个选项表示该用户可以将自己拥有的权限授权给别人。注意：经常有人在创建操作用户的时候不指定WITH GRANT OPTION选项导致后来该用户不能使用GRANT命令创建用户或者给其它用户授权。 备注：可以使用GRANT重复给用户添加权限，权限叠加，比如你先给用户添加一个select权限，然后又给用户添加一个insert权限，那么该用户就同时拥有了select和insert权限。

grant all on *.* to 'test1'@‘%’; -- 给test1授权可以操作所有库所有权限，相当于dba
grant select on seata.* to 'test1'@'%'; -- test1可以对seata库中所有的表执行select
grant select,update on seata.* to 'test1'@'%'; 
-- test1可以对seata库中所有的表执行select、update
grant select(user,host) on mysql.user to 'test1'@'localhost';
-- test1用户只能查询mysql.user表的user,host字段

-- 查看权限
show grants for 'test1'@'localhost';
show grants; -- 查看当前用户的权限

-- 撤销用户的权限 语法: revoke privileges ON database.table FROM '用户名'[@'主机'];
revoke select(host) on mysql.user from test1@localhost;

-- 删除用户
drop user 'test1'@'%';
delete from user where user='test2' and host='localhost';
```

##### 数据库管理

```mysql
create database database_name; -- 创建数据库
drop database if exists database_name; -- 删除数据库
```

##### 表管理

```mysql
-- 创建表
create table test11(
  id int not null AUTO_INCREMENT  comment '字段id',
  b int not null default 0 comment '字段b',
  PRIMARY KEY (`id`),
  unique key(b)
 ) ENGINE=InnoDB AUTO_INCREMENT=2729 DEFAULT CHARSET=utf8 COMMENT='test）';
 
primary key : 该表的主键
AUTO_INCREMENT : 自增
unique key(b) ：唯一标识
ENGINE=InnoDB : 使用innodb引擎
DEFAULT CHARSET=utf8 : 数据库默认编码为utf-8
AUTO_INCREMENT=2729 主键起始值

-- 删除表
drop table if exists table_name;
-- 修改表名
alter table table1 rename table2;
-- 表设置备注
alter table table1 comment 'table11';
-- 复制表
create table test12 like test11;
create table test13 as select * from test11;

-- 添加列
alter table test14 
add column b int not null default 0 comment '字段b';

-- 修改列 modify不能修改列名，change可以修改列名
alter table test14 
modify column c  varchar(10) not null default '' comment '字段cc';

alter table test14 
change column c d varchar(10) not null default '' comment '字段d';

-- 删除列
alter table test14 drop column d;

-- drop，truncate，delete区别
drop 删除表
truncate 清空表
delete 删除表数据
```

##### desc

```mysql
-- 查看表结构：
desc table_name;
```

##### use

```mysql
-- 切换数据库
use database_name;
```

##### select

```mysql
-- 查看版本
select version();
-- 查看当前所在库
select database();
```

##### explain

解释select 查询语句 参考：<https://blog.csdn.net/why15732625998/article/details/80388236>

结果集：id、select_type、table、type、possible_keys、key、key_len、ref、rows、Extra



##### show

```mysql
-- 查看所有数据库
show databases;
-- 查看数据库表
show tables;
show tables from database_name;
-- 查看表的创建语句
show create table biz_tags;
-- 查看当前mysql支持的存储引擎
SHOW ENGINES;
-- 查看系统变量及其值
SHOW VARIABLES;
SHOW VARIABLES like 'wait_timeout';
--  查看所有事件
show events; 
-- 查看事件状态
-- 开启事件：SET GLOBAL event_scheduler = 1;SET GLOBAL event_scheduler = 'no';
show variables like 'event_scheduler';  
-- 查看所有过程
show procedure status;
--  查看存储过程或函数的创建代码
show create procedure proc_name;
--  查看用户权限
show grants for 'username'; 
```

#### Mysql函数

##### 字符串函数

```mysql
LEFT(str, length)  -- 从左截取length个字符
RIGTH(str, length) -- 从右截取length个字符
LENGTH(str) -- 字符串字节数
CHAR_LENGTH(str) -- 字符串长度
SUBSTR(str,n) -- 从第n个字符开始往后截取所有字符
SUBSTR(str,n,len) -- 从第n个字符开始往后截取len个字符
CONCAT(str1,str2) -- 合并字符串
REPLACE(Str,char1,char) -- 替换
GROUP_CONCAT(str) -- ',' 连接合并字符串
trim(str) -- 去除空格
locate('aa','aaba'); -- 校验包含 0不包含 >0 包含
```

##### 日期函数

```mysql
NOW()  -- 当前时间
CURDATE() -- 当前日期
-- 相关枚举  CURRENT_DATE
YEAR(date) -- 获取年
MONTH(date) -- 获取月
DAY(date) -- 获取日
LAST_DAY(date) -- 日期月份的最后一天日期
TIMESTAMPDIFF(HOUR,t1.create_time,t2.create_time) -- 两日期小时差值
-- 单位：
-- MICROSECOND  微秒 
-- SECOND  秒
-- MINUTE  分钟
-- HOUR  小时
-- DAY  天
-- WEEK  周
-- MONTH  月份
-- QUARTER 季度
-- YEAR  年份
DATE_ADD(date, INTERVAL 5 DAY) -- 日期加
DATE_SUB(date, INTERVAL 5 DAY) -- 日期减
DATE_FORMAT(date,formatStr) -- riq
```

##### 常用

```mysql
IFNULL(expre,value) 
ROUND(num,n) -- 四舍五入
```



