---
post: layout
titile:  2020-4-4 MySql Lesson3
---
##  一、使用子查询
本章学习什么是子查询以及如何使用它们。
### 1 子查询

	SQL允许创建子查询(subquery），即嵌套在其他查询中的查询。<br>
	为什么需要子查询----因为通常存储一组关系数据是通过多个表完成的.
### 2 数据关系存在多个表中
我们将顾客按照编号存储顾客信息存储于customers.中

	mysql> select * from customers;
	+---------+----------------+---------------------+-----------+------------+----------+--------------+--------------+---------------------+
	| cust_id | cust_name      | cust_address        | cust_city | cust_state | cust_zip | cust_country | cust_contact | cust_email          |
	+---------+----------------+---------------------+-----------+------------+----------+--------------+--------------+---------------------+
	|   10001 | Coyote Inc.    | 200 Maple Lane      | Detroit   | MI         | 44444    | USA          | Y Lee        | ylee@coyote.com     |
	|   10002 | Mouse House    | 333 Fromage Lane    | Columbus  | OH         | 43333    | USA          | Jerry Mouse  | NULL                |
	|   10003 | Wascals        | 1 Sunny Place       | Muncie    | IN         | 42222    | USA          | Jim Jones    | rabbit@wascally.com |
	|   10004 | Yosemite Place | 829 Riverside Drive | Phoenix   | AZ         | 88888    | USA          | Y Sam        | sam@yosemite.com    |
	|   10005 | E Fudd         | 4545 53rd Street    | Chicago   | IL         | 54545    | USA          | E Fudd       | NULL                |
	+---------+----------------+---------------------+-----------+------------+----------+--------------+--------------+---------------------+
	5 rows in set (0.00 sec)

我们将订单信息（购买的物品）存储于orderitems中

	mysql> select * from orderitems;
	+-----------+------------+---------+----------+------------+
	| order_num | order_item | prod_id | quantity | item_price |
	+-----------+------------+---------+----------+------------+
	|     20005 |          1 | ANV01   |       10 |       5.99 |
	|     20005 |          2 | ANV02   |        3 |       9.99 |
	|     20005 |          3 | TNT2    |        5 |      10.00 |
	|     20005 |          4 | FB      |        1 |      10.00 |
	|     20006 |          1 | JP2000  |        1 |      55.00 |
	|     20007 |          1 | TNT2    |      100 |      10.00 |
	|     20008 |          1 | FC      |       50 |       2.50 |
	|     20009 |          1 | FB      |        1 |      10.00 |
	|     20009 |          2 | OL1     |        1 |       8.99 |
	|     20009 |          3 | SLING   |        1 |       4.49 |
	|     20009 |          4 | ANV03   |        1 |      14.99 |
	+-----------+------------+---------+----------+------------+
	11 rows in set (0.00 sec)

我们将订单时间存储于orders表中

	mysql> select * from orders;
	+-----------+---------------------+---------+
	| order_num | order_date          | cust_id |
	+-----------+---------------------+---------+
	|     20005 | 2005-09-01 00:00:00 |   10001 |
	|     20006 | 2005-09-12 00:00:00 |   10003 |
	|     20007 | 2005-09-30 00:00:00 |   10004 |
	|     20008 | 2005-10-03 00:00:00 |   10005 |
	|     20009 | 2005-10-08 00:00:00 |   10001 |
	+-----------+---------------------+---------+
	5 rows in set (0.00 sec)

现在的问题，我们要查找购买"TNT2"的所有客户信息，如何检索。大致步骤应该如下：

	》首先在orderitems 表中，找到prod_id为"TNT2"的order_num编号。
	》其次，通过order_num，在orders表中查找cust_id 满足以上order_num的值
	》最后，根据order_num的值，在customers 中获取信息。

这样，三条SELECT 语句我们可以合并为一条编写，即是使用子查询，最内的SELECT 语句最先执行。

	mysql> SELECT * FROM customers
	    -> WHERE cust_id IN (
				SELECT cust_id FROM orders WHERE order_num IN(
						SELECT order_num FROM orderitems WHERE prod_id = "TNT2"
															)
							);
	+---------+----------------+---------------------+-----------+------------+----------+--------------+--------------+------------------+
	| cust_id | cust_name      | cust_address        | cust_city | cust_state | cust_zip | cust_country | cust_contact | cust_email       |
	+---------+----------------+---------------------+-----------+------------+----------+--------------+--------------+------------------+
	|   10001 | Coyote Inc.    | 200 Maple Lane      | Detroit   | MI         | 44444    | USA          | Y Lee        | ylee@coyote.com  |
	|   10004 | Yosemite Place | 829 Riverside Drive | Phoenix   | AZ         | 88888    | USA          | Y Sam        | sam@yosemite.com |
	+---------+----------------+---------------------+-----------+------------+----------+--------------+--------------+------------------+
	2 rows in set (0.01 sec)

### 使用计算字段作为子查询
在以上的数据库中，加入需要显示customers表中每个客户的订单总数。顾客的基本信息存储于customers。订单与相应的客户ID存储在orders表中。<br>
为了执行这个操作，遵循下面的步骤。<br>
	
	》 从customers表中检索客户列表。
	》 对于检索出的每个客户，统计其在orders表中的订单数目。  # 使用select count(*)对表中的行进行计算

	mysql> SELECT cust_name,cust_state,
					(SELECT COUNT(*) FROM orders WHERE orders.cust_id = customers.cust_id) AS orders
	    -> FROM customers
	    -> ORDER BY cust_name;
	+----------------+------------+--------+
	| cust_name      | cust_state | orders |
	+----------------+------------+--------+
	| Coyote Inc.    | MI         |      2 |
	| E Fudd         | IL         |      1 |
	| Mouse House    | OH         |      0 |
	| Wascals        | IN         |      1 |
	| Yosemite Place | AZ         |      1 |
	+----------------+------------+--------+
	5 rows in set (0.02 sec)

## 二、 联结表
本章将介绍什么是联结，为什么要使用联结，如何编写使用联结的SELECT语句。

### 1 联结
SQL最强大的功能之一就是能在数据检索查询的执行中联结(join)表，联结是利用SQL的SELECT能执行的最重要的操作，很好地理解联结及其语法是学习SQL
的一个极为重要的组成部分。

### 2 基础知识

关系表<br>
	
	为避免不必要的数据重复，关系表的设计就是要保证把信息分解成多个表，一类数据一个表，。各表通过某些常用的值（即关系设计中的关系（relational）
	互相关联。
	典型的存储产品数据库，一般建立两个表，一个存储供应商信息，另一个存储产品信息。vendors表中包含所有供应商信息，每个供应商占一行，每个供应商
	具有唯一的标识。此标识称为主键（primary key).
	products表只存储产品信息，它除了存储供应商ID（vendors表的主键）外不存储其他供应商信息。vendors表的主键又叫作products的外键，它将vendors
	表与products表关联，利用供应商ID能从vendors表中找出响应供应商的详细信息。
	外键（foreign key)外键为某个表中的一列，它包含另一个表的主键值，定义了两个表之间的关系。

