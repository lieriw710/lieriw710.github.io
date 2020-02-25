---
post: layout
title: 2020-2-25 Python lesson3
---
##今日学习任务
	
	1、Python的内置函数和内置方法的调用
	2、Python安装其他库
	3、Python字符串编解码
	4、Python 文件和目录操作


### 1 Python的内置函数和内置方法的调用
#### Python 内置函数
通过查看Python官方网站https://docs.python.org/3/library/functions.html#pow，可以查到，内置的函数如下表所示。

	abs()         delattr()   hash()       memoryview()   set()
	all()         dict()      help()       min()          setattr()
	any()         dir()       hex()        next()         slice()
	ascii()       divmod()    id()         object()       sorted()
	bin()         enumerate() input()      oct()          staticmethod()
    bool()        eval()      int()        open()         str()
    breakpoint()  exec()      isinstance() ord()          sum()
    bytearray()   filter()    issubclass() pow()          super()
    bytes()       float()     iter()       print()        tuple() 
    callable()    format()    len()        property()     type()
    chr()         frozenset() list()       range()        vars()
    classmethod() getattr()   locals()     repr()         zip()
    compile()     globals()   map()        reversed()     __import__()
	complex()     hasattr()   max()round()

调用内置函数的用法，是需要在将变量作为参数传递给内置函数，如下所示：

	abs(x): Return the absolute value of a number.
	all(iterable):if all elements of the iterable are true return True.
	any(iterable):Return True if any element of the iterable is true. If the iterable is empty, return False

#### Python内置方法
准确的来说，每一个变量所具有的内置方法，是不一样的，可在console终端通过dir(x),调用内置函数的方式获取，例如：

	>>>a = [1,3,5,97,23,33,5]
	>>>dir(a)
	>>>
	['__add__', '__class__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']

打印的方法，都可以作为a变量使用，内置方法，可通过x.method()方式进行调用，方法中可传入具体参数。若要获取某个方法的用法，可使用内置函数help方法进行调用，具体如下：

	>>>help(a.count)
	Help on built-in function count:
	count(value, /) method of builtins.list instance
    Return number of occurrences of value.
	>>>a.count(5)
	>2  #5这个数字在a列表中出现2次

我们学习一个比较有用的方法,split(),该方法经常用来从字符串中截取出我们想要的信息。split方法以参数字符为分割符，将字符串切割为多个字符串，作为元素存入一个列表，并返回这个列表。

	>>>str1 = "wangyuhong:100,wuliping:90,wangxiwen:80"
	>>>post1 = str1.split(",")
	>["wangyuhong:100","wuliping:90","wangxiwen:90"]

join()方法和split方法正好相反。split()是将字符串，以某字符串为界，切割成多个字符串，存入列表，join()是将列表中的字符串以某个字符串为连接符，连接成一个字符串。

	>>>" * ".join(post1)
	>'wangyuhong:89 * wuliping:89 * wangxiwen:23 * wangzhenbo:80'


### Python安装其他库
Python之所以这么流行，就是因为它由太多的优秀的第三方库和框架。有了这些库，Python才能在各行各业这么受欢迎。因为这些库极大的提高了开发效率。要使用这些库，我们需要安装到本地。
在Python中，安装第三方库，通常是使用pip命令。<br>
那些优秀的第三方库，基本都是放在一个叫PYPI的网站上，例如，我们要安装requests。可以执行如下的命令：

	pip install requests

我们在国内，使用pip安装的时候，可能由于网络原因，到国外访问PYPI会比较慢。而国内由网站（比如豆瓣）对PYPI做了镜像备份，我们可以通过在命令行中加上参数-i https://pypi.douban.com/simple/，这样就指定使用豆瓣作为安装包的下载地址。如下所示：

	pip install requests -i https://pypi.baidu.com/simple/

如果pip安装库的时候，出现SSL错误，可能是网络对https证书校验的问题，可以该用http协议下载，如下所示：

	pip install requests -i http://pypi.douban.com/simple/  --trusted-host pypi.douban.com

安装好以后，我们就可以使用import去导入这些库并且使用：

	import requests
	requests.get("http://www.baidu.com")

### Python字符串编解码
Python语言要对字符串对象进行存储和传输是，通常要使用字符串的encode方法，参数指定编码方式，编码为一个bytes对象。<br>
bytes对象的底层就是一个的字节来存储字符串的文字的。如：

	str = "我是王玉红"
	print(str.encode('utf8'))
	print(str.encode('gbk'))

当我们的Python程序从文件中读入文字信息，从网络上接收文字信息，获取的数据通常是使用某种字符编码的字节串。程序通常需要解码字节串为字符串。我们使用decode方法进行解码，参数指定
编码方式，比如：

	sbytes = b'\xe4\xbd\xa0\xe5\xa5\xbd'
	print(sbytes.decode('utf8'))
	print(sbytes.decode('gbk'))

### Python 捕获异常
简单的匹配所有异常<br>
如果我们在编写一段代码的时候，不知道这段代码会抛出什么样的异常，并且我们不许忘程序因为异常而中止。我们可以匹配所有类型的异常，这样任何类型的异常都不会终止程序。如下：

	try:
    	100/0
	except Exception as e:
    	print('未知异常:', e)

因为所有的异常都是Exception的子类，所有Exception能匹配所有类型的异常。<br>
还有一种更简洁的写法，也可以匹配所有类型的异常，如下所示：

	try:
		100/0
	except:
		print("unknow error!")

