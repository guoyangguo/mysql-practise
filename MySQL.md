# MySQL

数据库的三范式

1. 属性不可分割，每一个属性都是不可分割的原子项，
2. 

MySQL float double的精度问题是因为MySQL底层的存储都是通过二进制存储的，如果尾数不是0或5那么就无法用一个二进制的数值来精确的表达，所以只能四舍五入取一个近似值，同一个数值，使用double得到的最终数值误差比较小是因为double是8位的，精度更高。



DECIMAL 定点数类型，它的数值一定是精确的，因为MySQL底层针对DECIMAL的存储是将整数和小数分开的，分别转换成16进制进行存储；

DECIMAL类型 DECIMAL(5,2)的意思是 整数部分的长度不能超过（5-2）小数部分的长度不能超过2 ，11.22 可以，111.222最终结果会是111.22最后一位四舍五入后被丢弃，但是 1111.222就不行，因为整数部分的长度超过了；



Char 固定长度字符串类型 需要预定长度，如果太短会超过长度，如果太长会浪费存储空间,超过指定长度就会报错;

Varchar 可变长度字符串类型，varchar定义的时候也需要设置最大值，但是只要不超过最大值都是按照实际的长度大小进行存储的;超过指定长度就会报错;

Text 系统会自动按照实际长度进行存储，不需要设置范围,因为实际长度不固定，所以不能作为主键,也无法作为索引；

MySQL表约束

默认约束：添加默认值

主键约束： 主键不能为空，主键的值必须唯一，主键可以作为外键，一张表只能有一个主键

外键约束： 

非空约束  字段值不能为空

唯一性约束 对应的值必须唯一，但是可以为空，不能作为外键（因为外键不能为空），可以有多个满足唯一性约束的column

自增约束 只有int类型的数据才可以自增，每次+1，且不重复，自增的基数可以人为的改变

`null值和任何值都不相同包括自身，所有唯一性约束可以有多个null`

常用的建表语句

``` sql
CREATE TABLE
(
字段名 字段类型 PRIMARY KEY
);
CREATE TABLE
(
字段名 字段类型 NOT NULL
);
CREATE TABLE
(
字段名 字段类型 UNIQUE
);
CREATE TABLE
(
字段名 字段类型 DEFAULT 值
);
-- 这里要注意自增类型的条件，字段类型必须是整数类型。
CREATE TABLE
(
字段名 字段类型 AUTO_INCREMENT
);
-- 在一个已经存在的表基础上，创建一个新表
CREATE TABLE demo.importheadhist LIKE demo.importhead;
-- 修改表的相关语句
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 数据类型;
ALTER TABLE 表名 ADD COLUMN 字段名 字段类型 FIRST|AFTER 字段名;
ALTER TABLE 表名 MODIFY 字段名 字段类型 FIRST|AFTER 字段名;
```

查询语句的语法结构

```sql
SELECT * |字段列表
FROM 数据源
WHERE 条件
GROUP BY 字段
ORDER BY 字段
LIMIT 起始点,行数
```

GROUP BY： 用来对数据进行分组，通常跟聚合函数一起使用

HAVING: 用来筛选查询结果，必须和Group By 一起使用

ORDER BY 字段 DESC|ASC: 对查询结果进行排序，首先根据第一个字段进行排序，然后字段值相同的在根据第二个字段进行排序

ON DUPLICATE: 主键约束或者唯一约束受到破坏时的操作逻辑

```sql
INSERT INTO demo.goodsmaster SELECT *FROM demo.goodsmaster1 as a ON DUPLICATE KEY UPDATE barcode = a.barcode,goodsname=a.goodsname;
```

mysql 分页有必要加order by 某个字段吗 听说如果不加的话 查询第二页数据的时候可能会查询到第一页的数据 如果加order by,可能会影响性能，这该如何取舍呢？



如何设置主键

1. 业务字段做主键: 尽量不要使用业务字段作为主键，因为谁也无法预测在整个项目的迭代过程中，业务字段会发生什么变化
2. 自增字段做主键: 在分布式系统下存在问题
3. 手动赋值字段做主键：根据具体业务去实现逻辑生成



外键 从表中用来引用主表中的数据的公共字段