这样做的好处是：

	供应商信息不重复，从而不浪费时间和空间；
	如果供应商信息变动，可以只更新vendors表中的单个记录，相关表中的数据不同改动；
	由于数据无重复，显然数据是一致的，这使得处理数据更简单。
总之，关系数据可以有效地存储和方便地处理。因此，关系数据库的可伸缩性远比非关系数据库要好。

为什么要使用联结
	
	分解数据为多个表能更有效地存储，更方便地处理，并且有更大的可伸缩性。若数据存储在多个表中，怎样用单条SELECT语句检索出数据？
	答案是使用联结，简单地说，联结是一种机制，用来在一条SELECT语句中关联表，因此称之为联结。使用特殊的语法，可以联结多个表返回一组
	数据输出，联结在运行时关联表中正确的行。

	联结不是物理实体。换句话说，它在实际的数据库表中不存在。联结有MySQL根据需要建立，它存在于查询的执行当中。

### 3 创建联结
联结的创建非常简单，规定要联结的所有表以及它们如何关联即可。例如下面实例
	
	》 vendors中存储了供应商的信息
	mysql> select * from vendors;
	+---------+----------------+-----------------+-------------+------------+----------+--------------+
	| vend_id | vend_name      | vend_address    | vend_city   | vend_state | vend_zip | vend_country |
	+---------+----------------+-----------------+-------------+------------+----------+--------------+
	|    1001 | Anvils R Us    | 123 Main Street | Southfield  | MI         | 48075    | USA          |
	|    1002 | LT Supplies    | 500 Park Street | Anytown     | OH         | 44333    | USA          |
	|    1003 | ACME           | 555 High Street | Los Angeles | CA         | 90046    | USA          |
	|    1004 | Furball Inc.   | 1000 5th Avenue | New York    | NY         | 11111    | USA          |
	|    1005 | Jet Set        | 42 Galaxy Road  | London      | NULL       | N16 6PS  | England      |
	|    1006 | Jouets Et Ours | 1 Rue Amusement | Paris       | NULL       | 45678    | France       |
	+---------+----------------+-----------------+-------------+------------+----------+--------------+
	6 rows in set (0.00 sec)

	》products中存储了商品的具体信息
	mysql> select * from products;
	+---------+---------+----------------+------------+----------------------------------------------------------------+
	| prod_id | vend_id | prod_name      | prod_price | prod_desc                                                      |
	+---------+---------+----------------+------------+----------------------------------------------------------------+
	| ANV01   |    1001 | .5 ton anvil   |       5.99 | .5 ton anvil, black, complete with handy hook                  |
	| ANV02   |    1001 | 1 ton anvil    |       9.99 | 1 ton anvil, black, complete with handy hook and carrying case |
	| ANV03   |    1001 | 2 ton anvil    |      14.99 | 2 ton anvil, black, complete with handy hook and carrying case |
	| DTNTR   |    1003 | Detonator      |      13.00 | Detonator (plunger powered), fuses not included                |
	| FB      |    1003 | Bird seed      |      10.00 | Large bag (suitable for road runners)                          |
	| FC      |    1003 | Carrots        |       2.50 | Carrots (rabbit hunting season only)                           |
	| FU1     |    1002 | Fuses          |       3.42 | 1 dozen, extra long                                            |
	| JP1000  |    1005 | JetPack 1000   |      35.00 | JetPack 1000, intended for single use                          |
	| JP2000  |    1005 | JetPack 2000   |      55.00 | JetPack 2000, multi-use                                        |
	| OL1     |    1002 | Oil can        |       8.99 | Oil can, red                                                   |
	| SAFE    |    1003 | Safe           |      50.00 | Safe with combination lock                                     |
	| SLING   |    1003 | Sling          |       4.49 | Sling, one size fits all                                       |
	| TNT1    |    1003 | TNT (1 stick)  |       2.50 | TNT, red, single stick                                         |
	| TNT2    |    1003 | TNT (5 sticks) |      10.00 | TNT, red, pack of 10 sticks                                    |
	+---------+---------+----------------+------------+----------------------------------------------------------------+
	14 rows in set (0.00 sec)

