---
title: SQL
toc: true
date: 2021-12-02 20:58
tags:
  - Database
  - SQL
categories: [Database]
updated: 2021-12-02 20:58
references:
  - title: 常用 SQL 语句 - LeetCode
    url: https://leetcode-cn.com/leetbook/read/database-handbook/pxsgh2/
  - title: SQL约束 - 掘金
    url: https://juejin.cn/post/6983974232656969742
---

SQL 语句可分为以下几类：

- 数据定义语言 DDL（Data Ddefinition Language）：即逻辑操作，如 `CREATE`，`DROP`，`ALTER`
- 数据查询语言 DQL（Data Query Language）：即查询操作，以 `SELECT` 关键字为主
- 数据操纵语言 DML（Data Manipulation Language）：即增删改操作，如 `INSERT`，`UPDATE`，`DELETE`
- 数据控制功能 DCL（Data Control Language）：即权限控制操作，如 `GRANT`，`REVOKE`，`COMMIT`，`ROLLBACK`

<!-- more -->

## 1、键

- **超键**：在关系中，能唯一标识元组的属性集称为关系模式的超键。

> 一个属性可以作为一个超键，多个属性组合在一起也可以作为一个超键。
> 超键包含候选键和主键。

- **候选键**：是最小超键，即没有冗余元素的超键。
- **主键**：数据库表中对储存数据对象予以 **唯一和完整标识的数据列或属性的组合**。

> 一个数据列只能有一个主键，且主键的取值不能缺失，即不能为空值（NULL）。

- **外键**：在一个表中存在的另一个表的主键称此表的外键。

> 外键可以有重复的, 可以是空值。

## 2、约束

约束是一种简单地强加于表中一列或多列的限制，从而保证表中数据一致性（准确和可靠）。以下为六大约束：

- **非空约束（NOT NULL）**：保证该字段值一定不为空；
- **默认约束（DEFAULT）**：保证字段有默认值；
- **主键约束（PRIMARY KEY）**：标志一列或者多列，并保证其值在表内的唯一性；
- **外键约束（FOREIGN KEY）**：限制一列或多列中的值必须被包含在另一表的外键列中，并且在级联更新或级联删除规则建立后也可以限制其他表中的可用值；
- **唯一约束（UNIQUE）**： 限制一列或多列的值，保证字段值在表内的唯一性，可以为空（主键约束是一种特殊类型的唯一约束）；
- **检查约束（CHECK）**：限制一列的可用值范围。

### 创建约束

1. 创建表时，在字段描述处，声明指定字段为主键：

```sql
CREATE TABLE persons (
  pid int primary key, -- 添加了主键约束
...
);
```

2. 创建表时，在 constraint 约束区域，声明指定字段为主键：

```sql
CREATE TABLE persons (
  pid INT,
  lastname VARCHAR(255),
  firstname VARCHAR(255),
  address VARCHAR(255),
  CONSTRAINT pk_persons PRIMARY KEY (lastname, firstname) -- 添加主键约束, 多个字段, 我们称为联合主键。
);
-- pk_persons 是约束名，在数据库中应是惟一的。如果不指定，则系统会自动生成一个约束名。
```

3. 创建表之后，通过修改表结构，声明指定字段为主键：

```sql
-- 添加联合主键约束
-- 其他约束类似
alter table persons add constraint pk_persons primary key (lastname, firstname); 
```

### 删除约束

```sql
-- 删除主键约束
-- ALTER TABLE 表名 DROP PRIMARY KEY
alter table persons drop primary key;	

-- 删除非空约束
-- ALTER TABLE 表名 MODIFY 字段名 数据类型[长度]
alter table persons modify lastname varchar(255);

-- 删除唯一约束
-- ALTER TABLE 表名 DROP INDEX 名称
-- 有唯一约束名称, 使用约束名称删除
alter table persons drop index uni_persons_address; 
-- 没有唯一约束名称, 使用字段名删除
alter table persons drop index address; 

```

## 3、SQL 语句

参考 w3cschool 即可：

