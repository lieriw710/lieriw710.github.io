---
layout: post
title: 2020-2-21 Python学习日志
---
# 学习内容
1、Python类中对象数据的获取<br>
2、Python<br>
3、Python<br>

## 一、Python类中对象数据的获取

### 1、实例直接调用方式

定义的类，使用自动绑定方式，直接调用，则代码编写如下：

	class Person(object):
		def __init__(self,name,age):
			self.name = name
			self.age = age
	p = Person("lieriw710",34)

### 2、实例间接赋值方式

如需要在调用的实例中，给某一个参数赋值，则需要在类定义的__init__方法中使用字典方式获取对象的数据，代码编写如下：

	class Person(object):
		def __init__(self.name,age,**kw):
			self.name = name
			self.age = age
			for k,v in items():  #Python3中不能使用iteritems()
				setattr(self.k,v)
	p = Person("bob","Male",age=18, couse="Python")
	print(p.age)
	print(p.course)

上面代码中setattr(self,k,v),该方法是设置属性操作 <br>
	
	>>> setattr(self.name,"wangyuhong")  # 设置新的k属性
	
	>>self.k
	>"wangyuhong"
getattr（）函数是获取属性<br>
	
	>>>getattr(self,name)
	>>>self.name
	>>>"wangyuhong"

## 二、Python中特殊方法

### 1、Python如何把任意变量变成str？

因为任意数据类型的实例都有一个特殊方法   __str__()，<br>
我们定义一个 lst = [1,2,3],定义一个类P。对其进行操作，默认是调用__str__()方法，如下代码所示：

	>>>print(lis.__str__())
	> [1,2,3]
	>>>print(p.__str__())
	> <__main__.Person object at 0x10da938200>
Python 的特殊方法，常用的有：

	用于print 的__str__()
	用于len的__len__()
	用于cmp的__cmp__()
python de 特殊方法的特点有：
	
	特殊方法定义在class中
	不需要直接调用
	Python的某些函数或操作符会调用对应的特殊方法

正确实现特殊方法
	
	只需要编写用到的特殊方法
	有关联性的特殊方法都必须实现
		如要实现__getattr__方法，必须把__setattr__()和__delattr__都实现

### 2、Python中的__str__和__repr__方法的使用

如果要把一个类的实例变成str,就需要实现特殊方法__str__()
	
	class Person(object):
		def __init__(self,name,gender):
			self.name = name
			self.gender = gender
		def __str__(self):
			return "(Person:%s,%s)"%(self.name,self.gender)
现在，在交互式命令行下用print打印相关数据：
	
	>>p = Person("wangyuhong","male")
	>>print(p)
	> (Person:wangyuhong,male)
但是，如果直接敲变量p：

	>>p
	><main.Person object at 0x10c134343>
似乎__str__()不会调用。<br/>
这是因为Python定义了__str__()和__repr__()两种方法，__str__()用于显示给用户，而__repr__()用于显示给开发人员。<br>
有一个偷懒的方法时在类的结尾，将

	__repr__ = __str__

### 3、多继承中，构造函数的编写

若一个类继承多个类，那么在新类中必须对父类的构造函数进行初始化，第一个继承的类可以通过调用super()函数的方法,如：

	super().__init__(name)
但第二个父类开始不能使用super()函数进行调用，必须使用未绑定方法调用父类的构造方法,如：

	Person.__init__(self.name,gender)
	Ablilty.__init__(self,skill)
</br>

### 4、Python中__cmp__的用法

对int、str等内置数据类型排序时，Python的sorted()按照默认的比较函数cmp排序，但是，如果对一组Student类的实例排序时，就必须提供我们的自己的特殊方法__cmp__().
未解决的问题，我在Python3.8编译环境下，改写类中的__cmp__()函数比较两个实例大小，运行失败，没找到好的办法。代码如下：

	class Person(object):
		def __init__(self,name,age):
			self.name = name
			self.age = age
		def __str__(self):
			return "Student:%s,%f"%(self.name,self.age)

		__repr__ = __str__

		def __cmp__(self,s):
			if self.name>s.name:
				return -1
			else:
				return 1
	p1 = Person("wangyuhong",34)
	p2 = Person("wuliping",37)
	l = [p1,p2]
	print(sorted(l))
以上代码运行错误，显示

	TypeError: '<' not supported between instances of 'Person' and 'Person'
以上问题，待会儿学习完其他的内容之后解决

### 5、Python中的__len__函数

