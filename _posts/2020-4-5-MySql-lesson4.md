---
post: layout
titile:  2020-4-4 MySql Lesson3
---
##  一、使用存储过程 
本章学习什么是存储过程，为什么要使用存储过程以及如何使用存储过程，并且介绍创建和使用存储过程的基本语法。<br>
之前学习的SQL语句都是针对一个或多个表的单条语句。并非所有操作都那么简单。经常会有一个完整的操作需要多条语句才能完成。
例如，考虑一下的情形。

	》为了处理订单，需要核对以保证库存中有相应的物品。
	》如果库存有物品，这些物品需要预定以便不将它们再卖给别的人
	》库存中没有的物品需要订购，这需要与供应商进行某种交互
	》关于哪些物品入库（并且可以立即发货）和哪些物品退订，需要通知相应的客户。

执行上述的要求，需要针对许多表的多条MySQL语句。因此我们需要创建存储过程。<br>
存储过程简单来说，就是为以后的使用而保存的一条或多条MySQL语句的集合，可将其视为批文件。

### 1 为什么要使用存储过程
下面是一些使用理由：
	
	》通过把处理封装在容易使用的单元，简化复杂的操作
	》由于不要求反复建立一系列处理步骤，这保证了数据的完整性。
	》简化对变动的管理。如果表名、列名或业务逻辑有变化，只需要更改存储过程的代码。
	》提高性能。因为使用存储过程比使用单独的SQL语句要快。
### 2 创建存储过程

	mysql> DELIMITER //
	mysql> CREATE PROCEDURE productpricing()
	    -> BEGIN
	    ->  SELECT AVG(prod_price) AS priceaverage
	    -> FROM products;
	    -> END //
	Query OK, 0 rows affected (0.20 sec)
	
其中由于我们在mysql终端上输入此语句，因此需要临时更改命令行使用程序的语句执行分隔符，

	DELIMITER //       # 告诉命令行使用程序用//作为结束分隔符。
	DELIMITER ;        # 恢复以前的；作为结束分隔符。

### 3 执行存储过程
使用CALL 语句直接执行存储过程，若存储过程带参数需要指定。我们执行上述创建的不带参数的存储过程。

	mysql> CALL productpricing()；
	+--------------+
	| priceaverage |
	+--------------+
	|    16.133571 |
	+--------------+
	1 row in set (0.01 sec)
	
	Query OK, 0 rows affected (0.01 sec)

### 4 删除存储过程
存储过程在创建之后，被保存在服务器上以供使用，直至被删除。删除命令类似与删除表。从服务器中删除存储过程。可使用以下语句

	DROP PROCEDURE productpricing;  #只用给出函数名即可，不需要();
	DROP PROCEDURE IF EXISTS productpricing; #若存在则删除，否则不删除。

### 5 使用参数
一般情况下，存储过程并不现实结果，而是把结果返回给指定的变量（用于存储临时数据）。

	mysql> CREATE PROCEDURE productpricing(OUT p1 DECIMAL(8.2),
	    -> OUT ph DECIMAL(8,2),
	    -> OUT pa DECIMAL(8,2))
	    -> BEGIN
	    -> SELECT Min(prod_price) INTO p1 FROM products;
	    -> SELECT Max(prod_price) INTO ph FROM products;
	    -> SELECT Avg(prod_price) INTO pa FROM products;
	    -> END//
	Query OK, 0 rows affected (0.07 sec)

mysql支持IN（传递给存储过程）、OUT（从存储过程传出）和INOUT（对存储过程传入和传出）类型的参数。存储过程的代码位于BEGIN和END语句内。
创建完毕后，我们可以调用存储过程。

	mysql> CALL productpricing(@pricelow,
	    -> @pricehigh,
	    -> @priceaverage);//
	Query OK, 1 row affected, 1 warning (0.00 sec)

调用时，必须参数要保持一致，该条语句并不现实任何数据，它返回以后可以现实的变量。为了现实检索出的评价价格，可执行如下命令：

	mysql> DELIMITER ;       #恢复；作为结束符
	mysql> SELECT @priceaverage;
	+---------------+
	| @priceaverage |
	+---------------+
	|         16.13 |
	+---------------+
	1 row in set (0.00 sec)

为了获得三个参数的值，我们也可以使用如下语句：

	mysql> SELECT @pricehigh,@pricelow,@priceaverage;
	+------------+-----------+---------------+
	| @pricehigh | @pricelow | @priceaverage |
	+------------+-----------+---------------+
	|      55.00 |         3 |         16.13 |
	+------------+-----------+---------------+
	1 row in set (0.00 sec)

### 6 接受参数的存储过程
我们使用IN用于接受外部输入的存储过程参数，创建格式如下，我们要求ordertotal接受订单号并返回该订单的总数。
	
	mysql> DELIMITER //
	mysql> CREATE PROCEDURE ordertotal(
	    -> IN onumber INT,
	    -> OUT ototal DECIMAL(8,2))
	    -> BEGIN
	    -> SELECT Sum(item_price*quantity)
	    -> FROM orderitems
	    -> WHERE order_num = onumber
	    -> INTO ototal;
	    -> END//
	Query OK, 0 rows affected (0.11 sec)

为调用这个新的存储过程，可使用以下语句

	mysql> CALL ordertotal(20005,@total)//
	Query OK, 1 row affected (0.00 sec)

显示存储过程值

	mysql> SELECT @total//
	+--------+
	| @total |
	+--------+
	| 149.87 |
	+--------+
	1 row in set (0.00 sec)

### 7 检查存储过程
为显示用来创建一个存储过程的CREATE语句，使用SHOW CREATE PROCEDURE语句：

	SHOW CREATE PROCEDURE ordertotal;
	