若要分类检索每个供应商的产品及价格。可使用如下语句。

	mysql> select vend_name , prod_id,prod_name ,prod_price
	    -> from vendors,products
	    -> where vendors.vend_id = products.vend_id
	    -> order by vend_name;
	+-------------+---------+----------------+------------+
	| vend_name   | prod_id | prod_name      | prod_price |
	+-------------+---------+----------------+------------+
	| ACME        | DTNTR   | Detonator      |      13.00 |
	| ACME        | FB      | Bird seed      |      10.00 |
	| ACME        | FC      | Carrots        |       2.50 |
	| ACME        | SAFE    | Safe           |      50.00 |
	| ACME        | SLING   | Sling          |       4.49 |
	| ACME        | TNT1    | TNT (1 stick)  |       2.50 |
	| ACME        | TNT2    | TNT (5 sticks) |      10.00 |
	| Anvils R Us | ANV01   | .5 ton anvil   |       5.99 |
	| Anvils R Us | ANV02   | 1 ton anvil    |       9.99 |
	| Anvils R Us | ANV03   | 2 ton anvil    |      14.99 |
	| Jet Set     | JP1000  | JetPack 1000   |      35.00 |
	| Jet Set     | JP2000  | JetPack 2000   |      55.00 |
	| LT Supplies | FU1     | Fuses          |       3.42 |
	| LT Supplies | OL1     | Oil can        |       8.99 |
	+-------------+---------+----------------+------------+
	14 rows in set (0.00 sec)