外键约束：如果删除主表中的一条数据，这条数据在从表中被引用，那么MySQL会错误，从而保证关联数据的完整性，先删除从表的数据，在删除主表的关联数据

插入的时候需要先插入主表的数据，否则从表的外键找不到references

使用外键是有成本的，会消耗系统资源，虽然外键能保证关联数据的完整性，但是也可以在业务代码层面进行保障。

如何在业务层面进行关联数据的一致性：

提供一个保证数据一致性的业务代码

【强制】不得使用外键与级联，一切外键概念必须在应用层解决。 说明：（概念解释）学生表中的student_id是主键，那么成绩表中的student_id则为外键。如果更新学生表中的student_id，同时触发成绩表中的student_id更新，即为级联更新。外键与级联更新适用于单机低并发，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库的插入速度。

外键约束带来的开销还包括锁开销，比如向子表中添加一条数据，外键约束会让innodb检查对应的父表。这里会需要对父表进行加锁操作，来确保这条记录不会在事务完成之前被删除。这就带来了额外的锁开销。甚至死锁。

外键是怎么带来死锁的：

有A B两个table，A为主表，B为从表，有如下过程

``` sql
-- 事务1 
-- step1 对从表B的一条数据进行更新，会触发外键约束，将主表A id=1的record加上共享锁
update B set B.num = 2 where a_id = 1
-- step2 发现 id = 1 存在事务2的共享锁，等待
update A set A.num = 3 where id = 1
-- 
-- 事务2
-- step1 对从表B的一条数据进行更新，会触发外键约束，将主表A id=1的record加上共享锁
update B set B.num = 2 where a_id = 1
-- step2 发现 id = 1 存在事务1的共享锁，等待
update A set A.num = 3 where id = 1

-- 以上过程会形容事务1和事务2之间的相互等待，导致死锁

```





``` sql
-- 创建外键

CREATE TABLE 从表名
(
  字段名 类型,
  ...
-- 定义外键约束，指出外键字段和参照的主表字段
CONSTRAINT 外键约束名
FOREIGN KEY (字段名) REFERENCES 主表名 (字段名)
)

-- 修改表结构创建外键

ALTER TABLE 从表名 ADD CONSTRAINT 约束名 FOREIGN KEY 字段名 REFERENCES 主表名 （字段名）; 

-- 查询外键
SELECT * FROM information_schema where constraint_name = 'constraint_name'
```



Join 连接类型

Inner join 内连接 ：只返回 符合条件的记录

inner join， join， cross join 都叫内连接

Outer join 外连接：返回某一个表中的所有记录和另一个表中的满足连接条件的记录

外连接分为2种：

左连接：LEFT JOIN 返回左表全部记录以及右表符合条件的记录

右连接：RIGHT JOIN 返回右表的全部记录以及左表符合条件的记录

``` sql
-- 定义外键约束：
CREATE TABLE 从表名
(
字段 字段类型
....
CONSTRAINT 外键约束名称
FOREIGN KEY (字段名) REFERENCES 主表名 (字段名称)
);
ALTER TABLE 从表名 ADD CONSTRAINT 约束名 FOREIGN KEY 字段名 REFERENCES 主表名 （字段名）;

-- 连接查询
SELECT 字段名
FROM 表名 AS a
JOIN 表名 AS b
ON (a.字段名称=b.字段名称);
 
SELECT 字段名
FROM 表名 AS a
LEFT JOIN 表名 AS b
ON (a.字段名称=b.字段名称);
 
SELECT 字段名
FROM 表名 AS a
RIGHT JOIN 表名 AS b
ON (a.字段名称=b.字段名称);
```

HAVING 必须和group by 一起使用，

如何正确的使用 HAVING和 WHERE 

对于有连接的查询，where是先筛选出符合where条件的语句，再进行连接查询

having 对查询后的结果集进行分组过滤



count(*) 和 count(字段)

count(*)： 统计有多少条记录

count(*): 统计有多少不为空的字段值

count(1) 统计表的行数，在MySQL中count(*)与count(1)是一样的

count(字段)，如果字段是主键则count(字段更快)





时间函数：

YEAR（date）：获取 date 中的年。

MONTH（date）：获取 date 中的月。

