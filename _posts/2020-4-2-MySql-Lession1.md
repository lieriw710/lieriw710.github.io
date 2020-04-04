---
post: layout
titile:  2020-3-22 MySQL安装
---
# 1 解压安装
1、在MySQL官方网站下载社区解压版本压缩包，解压到E盘其中一个目录下。<br>
2、将文件下bin目录设置环境变量path下。<br>
3、在bin下创建my-default.ini文件（注意，不要命名为my.ini）。编辑内容如下：

	[mysqld]
	# 设置3307端口
	port = 3307
	# 设置mysql的安装目录
	basedir=D:/Program Files/mysql-5.7.25-winx64
	# 设置mysql数据库的数据的存放目录
	datadir=D:/Program Files/mysql-5.7.25-winx64/data
	# 允许最大连接数(建议设置大一点,防止出现连接数过多的错误信息)
	max_connections=2000
	# 服务端使用的字符集默认utf8
	character-set-server=utf8
	# 创建新表时使用的默认存储引擎
	default-storage-engine=INNODB
	# 设置sql语法模式
	sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
	
	[mysql]
	# 设置mysql客户端默认字符集
	default-character-set=utf8

4、用管理员身份进入bin所在文件，运行cmd。<br>
5、安装mysql服务
	
	mysqld install

6、生成data数据库文件夹
	
	mysqld --initialize-insecure --user=mysql

7、启动mysql服务
	
	net start mysql  //启动服务
	net stop  mysql  //停止服务

8、进入数据库
	
	mysql -u root -p 

	mysql -u root --skip-password  //不使用密码
	
由于之前并未随机生成密码，因此当出现Enter passowrd时直接回车即可。<br>
9、设置密码(进入数据库后执行)
	
	ALTER USER 'root'@'localhost' IDENTIFIED BY 'wlpwyh710';  //修改密码为wlpwyh710
    set password = 'wlpwyh710'  //两种方法均可


10、刷新权限
	
	flush privileges

11、将已有SQL文件导入MySql

	1、进入Mysql 
		#mysql -u root -p
	2、创建新的数据库
		mysql>create database test;
	3、选择数据库
		mysql>use test;
	4、设置数据库编码
		mysql>set names utf8;
	5、导入数据库文件
		mysql>source e:/mysql/test.sql;