使用联结时，多个查找表需要指定完全限定名表示。同时，使用联结时，应该保证所有的联结都有WHERE子句，若没有，输出结果将是笛卡尔积（两个表相乘）.
同理，应该保证WHERE子句的正确性，不正确的过滤条件将导致MySQL返回不正确的数据。

### 4 内部联结
目前为止所用的联结称为等值联结（equijion），它基于两个表之间的相等测试。这种联结也称为内部联结。其实，对于这种联结可以使用稍微不同的语法来明确指定
联结的类型。

	mysql> SELECT vend_name,prod_id,prod_name,prod_price
	    -> FROM vendors INNER JOIN products
	    -> ON vendors.vend_id = products.vend_id;
	+-------------+---------+----------------+------------+
	| vend_name   | prod_id | prod_name      | prod_price |
	+-------------+---------+----------------+------------+
	| Anvils R Us | ANV01   | .5 ton anvil   |       5.99 |
	| Anvils R Us | ANV02   | 1 ton anvil    |       9.99 |
	| Anvils R Us | ANV03   | 2 ton anvil    |      14.99 |
	| LT Supplies | FU1     | Fuses          |       3.42 |
	| LT Supplies | OL1     | Oil can        |       8.99 |
	| ACME        | DTNTR   | Detonator      |      13.00 |
	| ACME        | FB      | Bird seed      |      10.00 |
	| ACME        | FC      | Carrots        |       2.50 |
	| ACME        | SAFE    | Safe           |      50.00 |
	| ACME        | SLING   | Sling          |       4.49 |
	| ACME        | TNT1    | TNT (1 stick)  |       2.50 |
	| ACME        | TNT2    | TNT (5 sticks) |      10.00 |
	| Jet Set     | JP1000  | JetPack 1000   |      35.00 |
	| Jet Set     | JP2000  | JetPack 2000   |      55.00 |
	+-------------+---------+----------------+------------+
	14 rows in set (0.00 sec)

此中用法与上述稍有不同，不使用WHERE语句，在FROM子句中以INNER JOIN指定。传递给ON的实际条件与传递给WHERE的相同。

	ANSI SQL规范首选INNER JOIN语法，此外，尽管使用WHERE子句定义联结的确比较简单，
	但是使用明确的联结语法确保不会忘记联结条件，有时候这样做也能影响性能。

### 5 联结多个表
SQL对一条SELECT 语句中可以联结的表的数目没有限制。创建联结的基本规则也相同，首先列出所有表，然后定义表之间的关系。例如：查找编号为20005的
订单中的物品。订单物品存储在orderitems表中，每个产品按其产品ID存储，它引用products表中的产品。这些产品通过供应商ID联结到vendors表中响应
的供应商。因此需要联结3个表，查找出数据。

	mysql> SELECT prod_name,vend_name,prod_price,quantity
    -> FROM orderitems,products,vendors
    -> WHERE products.vend_id = vendors.vend_id
    -> AND orderitems.prod_id = products.prod_id
    -> AND order_num = 20005;
	+----------------+-------------+------------+----------+
	| prod_name      | vend_name   | prod_price | quantity |
	+----------------+-------------+------------+----------+
	| .5 ton anvil   | Anvils R Us |       5.99 |       10 |
	| 1 ton anvil    | Anvils R Us |       9.99 |        3 |
	| TNT (5 sticks) | ACME        |      10.00 |        5 |
	| Bird seed      | ACME        |      10.00 |        1 |
	+----------------+-------------+------------+----------+
	4 rows in set (0.00 sec)

