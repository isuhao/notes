﻿# Oracle学习笔记

标签（空格分隔）： DataBase

---



##《Oracle入门很简单》

```sql
-- oracle中表的全名是用户名加表名？  test.sql_cookbook
-- 查看当前数据库中表空间及其物理文件的完整路径，其中 dba_data_files 为视图
SELECT tablespace_name,
       file_name
FROM dba_data_files
ORDER BY file_name;

-- 创建表空间
-- 该表空间初始大小为20M，每次自动增加5M，最大大小为500M。 数据文件的命名一般包含表空间的名字，一般识别。
CREATE TABLESPACE test datafile '/home/oracle/database/ORA11G/test.dbf' SIZE 20 M autoextend ON next 5 M maxsize 500 M;

-- 数据库用户及用户默认的表空间;
SELECT user_id,
       username,
       default_tablespace
FROM dba_users
ORDER BY user_id DESC;

SELECT *
FROM dba_users;

-- 更改数据库的默认表空间。原本的默认表空间为 users
ALTER DATABASE DEFAULT TABLESPACE test;

-- 重命名表空间(但文件名不变)
ALTER TABLESPACE test01 RENAME TO test_data;

-- 利用descibe命令查看已有数据表的表结构
DESC student;

/*
alter table 

add 用于添加列
alter table student add (class_id number);

modify 用于更改列数据类型
alter table student modify (class_id varchar2(20);

drop column 用于删除已有列
alter table student drop column class_id;

rename column ×× to ××× 用于重命名列

move tablespace tbs_name 转移表空间
alter  table student move tablespace users;
*/ /*
特殊的数据吧dual

dual表实际属于系统用户sys,具有数据库基本权限的用户，均可查询该表的内容。

在Oracle中，所有查询语句必须满足 select from 的格式，但某些时候，数据源并不明确。
例如：函数sysdate()用于返回当前日期时，可以使用dual表，用于提供数据源。


dual表提供了一行一列的数据格式，从而使各种表达式、函数运算在以其为数据源时，能够输出单行单列的形式。
*/ 
SELECT *
FROM dual;

SELECT *
FROM sys.dual;

-- 或者
DESC sys.dual;

SELECT SYSDATE
FROM dual;

SELECT 5 *4.5 +7 result
FROM dual;

/*
根据列名获得该列所出现的表名
很多时候需要从列追溯su表信息

在oracle中可以通过视图 user_tab_cols获得当前用户的所有列的信息。
*/ 
SELECT table_name
FROM user_tab_cols
WHERE LOWER(column_name) = 'id';

-- 必须要使用 lower()函数
/*
第4章： SQL查询

*/ 
-- 获取唯一记录： distinct
-- group by 分组
-- 过滤分组 having子句
-- 示例：获得工资总额大于10000的记录
SELECT e.employee_id,
       e.employee_name,
       SUM(s.salary) total_salary
FROM employees e,
     salary s
WHERE e.employee_id = s.employee_id
GROUP BY e.employee_id,
         e.employee_name
HAVING (SUM(s.salary)) > 10000;

/*
执行步骤如下：
1.利用from子句获得数据源
2.利用where子句筛选符合条件的记录
3.利用group by子句进行分组
4.利用having子筛选分组
5.对每组进行循环处理，获得最终结果。（select）
*/ 
-- 排序 order by子句
-- asc 默认排序方式； desc 降序
-- order by 与 group by子句
SELECT e.employe_name,
       SUM(s.salary) total_salary
FROM employee e,
     salary s
WHERE e.employee_id = s.employee_id
GROUP BY e.employee_name
ORDER BY total_salary DESC;

注意： 经过group by动作和聚合函数sum () ，数据源已不是表employees与表salary的笛卡尔积所获得的结果集。 因此不能使用employee_name和total_salary之外的列进行排序。;

-- order by与distinct
-- 当两者进行同时使用时，也必须遵循这样一个规则——order by子句所指定的排序列，必须出现在select表达式中。
-- 下面的语句发生错误
SELECT DISTINCT e.employe_name
FROM employee e,
     salary s
WHERE e.employee_id = s.employee_id
ORDER BY s.salary;

-- ORA-01791:not a selecteed expression，该错误表明order by并非select所指定的表达式。
-- 有了group by的实例，distinct 与order by一起使用时的错误便容易理解许多。
-- 对于ditinct动作，可以将其，视为将数据库按照select之后的列或表达式进行**分组**。

-- select distinct e.employee_name来说，它会将数据源按照e.employee_name列进行分组
-- Oracle的distinct操作实际是获得每个分组之内的任意employee_name。那么，当使用其他列，例如s.salary，进行
-- 排序时，数据库便没有统一的标准。因为一个分组内有多个s.salary，除非将s.salary也作为分组条件——即写入distinct列表中，
-- 才能保证分组内所有s.salary也是相同的。

/*
上面这一节有助于重新理解 distinct
*/

```