DAY（date）：获取 date 中的日。

HOUR（date）：获取 date 中的小时。

MINUTE（date）：获取 date 中的分。

SECOND（date）：获取 date 中的秒。

EXTRACT(hour from create_time)：从create_time 中获取小时

计算时间的函数：

DATE_ADD(create_time,INTERVAL -1 YEAR): 在crete_time的基础上减去一年

DATE_ADD(create_time,INTERVAL 1 YEAR): 在create_time的基础上增加一年

LAST_DAY(create_time): 获取create_time 所在月的最后一天



CURDATE() 回去当前日期

DAYOFWEEK(CURDATE()) 获取当前日期是周几



CASE 语法

``` sql
CASE 表达式 WHEN 值1 THEN 表达式1 [ WHEN 值2 THEN 表达式2] ELSE 表达式m END
```

 数学函数

取整函数

向上取整 CEIL(X) 和 CEILING(X)：返回大于等于 X 的最小 INT 型整数。

向下取整 FLOOR(X)：返回小于等于 X 的最大 INT 型整数。

舍入函数 ROUND(X,D)：X 表示要处理的数，D 表示保留的小数位数，处理的方式是四舍五入。ROUND(X) 表示保留 0 位小数。

绝对值函数 ABS()

求余函数()



常用的字符串函数有 4 个。

CONCAT（s1,s2,...）：表示把字符串 s1、s2……拼接起来，组成一个字符串。

CAST（表达式 AS CHAR）：表示将表达式的值转换成字符串。

CHAR_LENGTH（字符串）：表示获取字符串的长度。

SPACE（n）：表示获取一个由 n 个空格组成的字符串。

SUBSTR(s,n): 从字符串S的第n个位置开始到结束的子字符串

MID(s,n,len):从字符串s的第n个位置开始，获取长度为len的子字符串

TRIM(s1 from s):除去字符串两端所有字符串s1

LTRIM(s): 除去字符串s左边的所有空格

RTRIM(s):除去字符串右边的所有空格



条件判断函数

IFNULL(V1,V2): 如果V1的值不为空则返回V1,否则返回V2

IF(表达式,V1,V2): 如果表示为true,返回v1,否则返回v2



索引：

单字段索引：

创建单字段索引的三种方法：

1. create index <indexname> on table <tablename> (字段名)

2. 创建表的时候直接创建索引 

   ```sql
   create table <tablename>
   (
     字段 数据类型,
     ...
     index indename字段
     
   )
   ```

   

3. 修改表时创建索引

   ```sql
   alter table demo.student
   add index stu_name_index(class_id);
   ```

给表设置逐渐约束和唯一性约束的时候，MySQL会自动创建对应的主键索引和唯一性索引

在选择索引的时候，要选择经常用来当作查询条件的字段当作索引

如果存在多个索引，MySQL优化器会选择检索条目书最少的索引

单字段索引的作用原理 



复合索引：

MySQL最多支持16个字段的组合索引；

创建组合索引的语法与创建组合索引的语法类似，不同之处在于组合索引的字段是多个的

```sql
-- 创建组合索引的三种方法
1. create index indexname on table (字段名)
2. alter table tablename
   add index indexname (字段名)
```

组合索引的原理

- 组合索引遵从最左匹配原则，也就是说查询条件需要与组合索引的字段匹配，比如 创建组合索引是（A，B，C）where 条件中的顺序是A，B,C那就是完全匹配，查询效率比较高，如果是 （C，B，A）那就只能匹配到A字段的索引，如果是（C，B）那就会发生索引中断。

- 组合索引不适合使用范围查询，因为无法精确定位，会倒是组合索引中断

- 只引用组合索引中的单个索引，没有直接引用单个索引的效果好
- 在创建组合索引的时候，尽量将查询频率最高的字段放在最左侧，从左往右依次添加；组合索引遇到范围查询的时候，后面的索引就不起作用了，所以尽量将等值查询字段放在最左

删除索引

``` sql
-- 普通索引的删除
drop index indexname on table
-- 删除主键索引
alter table drop primary key
```



使用索引带来的劣势：

