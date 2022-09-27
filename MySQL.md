# MySQL

MySQL float double的精度问题是因为MySQL底层的存储都是通过二进制存储的，如果尾数不是0或5那么就无法用一个二进制的数值来精确的表达，所以只能四舍五入取一个近似值，同一个数值，使用double得到的最终数值误差比较小是因为double是8位的，精度更高。



DECIMAL 定点数类型，它的数值一定是精确的，因为MySQL底层针对DECIMAL的存储是将整数和小数分开的，分别转换成16进制进行存储；

DECIMAL类型 DECIMAL(5,2)的意思是 整数部分的长度不能超过（5-2）小数部分的长度不能超过2 ，11.22 可以，111.222最终结果会是111.22最后一位四舍五入后被丢弃，但是 1111.222就不行，因为整数部分的长度超过了；



Char 固定长度字符串类型 需要预定长度，如果太短会超过长度，如果太长会浪费存储空间,超过指定长度就会报错

Varchar 可变长度字符串类型，varchar定义的时候也需要设置最大值，但是只要不超过最大值都是按照实际的长度大小进行存储的;超过指定长度就会报错

Text 系统会自动按照实际长度进行存储，不需要设置范围,因为实际长度不固定，所以不能作为主键

MySQL表约束

默认约束 

主键约束 主键不能为空，主键的值必须唯一，主键可以作为外键，一张表只能有一个主键

外键约束 

非空约束  

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

HAVING: 用来筛选查询结果

ORDER BY 字段 DESC|ASC: 对查询结果进行排序，首先根据第一个字段进行排序，然后字段值相同的在根据第二个字段进行排序

ON DUPLICATE: 主键约束或者唯一约束受到破坏时的操作逻辑

```sql
INSERT INTO demo.goodsmaster SELECT *FROM demo.goodsmaster1 as a ON DUPLICATE KEY UPDATE barcode = a.barcode,goodsname=a.goodsname;
```

mysql分页有必要加order by 某个字段吗 听说如果不加的话 查询第二页数据的时候可能会查询到第一页的数据 如果加order by,可能会影响性能，这该如何取舍呢？



如何设置主键

1. 业务字段做主键: 尽量不要使用业务字段作为主键，因为谁也无法预测在整个项目的迭代过程中，业务字段会发生什么变化
2. 自增字段做主键: 在分布式系统下存在问题
3. 手动赋值字段做主键：根据具体业务去实现逻辑生成



外键 从表中用来引用主表中的数据的公共字段

外键约束：如果删除主表中的一条数据，这条数据在从表中被引用，那么MySQL会错误，从而保证关联数据的完整性

插入的时候需要先插入主表的数据，否则从表的外键找不到references

使用外键是有成本的，会消耗系统资源，虽然外键能保证关联数据的完整性，但是也可以在业务代码层面进行保障。

如何在业务层面进行关联数据的一致性：



【强制】不得使用外键与级联，一切外键概念必须在应用层解决。 说明：（概念解释）学生表中的student_id是主键，那么成绩表中的student_id则为外键。如果更新学生表中的student_id，同时触发成绩表中的student_id更新，即为级联更新。外键与级联更新适用于单机低并发，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库的插入速度。

外键约束带来的开销还包括锁开销，比如向子表中添加一条数据，外键约束会让innodb检查对应的父表。这里会需要对父表进行加锁操作，来确保这条记录不会在事务完成之前被删除。这就带来了额外的锁开销。甚至死锁。

外键是怎么带来死锁的：



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

inner join join cross join 都叫内连接

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