## 课程笔记

```sql
-- 解锁scott用户，其默认密码是tiger。但默认密码提示无效(expire)更改密码为2tiger
alter user scott account unlock;
-- 重设密码
alter user scott identified by 2tiger;



/*
  一些常用语句
*/
-- 查看有哪些表
select table_name from user_tables;

-- 数据字典中有 $ 的，代表它是动态信息。
select * from V$SESSION;

-- 查询所有表空间
select distinct TABLESPACE_NAME from dba_free_space;
-- 或（更好），其中 dba_data_files 为视图
select tablespace_name, file_name from dba_data_files order by file_name;

-- 数据库用户及用户默认的表空间;
select user_id, username, default_tablespace from dba_users order by user_id desc;
select * from dba_users;

-- 查看所有用户
select * from all_users;
select * from dba_users;

-- 查看该用户有那些表（与MySQL的不同）
select * from user_tables;






-- 默认存在的表空间： SYSTEM表空间，Non-SYSTEM表空间，临时表空间

-- 在创建用户之前先创建表空间
-- 尽量让每个用户使用自己的表空间，以免造成数据访问热点。
-- 这里我的表空间的数据文件存放在 /home/oracle/database/ORA11G/ 目录下，文件后缀名为.dbf
-- 默认位置应该为: /ora01/app/oracle/oradata/ORA11G/ 目录下。

-- 创建表空间
-- 该表空间初始大小为20M，每次自动增加5M，最大大小为500M。 数据文件的命名一般包含表空间的名字，一般识别。
create tablespace test02
datafile '/home/oracle/database/ORA11G/test02.dbf'
size 20M
autoextend on
next 5M
maxsize 500M;

-- 创建表空间
-- 该表空间大小20M，不进行自动扩展。
create tablespace test01
datafile '/home/oracle/database/ORA11G/test01.dbf'
size 20M
  AUTOEXTEND on;

-- 删除表空间，
-- 不能删除使用中的表空间
-- 同时删除表空间内容 和 所在的磁盘文件需添加后面的几句语句
drop tablespace test01 including contents and datafiles;


-- 如果用户创建表时没有显示指定将表创建于哪个表空间，则会自动创建于该用户的默认表空间
-- 数据库用户默认用户的表空间;
select user_id, username, default_tablespace from dba_users order by user_id desc;
select * from dba_users;



-- 创建用户，
-- test，密码test，分配的默认表空间test01，临时表空间 Temp
create user test
identified by test
default tablespace test01;      -- 如果创建用户时没有指定，则默认表空间为 USERS
-- temporary tablesapce TEMP;   -- 指定临时表空间，但是我没看到我这里有这个表空间？见后面，无需显示指定

-- 一般创建用户使用工具进行创建

-- 查看所有用户
select * from all_users;
select * from dba_users;


-- 删除用户
drop user test;



-- 更改数据库的默认表空间；
-- 原来的数据库默认表空间是 USERS
alter database default tablespace test02;


-- 重命名表空间
-- 并不会更改数据文件名
alter tablespace test01 rename to test_data;
alter tablespace test_data rename to test01;



-- 系统权限，比如建表
-- 对象权限，操作数据库对象（如表，视图）
-- 角色：是一组相关权限的组合，可以将权限授予角色，在把角色授予用户，以简化授权管理

-- 系统权限： 100多种
-- ANY 关键字，表明所有
-- GRANT命令是增加权限
-- REVOKE 命令是删除权限

-- 角色：
-- 几个常用的默认存在的角色
-- connect: 数据库连接角色
-- resource: 可以创建表、序列已经PL/SQL编程用方案对象，包括过程、触发器。
--            还要一个创建任何表的权限，才能创建表，该权限名为 create any table。


-- 授予权限
-- GRANT命令
-- 一般使用工具比如(PL/SQL deverloper)进行授权，权限和角色多不便记忆。
-- grant connect to test with admin option;       -- admin选项在此
grant connect to test;          -- 授予connect角色
grant resource to test;         -- 授予resource角色
grant dba to test;              -- 授予所有使用 admin 选项创建的系统权限。一般都赋予该角色，权限小于sys用户

-- 删除权限
-- revoke命令
revoke connect from test;



-- 常用数据类型：
-- 字符型：
-- char(n)    占用固定空间
-- varchar(n) 可变空间
-- varchar2() 
-- long

-- 数值型
-- number(p,s)
-- float

-- 日期型
-- data     默认格式为 DD-MON-YY

-- LOB类型
-- BLOB     二进制大对象(图形，mp3)    4G 
-- CLOB     字符大对象                 4G




-- 第6课： Oracle表创建

-- SQL语句
-- 注释：
-- SQL标准： /* */多行注释，  -- 单行注释
-- MySql注释： #


select * from session_roles;
select * from user_tables;


-- 创建表：
-- 必须有建表权限： create table 权限
-- 表的命名规则：必须以字母开头。不能是保留字，比如oracle, number
/*
创建课程表
*/
create table tb_clazz(
id number,
code varchar2(18)     
);      -- 可以在后面指定该表需要存放的表空间


/*
创建一个学生表（id，姓名，性别，年龄，地址，电话)
*/
create table tb_student(
id number,
sex char(6),
age number,
address varchar2(50),
tel varchar2(20)
);

select * from TB_STUDENT;


-- 第7课： 维护表
/*
alter table 语句：
alter table tab1 add  列；    增加列
alter table tab2 modify 列；   更改列
alter table tab3 drop  列 ；  删除列
*/

/*
添加邮箱字段，列
*/
alter table TB_STUDENT add email varchar2(60);


/*
将address修改为100长度
*/
alter table TB_STUDENT modify address varchar2(100);

-- 删除列
alter table TB_STUDENT drop column tel;
alter table tb_student  add mobile varchar2(20); 


/*
修改列名： 
alter table tbname rename column old to new;
*/
alter table tb_student rename column mobile to tel;


/*
修改表名：
rename tbname_old to tbname_new;
*/
rename tb_student to tb_s;

rename tb_s to tb_student;




/*
增加注释：
为了描述表、列的作用，可以使用comment语句为表和列增加注释
*/
-- comment on table tbname is 'text';
-- comment on column table.column_name is 'text';
;
comment on table tb_student is '学生表';
comment on column tb_student.tel is '电话'; 
/*
查看注释：利用数据字典
user_tab_comments 和 user_column_comments
*/
-- select * from user_tab_comments where comments is not null;
-- select * from user_col_comments where comments is not null;
select * from user_tab_comments;
select * from user_tab_comments where comments is not null;
select * from user_col_comments;
select * from user_col_comments where comments is not null;

/*
查询该用户所有表和该表所属的表空间
*/
select table_name, tablespace_name from user_tables;




/*
删除表：
drop table tb_name;
删除数据和结构。
它还有一些选项： purge 彻底删除表，无法回滚。
*/

drop table tb_student;

/*
恢复被删除表：
执行drop table 语句的时候，Oracle会将被删除表
存放到数据库回收站。从Oracle 10g开始，使用flashback table可以
快速恢复被删除表。
*/
flashback table tb_student to before drop;


/*
截断表：
当表必须保留，而表数据不再需要的时候。
truncate table
*/




/*
第8课：Oracle DML语句
*/
-- 插入数据
insert into TB_CLAZZ values(1, 'zixue');


select * from tb_clazz;
/*
事务：
对于Oracle来说，所有的DML语句，需要手动提交。（而上面的建表语句是DDL）
我们执行一条DML语句时，sql语句先是存在缓存中（回滚段）
需要手动提交或者回滚。可使用 commit 进行提交; 使用 rollback进行回滚

事务可以包含是多条语句，可以一次提交或回滚
commit 到下一次  commit 之间的所有SQL语句是属于一个事务。
*/
commit;   -- 提交

rollback;   -- 回滚



/*
建表的另一种方式，从其它表中创建
*/
create table tb_clazz2
as 
select code from tb_clazz;    -- 将code改为 * 就相当于复制表

select * from tb_clazz2;

-- 实现只复制数据结构
create table tb_clazz3
as 
select code from tb_clazz where code=NULL;  -- 让where无匹配行，则没有数据只是复制了表结构

select * from tb_clazz3;

-- 查看有哪些表
select table_name from user_tables;


/*
insert select
*/


-- update更新
select * from TB_CLAZZ;
update tb_clazz set code='fkjava' where id=1;
-- 字符数据进行比较必须加单引号


-- delete 删除数据(行)
-- 如果没有提交还可以回滚;
;
insert into TB_CLAZZ(code, id) values('自学', 2);  -- ('疯狂学院',3),('公司培训',4),('麦子学院',5),('兄弟连',6);
insert into TB_CLAZZ(code, id) values('疯狂学院',3);
insert into TB_CLAZZ(code, id) values('公司培训',4);
insert into TB_CLAZZ(code, id) values('麦子学院',5);
insert into TB_CLAZZ(code, id) values('兄弟连',6);
insert into TB_CLAZZ(code, id) values('黑马',7);
delete from tb_clazz where id = 1;


-- 上一课的DDL语句 : truncate
-- 截断所有数据，保留表结构，速度比delete快。但是drop最快。
-- 它是DDL语句，无法回滚；针对与整个表无法使用where子句
truncate table tb_clazz2;



/*
第9课： Oracle列级约束和表级约束
*/
-- 数据完整性：是指数据的精确性和可靠性。
-- 它是防止数据库中存在不符合语义规定的数据和防止因错误信息的输入输出造成无效输出或错误信息而提出。

-- 数据库采用多种方法来保证完整性，包括外键，约束，规则，和触发器。




/*
约束：
大部分数据库支持下面五类完整性约束。
NOT NULL
UNIQUIE Key 唯一键



约束作为数据库的对象，存放在系统表中，也有自己的名字。
*/

-- 非空约束 not null
-- null
-- check 约束
-- unique 唯一约束; 但可以多个null值，也就是只有有值就必须唯一。
-- 外键约束： 
--          子表外键列的值必须在主表参照列值的范围之内，或者为空。
--          当主表的记录被子表参照时，主表记录不允许被删除。
--          需为主键

-- default 默认值

-- 主键： 建议和业务无关


;
create table tb_student2(
id number  primary key ,
name varchar2(18) not null,
sex varchar2(6) not null check(sex = '男' or sex = '女'),
age number not null check(age > 17 and age < 60),
email varchar2(50) unique,
address varchar2(100)  default '广州',
tel varchar2(15),
clazz_id number not null references tb_clazz2(id)
);

-- 该表需在tb_student2之前创建
create table tb_clazz2(
id number primary key ,
code varchar2(18) not null
);



drop table tb_clazz2;
drop table tb_student2;


/*
表级约束：
写在表之后.
比表级约束更麻烦
constraints  约束
*/


create table tb_clazz2(
id number,
code varchar2(18) not null,
constraints tb_clazz2_pk primary key (id),
);

create table tb_student2(
id number ,
name varchar2(18) not null,
sex varchar2(6) not null ,
age number not null ,
email varchar2(50) ,
address varchar2(100)  default '广州',
tel varchar2(15),
clazz_id number not null ，   -- 它不在本表，需要添加表名
constraints tb_student_pk primary key (id),
constraints tb_student_unique unique (email), 
constraints tb_student_check_sex check(sex = '男' or sex = '女'),
constraints tb_student_check_age check(age > 17 and age < 60),
constraints tb_student_fk foreign key (clazz_id) references tb_clazz2(id) 
);




/*
第10课： 维护约束
可以增加或删除约束，但不能直接更改
*/
select * from user_tables;
select * from tb_clazz;


constraints tb_student_pk primary key (id),
constraints tb_student_unique unique (email), 
constraints tb_student_check_sex check(sex = '男' or sex = '女'),
constraints tb_student_check_age check(age > 17 and age < 60),
constraints tb_student_fk foreign key (clazz_id) references tb_clazz2(id)

-- 添加 primary key
alter table student2 add primary key();
-- 添加 foreign key
alter table student2 add foreign key (clazz_id) references tb_clazz(id);
-- 添加 unique
-- 添加 check
-- 添加 

-- 删除约束（一般不用）
-- 之前添加的表级约束
alter table student2 drop constraints 约束名;
-- 表级约束,的好处就在这里。另外其不同通用性

-- 禁止约束
alter table student2 disable constraints 约束名;
-- 激活约束
alter table student2 enable constraints 约束名;


-- 定义符合约束
-- 比如：联合主键
create table tb_person(
lastname varchar2(20),
firstname varchar2(20),
constraints tb_person_pk primary key (lastname, firstname)
);

-- 查看约束相关的列
select constraint_name, column_name
from user_cons_columns
where table_name = 'student2';



/*
第11课 ： 数据建模之三范式
*/
-- 数据模型规范化
-- 另见《Oracle入门很简单》

```