1. 占用存储空间，因为索引的存储是需要空间的，典型的空间换时间
2. SQL操作上的开销，无论是新增 删除 修改只要是涉及到索引字段都是对索引进行修改，以保证索引能指向正确的记录。



事务

事务是数据库的一项功能，能保证一组操作要么全部执行，要么不执行。

事务的四个特性：ACID

原子性：不可分割，要么全部执行，要么不执行

一致性：数据的完整性不会因为事务的执行而受到破坏

隔离性：多个事务同时执行的时候，互相隔离，不同的隔离级别，隔离的程度不一样

事务的4种隔离等级：

1. read uncommitted: 可以读取事务中被修改但还没提交的数据
2. read committed: 可以读取事务中已经被提交的数据
3. repeatable read: 在一个事务中，对一个数据读取的值永远和第一次的读取一致，不受其他事务的影响，MySQL的默认隔离级别
4. Serializable: 表示任何一个事务只要对数据进行了任何操作，那么直到这个事务结束，MySQL都会将会这个数据锁住，禁止其他事务对这个数据的操作；安全性最好



持久性：事务对数据的修改是永久有效的



事务不会对整个过程的SQL语句的错误进行处理，一旦某个SQL语句出现了错误，需要手动的进行rollback，如果仍然commit的话，那么会导致数据不一致，只有所有SQL语句都正常执行，才能进行commit



临时表：临时表是一种特殊的表，用来存储查询的中间结果，临时表会随着连接的断开而销毁。

MySQL:有2种临时表：

1. 内部临时表：系统自动产生，主要用于性能优化，无法看到，无法使用
2. 外部临时表：可以通过SQL自己创建，自己使用

``` sql
--创建临时表
create temporary table <tablename>
(
  ...
);
```

临时表具有连接隔离性，只在当前连接可见，互不干扰，所以不同连接可以创建同名的临时表，适合并发的应用场景。

临时表是可以建立索引的

按照存储类型来分：

临时表又可以分为：内存临时表，磁盘临时表，内存临时表需要制定 engineer=memory,内存临时比较快，但是占用内存资源，相比之下，磁盘临时表存储在磁盘上，内存占用少，但是查询速度比较慢



视图 view

```sql
-- 创建视图
create [or replace] view <viewname>
[(字段列表)]
as 查询语句;	

-- 修改视图
alter view <viewname>
as 查询语句;

-- 查看视图
describe <viewname>
```

视图是以查询语句建立的，如果视图与基础表之间没有一一对应的关系，那么视图就只能进行查看		 	 	

使用视图的好处：

1. 视图是将一段查询语句存储到数据库中，在需要的时候将视图看作一张表，进行查询

2. 视图简化了复杂的查询语句，可读性更好，也更方便维护
3. 视图本身不存储数据，不占用存储资源
4. 视图具有隔离性，因为视图是将查询语句进行封装，避免了直接操作数据源的表

使用视图的缺点：

1. 增加维护的成本：视图与源表之间具有依赖性，当源表结构发生变动时，视图也需要进行对应的修改

视图会使用索引吗？

视图的本质是一段查询语句，如果源表的创建索引，视图的查询语句中使用到了该字段，就会使用索引；

视图与临时表有什么区别

1. 作用范围：临时表是作用于当前连接的，当前连接断开，临时表也会销毁，具有隔离性，不同连接之间互相不影响；视图是持久化的，只有当前连接拥有对应的权限就可以进行对应的操作。
2. 功能特性：视图是用来简化查询的，本身不存储数据，不占用存储空间；临时表是用来存储中间数据的，会占用内存或者磁盘空间（取决于使用的是内存临时表（指定engine=memory）还是磁盘临时表）

视图操作

```SQL
-- 创建 demo.student 视图
CREATE OR REPLACE VIEW demo.view_demo_student (name , class_id , create_time) AS
    SELECT 
        a.name, a.class_id, a.create_time
    FROM
        demo.student AS a;
-- 视图查询 
select * from demo.view_demo_student;

-- 向视图中插入数据 需要视图与基础表之间字段之间存在对应关系 
insert into demo.view_demo_student (name,class_id,create_time) values ('赵六',4,now());

-- 查看基础表的数据 
select * from demo.student;

-- 更新视图的数据 更新失败，因为 视图中没有id字段
update demo.view_demo_student as a set a.name = '赵6' where a.id = 6; 

-- 更新视图数据 
UPDATE demo.view_demo_student AS a 
SET 
    a.name = '赵6'
WHERE
    a.name = '赵六';

-- 查看视图数据与基础表数据 
select * from demo.view_demo_student;

select * from demo.student;

-- 修改视图 
alter view demo.view_demo_student
as select * from demo.student;

-- 查看视图
describe demo.view_demo_student;

select * from demo.view_demo_student;

-- 删除视图 
drop view demo.view_demo_student;

show variables like '%safe%';
```