## 四、创建高级联结
本章将讲解另外一些联结类型（包括它们的含义和使用方法），介绍如何对被联结的表使用表名和聚集函数。

### 1 使用表别名
回忆一下给列起别名的语法。

	SELECT CONCAT(RTRIM(vend_nam),"(",Rtrim(vend_country),")") AS vend_titile
	FROM vendors
	ORDER BY vend_name;

	mysql> SELECT Concat(RTrim(vend_name),"(",RTrim(vend_country),")") AS vend_titile
	    -> FROM vendors
	    -> ORDER BY vend_name;
	+------------------------+
	| vend_titile            |
	+------------------------+
	| ACME(USA)              |
	| Anvils R Us(USA)       |
	| Furball Inc.(USA)      |
	| Jet Set(England)       |
	| Jouets Et Ours(France) |
	| LT Supplies(USA)       |
	+------------------------+
	6 rows in set (0.00 sec)

别名除了用于列名和计算字段外，SQL还允许给表名起别名。这样做有两个主要理由：
	
	》 缩短SQL语句
	》 允许在单条SELECT 语句中多次使用相同的表。

例如，将上一章联结的语句改成使用别名：
	mysql> SELECT cust_name,cust_contact
    -> FROM customers AS c,orders AS o,orderitems AS oi
    -> WHERE c.cust_id = o.cust_id
    -> AND oi.order_num = o.order_num
    -> AND prod_id = "TNT2";
	+----------------+--------------+
	| cust_name      | cust_contact |
	+----------------+--------------+
	| Coyote Inc.    | Y Lee        |2
	| Yosemite Place | Y Sam        |
	+----------------+--------------+
	2 rows in set (0.00 sec)
注意，表别名只在查询执行中使用，与列别名不一样，表别名不返回客户机。

### 2 自联结
使用自联结主要处理条件为表中数据，查找与该数据相关的整个表中其他数据。例如我们发现某个物品(prid_id)有问题，因此想知道
生成该物品的供应商的其他物品是否也存在这些问题，若使用子查询，步骤如下：
	
	》 首先通过prod_id的条件（例如“DTNTR")查找到供货商的vend_id
	》 然后通过vend_id的条件查找该表中的prod_id,prod_name等信息。

	SELECT prod_id ,prod_name
	FROM products
	WHERE vend_id = (SELECT vend_id
					 FROM products
					 WHERE prod_id = "DTNTR");

	mysql> SELECT prod_id,prod_name
	    -> FROM products
	    -> WHERE vend_id = (SELECT vend_id
	    ->                  FROM products
	    ->                  WHERE prod_id = "DTNTR");
	+---------+----------------+
	| prod_id | prod_name      |
	+---------+----------------+
	| DTNTR   | Detonator      |
	| FB      | Bird seed      |
	| FC      | Carrots        |
	| SAFE    | Safe           |
	| SLING   | Sling          |
	| TNT1    | TNT (1 stick)  |
	| TNT2    | TNT (5 sticks) |
	+---------+----------------+
	7 rows in set (0.00 sec)

