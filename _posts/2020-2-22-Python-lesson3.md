---
post: layout
title: 2020-2-22 Python高级运用
---
今天周末，继续学习Python语言，昨天学习了Pyhon中特殊函数，包括__len__、__str__、__int__、__float__。学习了property()作为构造器的运用等，今天继续学习下面课程。

### 一、Python中__slots__

由于Python是动态语言，任何实例在运行期都可以动态添加属性。因此若要限制类添加属性。可以利用Python的一个特殊的__slot__来实现，即是在该方法中定义运行该类的属性列表。如在Student类中只允许添加name、gender、score这3个属性，可定义如下：

	class Student(object):
		__slots__("name","gender","score")
		def __init__(self,name,gender,score):
			self.name = name
			self.gender = gender
			self.score = score

现在对实例进行操作

	>>> s = Student("Bob","Male",49)
	>>> s.name = "Time" # ok
	>>> s.score = 99 # ok
	>>> s.grad = "A"
	>Tracebake(most recent call last):
	>...
	>AttributeError:"Student"object has no attribute "grade".

__slots__的目的是限制当前类所拥有的属性，如果不需要添加任何动态的属性，使用__slots__也能节省内存。**__slot__**中定义的可以是属性也可以是方法。

### 二、Python中__call__

在类中实现__call__功能，可以将一个类实例变成一个可调用对象，例如，我们把Person类变成一个可调用对象：

	class Person(object):
		def __init__(self,name,gender):
			def.name = name
			def.gender = gender
		
		def __call__(self,friend):
			print("my name is %s..."%self.name)
			print("my friend is %s..."%self.friend)

现在可以对Person实例直接调用
	
	>>>p = Person("Bob","Male")
	>>>p("Tim")
	>my name is Bob...
	>my friend is Tim...
	>
单看p("Tim")，无法确定p是一个函数还是一个类实例，所以，在Python中，函数也是对象，对象和函数的区别并不是很显著。<br>
下面实例通过实现__call__方法，调用类实例

	class Fib(object):
		def __init__(self):
			pass
		def __call__(self.num):
			a,b,lst = 0,1,[]
			for data in range(0,num):
				lst.append(a)
				a,b = b,a+b
			return lst

在命令行中进行调用

	>>>f = Fib()
	>>>print(f(10))
	>[0,1,1,2,3,5,8,13,21,34]