存储过程的使用

1. 存储过程是将某个操做的一系列的SQL语句定义好，存储在MySQL服务端，在需要用的时候向服务端发出call命令进行调用

   ``` SQL
   -- 创建存储过程 
   delimiter //
   create procedure get_scores (in class_id int, out scores int)
   begin 
    select sum(a.score) into scores from demo.course a ; 
   end
   //
   delimiter ;
   
   -- 删除存储过程 
   drop procedure get_scores;
   
   -- 查看存储过程 
   show create procedure get_scores;
   
   -- 调用存储过程
   call get_scores(1,@scores);
   
   -- 获取存储过程运行的结果
   select @scores;
   ```

   存储程序分为2种，存储过程和存储函数

   存储函数

   创建函数的语法

   ``` SQL
   create function <funcname>(参数) returns 数据类型 函数体
   ```

   函数与存储过程的不同之处

   1. 存储过程可以没有返回值，存储函数不行，必须有返回
   2. 存储过程可以通过call命令进行调用，存储函数不行
   3. 存储函数可以在select 语句中使用，存储过程不行
   4. 存储过程可以对表的结构进行修改，甚至可以删除表但是存储函数不行

条件处理语句

```SQL
DECLARE 处理方式 HANDLER FOR 问题 操作；
```

游标：游标是MySQL中对结果集操作的一种方式，游标可以获取结果集中的前一条后一条的数据，也可以直接跳转到某一条的数据。

使用的游标的方式

``` SQL
1. 声明游标
declare cursorname cursor for 查询语句;
2. 打开游标
open cursorname;
3. 使用游标操作结果集中的数据
fetch corsorname into 变量列表;
4. 关闭游标
close cursor;
```

游标使用完一定要及时的关闭，因为使用游标会损耗系统资源，如果没有及时关闭，游标会保留到存储程序执行结束。

游标的使用

```SQL
create database if not exists demo;

create table if not exists demo.test (
 id int primary key,
 quant int 
);

insert into demo.test(id,quant) values(1,100);
insert into demo.test(id,quant) values(2,101);
insert into demo.test(id,quant) values(3,102);
insert into demo.test(id,quant) values(4,103);

-- 创建存储过程
delimiter //
create procedure demo.test_pro ()
begin
  -- 声明变量进行操作 
  declare v_id int;
  declare v_quant int;
  declare done int DEFAULT FALSE;

  -- 声明游标 
  declare  test_cursor cursor for select * from demo.test ;
  -- 声明异常处理事件 
  declare continue handler for NOT FOUND set done = true;
  -- 打开游标
  open test_cursor;
  -- 使用游标
  fetch test_cursor into v_id,v_quant;
    repeat
    if (v_id MOD 2=0) then update demo.test a set a.quant = a.quant+1 where a.id = v_id;
    else update demo.test a set a.quant = a.quant+2 where a.id = v_id;
    end if;
    
    fetch test_cursor into  v_id,v_quant;
  until done end repeat;
  -- 关闭游标
  close test_cursor;
end 
//
delimiter ;

show create procedure demo.test_pro;

call demo.test_pro();

select * from demo.test;

drop procedure demo.test_pro;

```



触发器 

使用触发器的好处：

1. 触发器可以保证关联数据的完整性，不会存在遗忘的情况
2. 触发器可以记录操作日志

触发器的缺点：带来数据库的维护成本，因为多了需要关注点。