若使用联结，此种在同一个表中的联结叫《自联结》，需要使用表别名。
	
	SELECT p1.prod_id,p1.prod_name
	FROM products AS p1,products AS p2
	WHERE p1.vend_id = p2.vend_id
		  AND p2.prod_id = "DTNTR";
	
	mysql> SELECT p1.prod_id,p1.prod_name
	    -> FROM products
	    -> AS p1,products AS p2
	    -> WHERE p1.vend_id = p2.vend_id
	    -> AND p2.prod_id = "DTNTR";
	+---------+----------------+
	| prod_id | prod_name      |
	+---------+----------------+
	| DTNTR   | Detonator      |
	| FB      | Bird seed      |
	| FC      | Carrots        |
	| SAFE    | Safe           |
	| SLING   | Sling          |
	| TNT1    | TNT (1 stick)  |
	| TNT2    | TNT (5 sticks) |
	+---------+----------------+
	7 rows in set (0.00 sec)

### 3 自然联结
无论何时对表进行联结，应该至少有一个列出现在不止一个表中（被联结的列）。标准的联结（前面介绍的内部联结）返回所有数据，甚至
相同的列多次出现。<br>
自然联结排除多次出现，使每个列只返回一次。这一般是通过对表进行通配符(select *），对所有其他表的列使用明确的子集完成的。
例如： 查找满足prod_id = "FB"的数据信息。

	mysql> select *
    -> from customers as c
    -> ,orders as o
    -> ,orderitems as oi
    -> where c.cust_id = o.cust_id
    -> and o.order_num = oi.order_num
    -> and prod_id = "FB";
	+---------+-------------+----------------+-----------+------------+----------+--------------+--------------+-----------------+-----------+---------------------+---------+-----------+------------+---------+----------+------------+
	| cust_id | cust_name   | cust_address   | cust_city | cust_state | cust_zip | cust_country | cust_contact | cust_email      | order_num | order_date          | cust_id | order_num | order_item | prod_id | quantity | item_price |
	+---------+-------------+----------------+-----------+------------+----------+--------------+--------------+-----------------+-----------+---------------------+---------+-----------+------------+---------+----------+------------+
	|   10001 | Coyote Inc. | 200 Maple Lane | Detroit   | MI         | 44444    | USA          | Y Lee        | ylee@coyote.com |     20005 | 2005-09-01 00:00:00 |   10001 |     20005 |          4 | FB      |        1 |      10.00 |
	|   10001 | Coyote Inc. | 200 Maple Lane | Detroit   | MI         | 44444    | USA          | Y Lee        | ylee@coyote.com |     20009 | 2005-10-08 00:00:00 |   10001 |     20009 |          1 | FB      |        1 |      10.00 |
	+---------+-------------+----------------+-----------+------------+----------+--------------+--------------+-----------------+-----------+---------------------+---------+-----------+------------+---------+----------+------------+

可以看出，存在重复的列，mysql无法自动消除重复的列，需要手动明确。如下例所示：

	mysql> select c.*,o.order_num,o.order_date,oi.prod_id,oi.quantity,oi.item_price
	    -> from customers as c,
	    ->      orders as o,
	    ->      orderitems as oi
	    -> where c.cust_id = o.cust_id
	    ->       and oi.order_num = o.order_num
	    ->       and oi.prod_id = "FB";
	+---------+-------------+----------------+-----------+------------+----------+--------------+--------------+-----------------+-----------+---------------------+---------+----------+------------+
	| cust_id | cust_name   | cust_address   | cust_city | cust_state | cust_zip | cust_country | cust_contact | cust_email      | order_num | order_date          | prod_id | quantity | item_price |
	+---------+-------------+----------------+-----------+------------+----------+--------------+--------------+-----------------+-----------+---------------------+---------+----------+------------+
	|   10001 | Coyote Inc. | 200 Maple Lane | Detroit   | MI         | 44444    | USA          | Y Lee        | ylee@coyote.com |     20005 | 2005-09-01 00:00:00 | FB      |        1 |      10.00 |
	|   10001 | Coyote Inc. | 200 Maple Lane | Detroit   | MI         | 44444    | USA          | Y Lee        | ylee@coyote.com |     20009 | 2005-10-08 00:00:00 | FB      |        1 |      10.00 |
	+---------+-------------+----------------+-----------+------------+----------+--------------+--------------+-----------------+-----------+---------------------+---------+----------+------------+
	2 rows in set (0.00 sec)

### 4 外部联结
许多联结将一个表中的行与另一个表中的行相关联。但有时候会需要包含没有关联行的那些行。例如，可能需要使用联结来完成以下工作：
	
	》 对每个客户下了多少订单进行计数，包括那些至今尚未下订单的客户；
	》 列出所有产品以及订购数量，包括没有人订购的产品；
	》 计算平均销售规模，包括那些至今尚未下订单的客户。

在上述例子中，联结包含了那些在相关表中没有关联行的行。这种类型的联结称为外部联结。<br>
我们复习下上面的简单的内部联结，它检索所有客户及其订单：

	SELECT customers.cust_id , orders.order_num 
	FROM customers INNER JOIN orders
	ON customers.cust_id = orders.cust_id;

	mysql> SELECT customers.cust_id ,orders.order_num
	    -> FROM customers INNER JOIN orders
	    -> ON customers.cust_id = orders.cust_id;
	+---------+-----------+
	| cust_id | order_num |
	+---------+-----------+
	|   10001 |     20005 |
	|   10001 |     20009 |
	|   10003 |     20006 |
	|   10004 |     20007 |
	|   10005 |     20008 |
	+---------+-----------+
	5 rows in set (0.00 sec)


外部联结语法类似。为了检索所有客户，包括那些没有订单的客户，可如下进行：
	
	SELECT customers.cust_id,order.order_num
	FROM customers LEFT OUTER JOIN orders
	ON customers.cust_id = orders.cust_id;

	mysql> SELECT customers.cust_id,orders.order_num
	    -> FROM customers LEFT OUTER JOIN orders
	    -> ON customers.cust_id = orders.cust_id;
	+---------+-----------+
	| cust_id | order_num |
	+---------+-----------+
	|   10001 |     20005 |
	|   10001 |     20009 |
	|   10002 |      NULL |
	|   10003 |     20006 |
	|   10004 |     20007 |
	|   10005 |     20008 |
	+---------+-----------+
	6 rows in set (0.00 sec)

### 5 使用带聚集函数的联结
在联结中使用聚集函数，可以实现复杂的数据检索及汇总功能，例如，我们检索所有客户及每个客户所下的订单数，可使用COUNT()函数嵌入联结子句中完成。

	SELECT customers.cust_name,
		   customers.cust_id,
		   COUNT(orders.order_num) AS num_ord
	FROM customers INNER JOIN orders
	ON   customers.cust_id = orders.cust_id
	GROUP BY customers.cust_id;

	mysql> SELECT customers.cust_name,
	    ->        customers.cust_id,
	    ->        COUNT(orders.order_num) AS num_ord
	    -> FROM customers INNER JOIN orders
	    -> ON customers.cust_id = orders.cust_id
	    -> GROUP BY customers.cust_id;
	+----------------+---------+---------+
	| cust_name      | cust_id | num_ord |
	+----------------+---------+---------+
	| Coyote Inc.    |   10001 |       2 |
	| Wascals        |   10003 |       1 |
	| Yosemite Place |   10004 |       1 |
	| E Fudd         |   10005 |       1 |
	+----------------+---------+---------+
	4 rows in set (0.00 sec)
分析：
	
	》 此SELECT 语句使用INNER JOIN将customers 和orders表互相关联。
	》 GROUP BY 子句按客户分组数据，因此，函数调用COUNT(orders.order_num) 对每个客户的订单计数，将它作为num_ord 返回。

### 6 使用联结和联结条件
有关使用联结的总结：
	
	》 注意所使用的联结类型。一般我们使用内部联结，但使用外部联结也是有效的。
	》 保证使用正确的联结条件，否则将返回不正确的数据。
	》 应该总是提供联结条件，否则会得到笛卡尔积。
	》 在一个联结中可以包含多个表，甚至对于每个联结可以采用不同的联结类型。虽然这样做事合法的，一般也很有用，但应该在一起测试他们前，分别测试每个联结。