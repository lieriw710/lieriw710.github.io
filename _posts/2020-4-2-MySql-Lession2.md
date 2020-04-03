-----------
post:layout
title:2020-3-22 MySQL基本用法
-----------
##一、创建计算字段

### 1 计算字段

存储在数据库中的数据一般不是应用程序所需要的格式，下面举几个例子。<br>

	>如果想在一个字段中既显示公司名，又显示公司的地址，但这两个信息一般包含在不同的表列中。
	>城市、州和邮编存储在不同的列中（应该这样），但邮件标签打印程序却需要把它们作为一个恰当的字段检索出来。
	>列数据是大小写混合的，但报表程序需要把所有数据按大些表示出来。
	>物品订单表存储物品的价格和数量，但不需要存储每个物品的总价格（用价格乘以数量即可）。为打印发票，需要物品的总价格。
	>需要根据表数据进行总数、平均数计算或其他计算。
	>
在上述每个例子中，存储在表中的数据都不是程序所需要的。我们需要直接从数据库中检索除转换、计算或格式化过的数据；而不是检索出数据，然后再在客户机应用程序或报告程序中重新格式化。<BR>
这就是计算字段发挥作用的所在。字段（field）基本上于列（column）的意思相同，经常互换使用，不过数据列一般成为列，而术语字段通常用在计算字段的连接上。<br>
重要的是要注意到，只有数据库知道SELECT语句中哪些列是实际的表列，哪些列是计算字段。从客户机的角度来看，计算字段的数据是以与其他列的数据相同的的方式返回的。

###2 拼接字段

为了说明如何使用计算字段，举一个创建由两列组成的标题的简单例子。<br>
vendors表包含供应商名和位置信息，假如要生产一个供应商报告，需要在供应商的名字中按照name(location)这样的格式列出供应商的位置。<br>
此报表需要单个值，而表中数据存储在两个列vend_name和vend_country中。<br>
解决办法是把两个列拼接起来，在MySQL的SELECT语句中，可使用Concat（）函数来拼接两个列。

	MYSQL> SELECT CONCAT (VEND_NAME,"(",VEND_COUNTRY,")")
    -> FROM VENDORS
    -> ORDER BY VEND_NAME;
	+-----------------------------------------+
	| CONCAT (vend_name,"(",vend_country,")") |
	+-----------------------------------------+
	| ACME(USA)                               |
	| Anvils R Us(USA)                        |
	| Furball Inc.(USA)                       |
	| Jet Set(England)                        |
	| Jouets Et Ours(France)                  |
	| LT Supplies(USA)                        |
	+-----------------------------------------+
	6 rows in set (0.02 sec)