如果一个类表现得像一个list，要获取有多少个元素，就得泳道len()函数，要让len()函数正常工作，类必须提供一个特殊的方法__len__(),它返回元素的个数，例如，我们写一个Student类，把名字传进去：

	class Student(object):
		def __init__(self,*args):
			self.names = args
		def __len__(self):
			return len(self.names)

	ss = Student("Bob","Alice","Tim")
	print(len(ss)

### 6、Python中的数学运算

Python提供的基本数据类型int、float可以做整数和浮点的四则运算以及乘方等运算，但是，四则运算不局限于int和float，还可以是有理数和矩形。<br>

	有理数：整数几分数的集合，小数点之后的数不循环。
	无理数：也称为无限不循环小数，不能写作两整数之比。
要表示有理数，可以用一个Rational类来表示（也可以用其他名字）：

	class Rational(object):
		def __init(self,q,p):
			self.p = p
			self.q = q
p和q都是整数，表示有理数p/q <br>
如果要让Rational进行+运算，需要正确实现__add__:

	class Rational(object):
		def __init(self,q,p):
			self.p = p
			self.q = q
		def __add__(self.r):
			return Rational(self.p*r.q+self.q*r.p,self.q*r.q)
		def __str__(self):
			return "%s/%s"%(self.p,self.q)
		__repr__ = __str__
	
现在可以试试有理数加法:
	
	>>r1 = Rational(1,3)
	>>r2 = Rational(1,2)
	>>print(r1+r2)
	>5/6

同样，要实现有理数的减法、乘法、除法，需要继续晚上Rational类，需要给类添加

	减法运算:__sub__
	乘法运算:__mul__
	除法运算: __div__

代码如下：

	def __sub__(self.r):
		return Rational(self.p*r.q-self.q*r.p,self.q*r.q)
	def __mul__(self.r):
		return Rational(self.p*r.p,self.q*r.q)
	def __div__(self,r):
		return Rational(self.p*r.q,self.q*r.p)
如果需要将有理数转换为int类型或者float类型，必须在类中实现:

	int类型转换:__int__
	float类型转换：__float__
实现代码如下:

	def __int__(self,r):
		return self.p // self.q
	def __float__(self,r):
		return float(self.p) / self.q # Python 2.x 需要其中一个数为float型才能经浮点运算
	
类中实现了__int__及__float__函数功能后，可对实例进行int 或 float 操作，例如：

	r1 = Rational(1,3)
	r2 = Rational(2,5)
	data1 = int(r1) + int(r2)
	data2 = float(r1) + float(r2)

### 7、Python中的property函数功能

### 》使用property函数定义属性
如果为Python类定义了getter、setter等访问器方法，则可使用property()函数将他们定义成属性（相当于实例变量）<br>
property()函数的语法格式如下

	property(fget=None,fset=None,fdel=None,doc=None)
从上面的语法格式可以看出，在使用property()函数时，可传入4个参数，分别代表getter、setter 、del方法及doc，其中doc是一个文档字符串，用于说明该属性。当然，我们调用property也可传入0个（可任意出入)。特别需要说明的：

	* 传入getter方法时，该函数需要实现return功能
	* 传入setter方法时，需进行赋值
	* 传入del方法时，不作具体要求
	* 传入doc方法是，需传入str类型字符串。

例如，下面的程序定义了一个Rectangle，该类的property()函数定义了一个size属性。

	class Rectangle(object):
		def __init__(self,width,height):
			self.width = width
			self.height = height
		#定义setter类型的方法
		def setsize(self,size):
			self.width,self.height = size
		#定义getter类型的方法
		def getsize(self):
			return self.width,self.height
		#定义del类型的方法
		def delsize(self):
			self.width,self.heigth = 0,0
		
		#使用property()定义属性
		size = property(getsize,setsize,delsize,"lieriw710写的函数)
在交互式命令行中可实现
	
	>>print(Rectangle.size.__doc__) 
	>lieriw710写的文档
	>>help(Rectangle.seze)
	>lieriw710写的文档
	>>rect = Rectangle(3,4)
	>>print(rect.size)
	>（3,4)
	>>rect.size = 9,8
	>>print(rect.width)
	>9
	>>print(rect.height)
	>8
	>del rect.size
	>>print(rect.height)
	>0

### 》使用@property 作为装饰器使用。
我们可以使用@property装饰器来修饰类中的方法，在类方法前一行中添加@property相当于使用getter()方法，该装饰器会默认产生一个副作用，通过使用上述方法中的变量添加setter,即是实现了property中的setter()方法。如下面代码所示：

	class Cell:
		# 使用property修饰方法，相当于为该属性设置getter方法
		@property
		def state(self):
			return self.state
		# 使用@state.setter修饰，相当于该属性设置setter方法
		@state.setter
		def state(self,value):
			if "alive" in value.lower():
				self.__state = "alive"
			else:
				self.__state = "dead"

		@property
		def is_dead(self):
			return not self.__state.lower() == "alive"
	c = Cell()
	c.state = "Alive"
	print(c.state)
	print(c.is_dead)

### End


			