如果我们想知道异常的详细信息，可以使用traceback模块，如下：

	import tracekback
	try:
		100/0
	except:
		print(traceback.format_exc())

**自定义异常**

异常类型都是继承自Exception的类，表示各种类型的错误。
我们也可以自己定义异常，比如我们写一个用户注册的函数，要求用户输入的点好号码只能是中国的电话号码，并且电话号码中不能有非数字字符。
可以定义下面两种异常类型：

	class InvalidCharError(Excetpion):
		pass
	class NotChinaTelError(Exception):
		pass

	def register():
		tel = input("input No:")
		if not tel.isdigit():
			raise InvalidCharError
		if not tel.shartswitch('86'):
			raise NotChinaTelError
	    return tel
	
	try:
		ret = register()
	except InvalidCharError:
		print("wrong char"
	except NotChinaTelError:
		print("not digist")

### Python 文件和目录操作
#### 1、创建目录
os.makedirs 可以递归的创建目录结构，比如

	import os
	os.makedirs("tmp/python/fileop",exist_ok_True)

会在当前工作目录下面创建tmp目录，在tmp目录下面在创建python目录，在python目录下面在创建fileop目录。<br>
exist_ok = True 指定了，如果某个要创建的目录存在，也不报错，同时使用以创建的目录。

#### 2、删除文件和目录
os.remove 可以删除一个文件，比如：
	
	os.remove("sdf.py")
shutil。rmtree()可以递归的删除某个目录所有的子目录和子文件：

	import shutil
	shutil.rmtree("tmp")

#### 3、拷贝目录
如果我们要拷贝一个目录里面的所有内容(包括子目录和文件等）到另一个目录中，可以使用shutil的copytree函数。

	from shutil import copytree
	copytree("d:/tools/aaa","e:/new/bbb")
	# 拷贝d:/tools/aaa目录中的所有内容到 e:/new/bbb中

*** 注意，拷贝前，目标目录必须不存在，否则报错。 ***

#### 4、修改文件名、目录名
要修改文件名、目录名，可以使用os模块的rename函数，比如

	import os
	
	os.rename("d:/tools/aaa","d:/tools/bbb")
	# 修改目录名为d:/tools/aaa 为d:/tools/bbb

	os.rename("d:/tools/first.py","d:/tools/second.py")
*** 注意，重命名之前d:/tools/second.py已经存在，则会被覆盖***

#### 4、对文件路径名的操作
对于文件名的操作，比如获取文件名称，文件所在目录，文件路径的拼接等，都可以使用os.path模块.该模块能自动处理类似Data/data.csv和Data\data.csv这样在windows和linux系统的
差异

	>import os
	>path = "/Users/beazley/Data/data.csv"
	># 获取路径中的文件名部分
	>os.path.basename(path)
	"data.csv"
	
	># 获取路径中的目录部分
	>os.path.dirname(path)
	>'/Users/beazley/Data'

	># 文件路径的拼接
	>os.path.join('tmp','data',os.path.basename(path))
	>'tmp/data/data.csv'
	
#### 5、判断文件、目录是否存在
os模块中可以判断一个指定路径的文件或者目录是否存在，可以使用下面的方法：

	import os
	os.path.exists('d:/systems/cmd.exe')
	os.path.exists('d:/systems')
	# 返回值为True，表示存在，否则表示不存在

	os.path.isdir('d:/systems')
	# 返回值是True表示是目录

#### 6、获取文件的大小和日期

	>import os
	>os.path.getsize('etc/passed')
	3669
	>os.path.getmtime('etc/passd')
	>127234234.0234
	>import time
	>time.ctime(os.path.getmtime('etc/passd')


*** etc/passd 表示当前工作目录下的文件或文件名 ***

#### 7、递归的遍历目录下面所有的文件
假如我们要获取某个目录中所有的文件，包括子目录里面的文件。可以使用os库中的walk方法，<br>
比如我们要得到某个目录下面的所有的子目录和所有的文件，存放在两个列表中，可以这样写代码：

	import os
	#目标目录
	targetDir = r'd:\tmp\util\dist\check'
	files = []
	dirs  = []
	
	#下面的三个变量dirpath   dirnames    filenames分别代表如下：
	#dirpath：代表当前遍历到的目录名
	#dirnames：是列表对象，存放当前dirpath中的所有子目录名
	#filenames：是列表对象，存放当前dirpath中的所有文件名

	for(dirpath,dirnames,filenames) in os.walk(targetDir):
	files+=filenames
	dirs+=dirnames
	
	print(files)
	print(dirs)
	
#### 8、得到目录中所有的文件和子目录名

	import os
	targetDir = r'D:\QT\qt\02.QT案例源码大全'
	
	files = os.listdir(targetDir)
	print(files)

如果我们只需要获取目录中所有的文件，或者只需要子目录，可以这样

	import os
	form os.path import isfile,join,isdir
	
	targetDir = r'D:\QT\qt\02.QT案例源码大全'
	files for files in os.listdir(targetDir) if isfile(join(targetDir,f))]
	
***注意，判断一个文件是否为文件或者目录，使用isfile或者isdir时，需要传入全部路径，因此，上面代码中使用join(targetDir,f)获取当前的文件或者
目录的名字***

### 9、得到目录中指定扩展名的文件和子目录
可以使用glob库

	import glob
	exes = glob.glob(r'D:\QT\qt\02.QT案例源码大全')
	print(exes)
	
	