```SQL
-- 触发器的使用
-- 创建触发器
create trigger <triggersname> {before|after} [insert|update|delete] on <tablename> for each row <expression>;
-- 查看触发器
show triggers;
-- 删除触发器
drop trigger <triggername>;
-- 使用案例，在学生表插入信息，同时更新课程表

-- 创建触发器
 delimiter //
 CREATE 
    TRIGGER  demo_insert_course
 AFTER INSERT ON demo.student FOR EACH ROW 
    BEGIN 
     update demo.class a set a.number = a.number +1 where a.id =  new.class_id ;
    END
//
delimiter ;
-- 查看触发器
show triggers;

select * from demo.class;
insert into demo.class(name)values('107班');

-- 插入学生数据，观察触发器是否执行
insert into demo.student(name,class_id) values ('黄梦凯',1);

select * from student;
select * from demo.class;
```

MySQL 的用户角色权限管理

```SQL
create database if not exists demo;

CREATE TABLE IF NOT EXISTS demo.student (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(25),
    class_id INT NOT NULL,
    leader_id INT NOT NULL
);

describe demo.student;

SELECT 
    *
FROM
    demo.student;

insert into demo.student(name,class_id,leader_id) values('郭国阳',1,2);

CREATE TABLE IF NOT EXISTS demo.tearcher (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(25),
    department_id INT NOT NULL,
    leader_id INT NOT NULL
);

describe demo.tearcher;

insert into demo.tearcher(name,department_id,leader_id) values('张三',2,1);

CREATE TABLE IF NOT EXISTS demo.department (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(25)
);

describe demo.department;

insert into demo.department(name) values('教务处'),('语文组'),('数学组');

select * from demo.department;

use demo;
show tables;

-- 数据库用户权限管理
-- 创建角色
-- create role 角色名称>
-- 在MySQL中创建角色之后，默认是没有激活
set global activate_all_roles_on_login=ON;


-- 删除角色
-- drop  role 角色名称 

-- 给角色授权
-- grant 权限 on 表名 to 角色名
 
-- 查看角色权限
-- show grants for 角色名

-- 创建用户，可以不指定密码，但是不建议，无密码登陆不安全
-- create user username [identified by password]

-- 给用户授权 
-- grant 角色名称 to 用户名称

-- 也可以直接将表的权限直接授给用户

-- 查看用户权限
show grants for username;

-- 删除用户
drop user username;

-- 尽量不要在应用层数据库的root用户，因为root用户的密码一旦泄露，就存在很大的风险

-- eg. 创建校长 老师 学生三个角色
create role 'schoolmaster';
create role 'tearcher';
create role 'student'@'localhost';
-- create role 'schoolmaster'@'%'
-- 开始权限
set global activate_all_roles_on_login=ON;

-- 给角色授权
grant all privileges on demo.* to 'schoolmaster';
show grants for 'schoolmaster';

grant select on demo.tearcher to 'tearcher';
grant select ,update,insert on demo.student to 'tearcher';
show grants for 'tearcher';

grant select on demo.tearcher to 'student'@'localhost';
show grants for 'student'@'localhost';

-- 创建用户
-- 校长
create user 'xiaozhang' identified by 'xiaozhang';
create user 'laoshi' identified by 'laoshi';
create user 'guoyang' identified by 'guoyang';

-- 给用户赋予角色
grant 'schoolmaster' to 'xiaozhang';
grant 'tearcher' to 'laoshi';
grant 'student'@'localhost' to 'guoyang';

-- 查看用户权限
show grants for 'guoyang';

-- 删除用户
drop user 'guoyang';
```



MySQL日志

日志种类

1. 通用查询日志
2. 慢查询日志
3. 二进制日志
4. 错误日志
5. 中继日志
6.  重做日志
7. 回滚日志