从上面可以看出，Concat()需要一个或多个指定的串，各个串之间用逗号分隔。若要删除数据右侧多余的空格来整理数据，可使用MYSQL的RTRIM(）函数来完成。如下：
	
	SELECT CONCAT(RTRIM(vend_name),'(',RTRIM(vend_country),')')
	FROM vendors
	ORDER BY vend_name;

RTRIM()函数去掉右边的所有空格。通过使用RTRIM(),各个列都进行了整理。同理，LTRIM()去掉串左边的空格，以及TRIM()去掉串左右两边的空格。

###3 使用别名

从前面的输出中可以看到，SELECT语句拼接地址字段工作得很好。但此计算列的名字是什么呢？实际上它没有名字，它只是一个值。如果仅在SQL查询工具中查看以下结果，这样没有什么不好，但是，一个
命名的列不能用于客户机中。<br>
为了解决这个问题，SQL支持列别名（alias）是一个字段或值得替代名。别名用AS关键字赋予。

	mysql> SELECT CONCAT(vend_name,"(",vend_city,")")
	    -> AS vend_title
	    -> FROM vendors
	    -> ORDER BY vend_name;
	+-------------------------+
	| vend_title              |
	+-------------------------+
	| ACME(Los Angeles)       |
	| Anvils R Us(Southfield) |
	| Furball Inc.(New York)  |
	| Jet Set(London)         |
	| Jouets Et Ours(Paris)   |
	| LT Supplies(Anytown)    |
	+-------------------------+
	6 rows in set (0.00 sec)

###4 执行算术计算

orderitems表中包含每个订单中的各项物品。

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

若要汇总计算物品的价格（单价乘以订购数量）。可使用AS 将列表的计算术重命名。例如：

	mysql> SELECT order_num,order_item,prod_id,quantity,item_price,quantity*item_price AS expanded_price
	    -> FROM orderitems;
	+-----------+------------+---------+----------+------------+----------------+
	| order_num | order_item | prod_id | quantity | item_price | expanded_price |
	+-----------+------------+---------+----------+------------+----------------+
	|     20005 |          1 | ANV01   |       10 |       5.99 |          59.90 |
	|     20005 |          2 | ANV02   |        3 |       9.99 |          29.97 |
	|     20005 |          3 | TNT2    |        5 |      10.00 |          50.00 |
	|     20005 |          4 | FB      |        1 |      10.00 |          10.00 |
	|     20006 |          1 | JP2000  |        1 |      55.00 |          55.00 |
	|     20007 |          1 | TNT2    |      100 |      10.00 |        1000.00 |
	|     20008 |          1 | FC      |       50 |       2.50 |         125.00 |
	|     20009 |          1 | FB      |        1 |      10.00 |          10.00 |
	|     20009 |          2 | OL1     |        1 |       8.99 |           8.99 |
	|     20009 |          3 | SLING   |        1 |       4.49 |           4.49 |
	|     20009 |          4 | ANV03   |        1 |      14.99 |          14.99 |
	+-----------+------------+---------+----------+------------+----------------+
	11 rows in set (0.01 sec)
	
	mysql> select *,quantity*item_price AS expanded_price
	    -> from orderitems;
	+-----------+------------+---------+----------+------------+----------------+
	| order_num | order_item | prod_id | quantity | item_price | expanded_price |
	+-----------+------------+---------+----------+------------+----------------+
	|     20005 |          1 | ANV01   |       10 |       5.99 |          59.90 |
	|     20005 |          2 | ANV02   |        3 |       9.99 |          29.97 |
	|     20005 |          3 | TNT2    |        5 |      10.00 |          50.00 |
	|     20005 |          4 | FB      |        1 |      10.00 |          10.00 |
	|     20006 |          1 | JP2000  |        1 |      55.00 |          55.00 |
	|     20007 |          1 | TNT2    |      100 |      10.00 |        1000.00 |
	|     20008 |          1 | FC      |       50 |       2.50 |         125.00 |
	|     20009 |          1 | FB      |        1 |      10.00 |          10.00 |
	|     20009 |          2 | OL1     |        1 |       8.99 |           8.99 |
	|     20009 |          3 | SLING   |        1 |       4.49 |           4.49 |
	|     20009 |          4 | ANV03   |        1 |      14.99 |          14.99 |
	+-----------+------------+---------+----------+------------+----------------+
	11 rows in set (0.00 sec)

直接测试数据。

	可在mysql终端中通过SELECT 直接计算数据，例如
	mysql> select 2*3;
	+-----+
	| 2*3 |
	+-----+
	|   6 |
	+-----+
	1 row in set (0.00 sec)

	mysql> select ltrim("   bac   ");
	+--------------------+
	| ltrim("   bac   ") |
	+--------------------+
	| bac                |
	+--------------------+
	1 row in set (0.00 sec)

## 二、使用数据处理函数

学习什么是函数，MySQL支持何种函数，以及如何使用这些函数。

### 1 函数

	SQL支持利用函数来处理数据，函数一般在数据上执行的，它给数据的转换和处理提供了方便。<br>
	SQL中函数没有SQL语句的可移植性强，多数SQL语句是可移植的，在不同平台上SQL之间差异较小，而函数的可移植性却不强，每个平台提供的函数差异较大。
	因此若要决定使用函数，应该保证做好代码注释，以便查看SQL代码的含义。

### 2 使用函数

大多数SQL实现支持以下类型的函数

	>用于处理文本串（如删除或填充值，转换值为大写或小写）的文本函数。
	>用于在数组数据上进行算术操作（如返回绝对值，进行代数运算）的数组函数。
	>用于处理日期和时间并从这些值中提取特定成分（例如，返回两个日期之差，检查日期有效性等）的日期和时间函数。
	>返回DBMS正使用的特殊信息（如返回用户登录信息，检查版本细节）的系统函数。

### 3 文本处理函数

主要有以下：
	>>> upper（）               转换为大写的函数
	>left()                    返回串左边的字符
	>length()                  返回串的长度
	>locate()                  找到串的一个子串
	>ltrim()                   去掉串左边的空格
	>right()                   返回串右边的字符
	>rtirm()                   去掉串右边的空格
	>soundex()                 返回串的soundex值
	>substring()               返回子串的字符
	>lower()                   转换为小写的函数
	
使用Upper()函数的例子

	mysql> SELECT vend_name,UPPER(vend_name) AS vend_name_upper
	    -> FROM vendors
	    -> ORDER BY vend_name;
	+----------------+-----------------+
	| vend_name      | vend_name_upper |
	+----------------+-----------------+
	| ACME           | ACME            |
	| Anvils R Us    | ANVILS R US     |
	| Furball Inc.   | FURBALL INC.    |
	| Jet Set        | JET SET         |
	| Jouets Et Ours | JOUETS ET OURS  |
	| LT Supplies    | LT SUPPLIES     |
	+----------------+-----------------+
	6 rows in set (0.01 sec)


 soundex()函数是一个将任何文本串转换为描述其语音表示的字母数字模式的算法。soundex考虑了类似的发音字符和音节，使得能对串进行发音比较而不是字母比较。

	mysql> select cust_name,cust_contact
	    -> from customers
	    -> where soundex(cust_contact) = soundex("Y Lie");
	+-------------+--------------+
	| cust_name   | cust_contact |
	+-------------+--------------+
	| Coyote Inc. | Y Lee        |
	+-------------+--------------+
	1 row in set (0.01 sec)

### 4 日期和时间处理函数

日期和时间采用相应的数据类型和特殊的格式存储，以便能快速和有效地排序或过滤，并且节省物理存储空间。<br>
一般，应用程序不使用用来存储日期和时间的格式，因此日期和时间总是被用来读取、统计和处理这些值。
 
	AddDate()    	增加一个日期（天、周等）
	AddTime()   	增加一个时间（时、分等）
	CurDate()    	返回当前日期
	CurTime()    	返回当前时间
	Date()       	返回日期时间的日期部分
	DateDiff()   	计算两个日期之差
	Date_Add()   	高度灵活的日期运算函数
	Date_Format() 	返回一个格式化的日期或时间串
	Day() 			返回一个日期的天数部分
	DayOfWeek() 	对于一个日期，返回对应的星期几
	Hour() 			返回一个时间的小时部分
	Minute() 		返回一个时间的分钟部分
	Month() 		返回一个日期的月份部分
	Now() 			返回当前日期和时间
	Second() 		返回一个时间的秒部分
	Time() 			返回一个日期时间的时间部分
	Year() 			返回一个日期的年份部分

例如检索时间，需要使用DATE()，例如检索orders表中下单时间为2005-09-01的订单。则写为

	mysql> SELECT cust_id,order_num
	    -> FROM orders
	    -> WHERE Date(order_date) = "2005-09-1";
	+---------+-----------+
	| cust_id | order_num |
	+---------+-----------+
	|   10001 |     20005 |
	+---------+-----------+
	1 row in set (0.00 sec)

###5 数值处理函数

数值处理函数仅处理数值数据。这些函数一般主要用于代数、三角或几何运算，因此没有串或日期时间处理函数的使用那么频繁<br>
但在主要DBMS的函数中，数值函数是最一致最统一的函数。下面是一些常用的数值处理函数。

	Abs()    返回一个数的绝对值
	Cos()    返回一个角度的余弦
	Exp()    返回一个数的指数值
	Mod()    返回除操作的余数
	Pi()     返回圆周率
	Rand()   返回一个随机数
	Sin()    返回一个角度的正弦
	Sqrt()   返回一个数的平方根
	Tan()    返回一个角度的正切

##三 汇总数据

本章介绍什么是SQL的聚集函数以及如何利用它们汇总表的数据。

###1 聚集函数

我们经常需要汇总数据而不用把它们实际检索出来，为此MySQL提供了专门的函数，使用这些函数，Mysql查询可用于检索数据，以便分析和报表生成。这种类型的检索例子有以下几种：

	> 确定表中行数（或者满足某个条件或包含某个特定值得行数）
	> 获得表中行组的和
	> 找出表列（或所有行或某些特定的行）的最大值、最小值和平局值

为了方便这种类型的检索，Mysql给出了5个聚集函数（aggregate functon）。----运行在行组行，计算和返回单个值得函数。

	> AVG()                返回谋列的平均值
	> COUNT()              返回某列的行数
	> MAX()                返回某列的最大值
	> MIN()                返回某列的最小值
	> SUM()                返回某列值之和

例如，对表products。其数据如下：

	mysql> SELECT * FROM products;
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

使用AVG()函数可返回所有产品的平均值

	mysql> select avg(prod_price) as avg_price
    -> from products;
	+-----------+
	| avg_price |
	+-----------+
	| 16.133571 |
	+-----------+
	1 row in set (0.03 sec)

也可配合where语句实现部分行数的计算

	mysql> select avg(prod_price) as avg_price
	    -> from products
	    -> where vend_id = 1003;
	+-----------+
	| avg_price |
	+-----------+
	| 13.212857 |
	+-----------+
	1 row in set (0.00 sec)

使用count（）计算行数，count(*)表示计算所有行，包括NULL的值。

	mysql> SELECT COUNT(*) AS CUST_EMAIL
	    -> FROM CUSTOMERS;
	+------------+
	| CUST_EMAIL |
	+------------+
	|          5 |
	+------------+
	1 row in set (0.03 sec)

count()函数中添加参数，则统计数不包括NULL的值。

	mysql> SELECT count(cust_email) as num_cust
	    -> FROM customers;
	+----------+
	| num_cust |
	+----------+
	|        3 |
	+----------+
	1 row in set (0.00 sec)

max()返回指定列中的最大值，max()要求指定列名，如下所示：

	mysql> select max(prod_price) as max_price
	    -> from products;
	+-----------+
	| max_price |
	+-----------+
	|     55.00 |
	+-----------+
	1 row in set (0.02 sec)

###2 组合聚集函数
以上的聚集函数只涉及单个函数，SELECT语句可根据需要包含多个聚集函数，请看下面的例子：

	mysql> select count(*) as num_items,
	    -> max(prod_price) as max_price,
	    -> min(prod_price) as min_price,
	    -> avg(prod_price) as avg_price,
	    -> sum(prod_price) as sum_price
	    -> from products;
	+-----------+-----------+-----------+-----------+-----------+
	| num_items | max_price | min_price | avg_price | sum_price |
	+-----------+-----------+-----------+-----------+-----------+
	|        14 |     55.00 |      2.50 | 16.133571 |    225.87 |
	+-----------+-----------+-----------+-----------+-----------+
	1 row in set (0.00 sec)
	
##四 分组数据
本章将介绍如何分组数据，一遍能汇总表内容的子集。这涉及两个新SELECT语句子句，分别是GROUP BY子句和HAVING子句

###1 数据分组
使用SELECT 语句中的GROPU BY子句，可以对数据进行分组，例如实现对vend_id 的数目进行统计数量，可采用
	mysql> SELECT vend_id,count(*) as num_prods
	    -> FROM products
	    -> GROUP BY vend_id;
	+---------+-----------+
	| vend_id | num_prods |
	+---------+-----------+
	|    1001 |         3 |
	|    1002 |         2 |
	|    1003 |         7 |
	|    1005 |         2 |
	+---------+-----------+
	4 rows in set (0.02 sec)

###2 GROUP BY 的一些基本用法

	 	GROUP BY子句可以包含任意数目的列。这使得能对分组进行嵌套，
	  	为数据分组提供更细致的控制。
	 	如果在GROUP BY子句中嵌套了分组，数据将在最后规定的分组上
		进行汇总。换句话说，在建立分组时，指定的所有列都一起计算
		（所以不能从个别的列取回数据）。
	 	GROUP BY子句中列出的每个列都必须是检索列或有效的表达式
		（但不能是聚集函数）。如果在SELECT中使用表达式，则必须在
		GROUP BY子句中指定相同的表达式。不能使用别名。
	 	除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子
		句中给出。
	 	如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列
		中有多行NULL值，它们将分为一组。
		 GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。

###3 过滤分组
使用HAVING语句来实现过滤分组，语法与WHERE基本一致，区别在于WHERE只能过滤行，不能过滤分组，
HAVING 语句可以实现所有的WHERE语句的功能。
例如

	mysql> select cust_id,count(*) as num_orders
	    -> from orders
	    -> group by cust_id
	    -> having num_orders>=2;
	+---------+------------+
	| cust_id | num_orders |
	+---------+------------+
	|   10001 |          2 |
	+---------+------------+
	1 row in set (0.00 sec)

过滤分组一般要配合分组过滤后进行，因此having 需要在group by 子句后。

###4 SELECT子句顺序
回忆下之前学习的SELECT 子句，一般有如下子句可用，其顺序依次往下

	子句         	说明                                 	是否必须使用
	SELECT      	 要返回的列或表达式                    	是
	FROM         	从中检索数据的表                      	仅在从表选择数据时使用
	WHERE        	行级过滤                             	否
	GROUP BY     	分组说明                             	否
	ORDER BY     	输出排序顺序                         		否
	LIMIT        	要检索的行数                          	否