> [详尽的SQL语句大全分类整理_w3cschool](https://www.w3cschool.cn/sql/sql-sentence.html)

### 增删改查 CRUD

```sql
-- 插入
-- 插入完整行
INSERT INTO user VALUES (10, 'root', 'root', 'xxxx@163.com');
-- 插入行的部分
INSERT INTO user(username, password, email) VALUES ('admin', 'admin', 'xxxx@163.com');

-- 更新
UPDATE user SET username='robot', password='robot' WHERE username = 'root';

-- 删除
-- 删除指定数据
DELETE FROM user WHERE username = 'robot';
-- 清空表
TRUNCATE TABLE user;

-- 查询
-- 多行查询
SELECT prod_id, prod_name, prod_price FROM products;
-- 全部查询
SELECT * FROM products;
-- 查询去重
SELECT DISTINCT vend_id FROM products;
-- 查询限制 limit x,y 从x（起始0）行开始，查询y条记录
SELECT * FROM mytable LIMIT 0, 5; 
```

### 关联查询

在项目开发过程中，使用数据库查询语句时，有很多需求都是要涉及到较为复杂或者多表的连接查询，需要关联查询实现。以下为总结的 MySQL 的五种关联查询。

- **交叉连接（CROSS JOIN）**

	除了在 `FROM` 子句中使用 **逗号间隔连接的表** 外，SQL 还支持另一种被称为交叉连接的操作，它们都返回被连接的两个表所有数据行的 **笛卡尔积**，返回到的数据行数等于第一个表中符合查询条件的数据行数 **乘以** 第二个表中符合查询条件的数据行数。惟一的不同在于，交叉连接分开列名时，使用 `CROSS JOIN` 关键字而不是逗号，即以下两个表达式等价：

	`SELECT  *  FROM A, B`
	`SELECT  *  FROM A CROSS JOIN B`

- **内连接（INNER JOIN）**

	内连接分为三类，分别是 **等值连接**：`ON A.id = B.id`、**不等值连接**：`ON A.id > B.id` 和 **自连接**：`SELECT * FROM A T1 INNER JOIN A T2 ON T1.id = T2.pid`

- **外连接（LEFT JOIN/RIGHT JOIN）**

	**左外连接**：以左表为主，先查询出左表，按照 `ON` 后的关联条件匹配右表，没有匹配到的用 `NULL` 填充，可以简写成 `LEFT JOIN`
	**右外连接**：以右表为主，先查询出右表，按照 `ON` 后的关联条件匹配左表，没有匹配到的用 `NULL` 填充，可以简写成 `RIGHT JOIN`

- **联合查询（UNION 与 UNION ALL）**

	`SELECT * FROM A UNION SELECT * FROM B UNION …`

	联合查询就是把多个结果集集中在一起，`UNION` 前的结果为基准，需要注意的是联合查询的 **列数要相等**，相同的记录行会合并；

	如果使用 `UNION ALL`，不会合并重复的记录行，所以效率更高。

- **全连接（FULL JOIN）**

	MySQL 本身不支持全连接，但可以通过联合使用 `LEFT JOIN`、`UNION` 和 `RIGHT JOIN` 来实现。

	`SELECT * FROM A LEFT JOIN B ON A.id = B.id UNIONSELECT * FROM A RIGHT JOIN B ON A.id = B.id`

```sql
-- join
-- 内连接
SELECT vend_name FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id;
-- 自然连接：内连接提供连接的列，而自然连接自动连接所有同名列。
SELECT c1.cust_id FROM customers c1, customers c2 WHERE c1.cust_name = c2.cust_name AND c2.cust_contact = 'Jim Jones';
-- 外连接
-- 左连接：左外连接就是保留左表没有关联的行
SELECT orders.order_num FROM customers LEFT JOIN orders ON customers.cust_id = orders.cust_id;
-- 右连接：左外连接就是保留左表没有关联的行
SELECT orders.order_num FROM customers RIGHT JOIN orders ON customers.cust_id = orders.cust_id;

-- union
SELECT cust_name, cust_contact, cust_email FROM customers WHERE cust_state IN ('IL', 'IN', 'MI')
UNION
SELECT cust_name, cust_contact, cust_email FROM customers WHERE cust_name = 'Fun4All';
```

### 子查询

多条 MySQL 语句嵌套使用时，内部的 MySQL 查询语句称为子查询。

子查询是一个 `SELECT` 语句，它嵌套在另一个 `SELECT`、`SELECT…INTO` 语句、`INSERT…INTO` 语句、`DELETE` 语句、 `UPDATE` 语句或嵌套在另一子查询中。

MySQL 的子查询是多表查询的一个重要组成部分，常常和 **连接查询** 一起使用，是多表查询的基础。

子查询分为以下四类：

- **标量子查询**

	查询返回单一值的标量，如一个数字或一个字符串，是子查询中最简单的形式。

- **列子查询**

	子查询返回的结果集是 N 行一列，该结果通常来自对表的 **某个字段** 查询返回。

- **行子查询**

	子查询返回的结果集是一行 N 列，该结果通常是对表的 **某行数据** 进行查询而返回的结果集

- **表子查询**

	子查询返回的结果集是 N 行 N 列的一个表数据。

```sql
-- where
SELECT * FROM Customers WHERE cust_name = 'Kids Place';
-- in
SELECT * FROM products WHERE vend_id IN ('DLL01', 'BRS01');
-- between
SELECT * FROM products WHERE prod_price BETWEEN 3 AND 5;
-- 条件：and or not 省略
-- like：% 匹配n次，_ 匹配一次
SELECT prod_id FROM products WHERE prod_name LIKE '%bean bag%';
SELECT prod_id FROM products WHERE prod_name LIKE '__ inch teddy bear';
```

### 排序和分组

- `ORDER BY`：用于对结果集进行排序。
- `ASC` ：升序（默认）
	- `DESC` ：降序
- 可以按多个列进行排序，并且为每个列指定不同的排序方式
- `GROUP BY`：将记录分组到汇总行中
	- 为每个组返回一个记录
	- 按分组字段进行排序后，`ORDER BY` 可以以汇总字段来进行排序
- `HAVING`：对汇总的 `GROUP BY` 结果进行过滤
	- 要求存在一个 `GROUP BY` 子句

> - `WHERE` 和 `HAVING` 都是用于过滤。
>
> - `HAVING` 适用于汇总的组记录；而 WHERE 适用于单个记录。

```sql
-- order by：DESC 递减
SELECT * FROM products ORDER BY prod_price DESC, prod_name ASC;
-- group by
SELECT cust_name, COUNT(cust_address) AS addr_num FROM Customers GROUP BY cust_name;
-- having
SELECT cust_name, COUNT(*) AS num
FROM Customers
WHERE cust_email IS NOT NULL
GROUP BY cust_name
HAVING COUNT(*) >= 1;
```

## More

### Char 与 Varchar 的区别

- char 表示定长字符串，长度是固定的，最多能存放的字符个数为 255，和编码无关；而 varchar 表示可变长字符串，长度是可变的，最多能存放的字符个数为 65532；
- 使用 char 时，如果插入数据的长度小于 char 的固定长度时，则用空格填充；
- 因为固定长度，char 的存取速度比 varchar 快很多，同时缺点是会占用多余空间，属于空间换时间；

### DROP、DELETE 与 TRUNCATE 的区别

<table>
<thead>
<tr>
<th style="text-align:center"></th>
<th style="text-align:center">DROP</th>
<th style="text-align:center">DELETE</th>
<th style="text-align:center">TRUNCATE</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center">SQL 语句类型</td>
<td style="text-align:center">DDL</td>
<td style="text-align:center">DML</td>
<td style="text-align:center">DDL</td>
</tr>
<tr>
<td style="text-align:center">回滚</td>
<td style="text-align:center">不可回滚</td>
<td style="text-align:center">可回滚</td>
<td style="text-align:center">不可回滚</td>
</tr>
<tr>
<td style="text-align:center">删除内容</td>
<td style="text-align:center">从数据库中 <strong>删除表</strong>，所有的数据行，索引和权限也会被删除</td>
<td style="text-align:center">表结构还在，删除表的 <strong>全部或者一部分数据行</strong></td>
<td style="text-align:center">表结构还在，删除表中的 <strong>所有数据</strong></td>
</tr>
<tr>
<td style="text-align:center">删除速度</td>
<td style="text-align:center">删除速度最快</td>
<td style="text-align:center">删除速度慢，需要逐行删除</td>
<td style="text-align:center">删除速度快</td>
</tr>
</tbody>
</table>

> 在不再需要一张表的时候，采用 `DROP`
> 在想删除部分数据行时候，用 `DELETE`
> 在保留表而删除所有数据的时候用 `TRUNCATE`

### UNION 与 UNION ALL 的区别

- `UNION` 用于把来自多个 `SELECT` 语句的结果组合到一个结果集合中，MySQL 会把结果集中 **重复的记录删掉**
- `UNION ALL`，MySQL 会把所有的记录返回，且效率高于 `UNION`