```SQL
-- MySQL日志日志
-- 通用查询日志a
-- 慢查询日志
-- 错误日志

-- 二进制日志 bin log
-- 中继日志 relay log 
-- 重做日志 redo log
-- 回滚日志 undo log


-- 通用查询日志
-- 通用查询日志的状态和文件路径
show variables like '%general%';
-- 默认情况下通用查询日志是关闭的，因为开启通用查询日志后，会记录所有的连接起始的SQL操作，会消耗系统资源和占用磁盘空间，但能还原问题的场景
-- 开启通用查询日志
set global general_log = ON;
-- 通用查询日志文件会不断的追加，导致日志文件越来越大，所以需要定期的对日志文件进行备份
-- 备份过程 
-- 1.首先关闭通用查询日志 set global general_log = OFF;
-- 2.备份日志文件 将日志文件拷贝到备份目录
-- 3.删除当前文件
-- 4.开启通用日志查询  set global general_log = ON;

-- 慢查询日志
-- 慢查询日志是记录SQL执行时间超过指定值的日志
-- 慢查询日志信息是通过*/support-files/my.conf配置文件来配置的，也可以通过SQL命令进行修改，但是通过配置文件修改需要重启MySQL服务

-- 表示开启慢查询日志，系统将会对慢查询进行记录
-- slow-query-log=1 
 
-- 表示慢查询日志的名称是"GJTECH-PC-slow.log"
-- slow_query_log_file="GJTECH-PC-slow.log" 

-- 通过慢查询日志可以发现运行比较慢的SQL,发现系统问题，从而进行优化


-- 错误日志
-- 错误日志记录MySQL服务本身的运行信息，服务的启动 停止过程中的诊断信息，包含错误和提示

-- 二进制日志 
-- 二进制日志主要来记录数据库的更新事件，比如：创建数据库 更新数据花费的时长；而且二进制日志具有延续性，可以利用bin log 进行数据恢复和数据同步
-- 查看二进制日志
-- 1. 查看当前正在写的bin log
show master status;

-- 2. 查看所有的bin log
show binary logs;

-- 3. 查看bin log中所有数据的事件
-- show binlog events in biglog_filename
show binlog events in 'binlog.000002';

-- 刷新bin log
-- 刷新bin log将关闭服务器正在写的bin log，并重新生成一个bin log 文件进入写操作，新的bin log文件名会在前一个文件名后缀+1

-- bin log主要的作用是可以用来恢复数据
-- 可以用 mysqlbiglog -start-position=xxx --stop-position=xxx binlong_filename | mysql -u username -p password

-- 删除 bin log
-- 删除bin log 日志之前 一定要进行数据备份 

-- 删除比指定文件编号小的 bin log
purge master logs to 'binlog.000002';

-- 删除所有的bin log，并且重新从编号1开始新增bin log文件
reset master;

-- 通过 bin log 进行数据库恢复

-- 1. 模拟数据
CREATE TABLE demo.backup (
    id INT PRIMARY KEY AUTO_INCREMENT,
    operator VARCHAR(50) NOT NULL,
    create_time DATETIME NOT NULL DEFAULT NOW(),
    update_time DATETIME NOT NULL DEFAULT NOW()
);

insert into demo.backup (operator) values ('郭国阳');

select * from demo.backup;

-- 2. 备份当前数据
-- 对demo做整库备份
-- mysqldump -u root -p demo> db_demo_back.sql

-- 3. 

-- 4. 新增一条数据
insert into demo.backup (operator) values ('黄婉秋');

-- 5. 假设数据库服务崩溃，从备份文件中恢复数据
-- 首先查看当前MySQL服务使用的bin log文件，确定正在使用的bin log文件
show binary logs;
-- 查看当前bin log正在写位置，与恢复后的bin log 位置进行对比
show master status;
--  模拟MySQL服务崩溃，手动删除 db demo
drop database demo;
-- 新建db demo进行数据恢复
create database demo;
-- 根据备份文件进行数据恢复 
-- mysql -u root -p demo < db_demo_back.sql;
-- 查看恢复后的数据
select * from demo.backup;
-- 6.根据bin log 恢复备份与宕机之间丢失的数据
show binary logs;
-- 首先查看bin log event 确定丢失数据的position
show binlog events in 'binlog.000002';
-- 从bin log指定位置恢复数据
-- mysqlbinlog --start-position=4141 --stop-position=4296 'binlog.000002'
-- 查看恢复后的数据
select * from demo.backup;



-- 中继日志 relay log
-- 中继日志 只存在于MySQL主从服务的从服务器上，主要作用是用来将从主服务器上读取的bin log日志写到relay log中，在从relay log中同步到从服务器的数据中，完成主从服务之间的数据同步
-- 可以用mysqlbinlog 工具查看

-- 回滚日志 undo log



```

