---
post: layout
title: 2020-2-24 Python lesson4
---

1、调用其他程序 <br>
2、多线程


# 一、调用其他程序
Python经常被用来开发自动化程序。<br>
自动化程序往往需要调用其它的程序。<br>
Python中调用外部程序主要是通过两个方法实现的， 一个是os库的 system 函数，另外一个是 subprocess 库。
## 1、os.system函数
使用os库的 system 函数 调用其它程序 是非常方便的。

就把命令行内容 作为 system 函数的参数 即可

比如，我们要使用wget下载 nginx 安装包，如果在命令行执行 是这样的

d:\tools\wget http://mirrors.sohu.com/nginx/nginx-1.13.9.zip
那么在Python程序中调用，就可以像这样

	import os
	cmd = r'd:\tools\wget http://mirrors.sohu.com/nginx/nginx-1.13.9.zip'
	os.system(cmd)

	print('下载完毕')


*** 注意,使用os.system函数调用外部程序的时候， 必须要等被调用程序执行结束， 才会接着往下执行代码。 否则就会一直等待。 ***

## 2、subprocess模块
subprocess 模块提供了 更多的 调用外部程序的功能。 <br>
首先，我们可以获取外部程序输出到屏幕的内容。 这在自动化的时候，非常有用。 可以用来判断外部程序执行结果是否成功， 或者获取我们要分析的数据。
比如，我们要分析当前C盘可用空间是否不足10%，如果不足，就显示空间告急。
可以使用 subprocess里面的 Popen类， 写如下代码

	from subprocess import PIPE,Popen
	
	'''
	使用Popen函数将外部命令进行打开，'fsutil volume diskfree c:'可直接用于windows中CMD执行
	。studin ,stdout,stderr 为标准输入（键盘、鼠标、文件等），stdout为标准输出，默认为输出到
	终端，赋值为PIPE表示，输出到管道，后期可通过PIPE进行读取数据，stderr为标准错误输出，默认也是输出到
	终端，赋值为PIPE表述输出到管道。
	'''
	proc = Popen(
	    'fsutil volume diskfree c:',
	    stdin = None,
	    stdout = PIPE,
	    stderr = PIPE,
	    shell = True)
	
	# 从proc中获取数据
	outinfo,errinfo = proc.communicate()
	
	#对数据进行解码
	outinfo = outinfo.decode('gbk')
	
	print(outinfo)
	
	outputlist = outinfo.splitlines()
	print(outputlist)
	
	temp = []
	for data in outputlist:
	    temp.append(int(data.split(":")[1].strip().split(" ")[0]))
	
	if temp[0]/temp[1] < 0.1:
	    print("!! 空间不足,只有不到{temp[0]/temp[1]}空间")
	else:
	    print(f"空间足够，还有{temp[0]/temp[1]}空间")
	    
	'''
	free1 = outputlist[0].split(":")[1]
	print(free1)
	
	free = int(free1.strip().split(" ")[0])
	print(free)
	
	all1 = outputlist[1].split(":")[1]
	all = int(all1.strip().split(" ")[0])
	print(all)
	
	
	if free/all < 0.1:
	    print("!! 空间不足,只有不到{free/all}空间")
	else:
	    print(f"空间足够，还有{free/all}空间")
	
	'''

其中 communicate 方法会返回 两个字符串对象。分别对应 被调用程序 输出到 标准输出 和 标准错误 的字节串内容。

命令行程序运行的时候会自动打开 标准输出设备文件 和 标准错误设备文件， 缺省情况下，两者 都是对应当前终端设备。我们可以简单理解为 就是屏幕。

所以这个方法返回的就是 屏幕上显示的内容。但是这个返回的是 bytes 字节串，不是 str 字符串。

前面我们已经学过，bytes要解码为str，就需要调用 decode方法。 我的是中文windows，所以通常外部程序输出的都是gbk编码的字节串。 所以参数指定为gbk。

# 二、多线程
## 1、进程和线程的概念
简单的说：进程就是运行着的程序<br>
线程是操作系统创建的，或者是执行程序创建的，每一个线程对应一个代码执行的数据结果，保存了代码执行过程中的重要状态信息。没有线程，操作系统没法管理和维护代码运行的状态信息。所以没有创建线程之前，操作系统是不会执行我们的代码的。<br>
Python3中若没有特别创建线程，这默认运行时只有主线程，线程通过顺序方式执行，执行完毕，程序结束。<br>
因此，为了让多个CPU核心同时去执行任务，我们的程序必须创建多个线程，让CPU执行多个线程对应的代码。
## 2、Python代码中创建的新线程
应用程序必须 通过操作系统提供的 系统调用，请求操作系统分配一个新的线程。python3 将 系统调用创建线程 的功能封装在 标准库 threading 中。

	print('主线程执行代码') 
	
	# 从 threading 库中导入Thread类
	from threading import Thread
	from time import sleep
	
	# 定义一个函数，作为新线程执行的入口函数
	def threadFunc(arg1,arg2):
	    print('子线程 开始')
	    print(f'线程函数参数是：{arg1}, {arg2}')
	    sleep(5)
	    print('子线程 结束')
	
	
	# 创建 Thread 类的实例对象， 并且指定新线程的入口函数
	thread = Thread(target=threadFunc,
	                args=('参数1', '参数2')
	                )
	
	# 执行start 方法，就会创建新线程，
	# 并且新线程会去执行入口函数里面的代码。
	# 这时候 这个进程 有两个线程了。
	thread.start()
	
	# 主线程的代码执行 子线程对象的join方法，
	# 就会等待子线程结束，才继续执行下面的代码
	thread.join()
	print('主线程结束')


运行该程序，解释器执行到下面代码时

	thread = Thread(target=threadFunc,
	                args=('参数1', '参数2')
	                )

创建了一个Thread实例对象，其中，Thread类的初始化参数 有两个
target参数 是指定新线程的 入口函数， 新线程创建后就会 执行该入口函数里面的代码，
args 指定了 传给 入口函数threadFunc 的参数。 线程入口函数 参数，必须放在一个元组里面，里面的元素依次作为入口函数的参数。
注意，上面的代码只是创建了一个Thread实例对象， 但这时，新的线程还没有创建。
要创建线程，必须要调用 Thread 实例对象的 start方法 。也就是执行完下面代码的时候thread.start()
新的线程才创建成功，并开始执行 入口函数threadFunc 里面的代码。
有的时候， 一个线程需要等待其它的线程结束，比如需要根据其他线程运行结束后的结果进行处理。
这时可以使用 Thread对象的 join 方法
thread.join()
如果一个线程A的代码调用了 对应线程B的Thread对象的 join 方法，线程A就会停止继续执行代码，等待线程B结束。 线程B结束后，线程A才继续执行后续的代码。

所以主线程在执行上面的代码时，就暂停在此处， 一直要等到 新线程执行完毕，退出后，才会继续执行后续的代码。
## 3、共享数据访问控制
做多线程开发，经常遇到这样的情况，多个线程里面的代码需要访问同一个公共的数据对象。这个公共的数据对象可以是任何类型，比如一个列表、字典、或者自定义的对象。有的时候，程序需要防止线程的代码同时操作公共数据对象。否则，就有可能导致数据的访问互相冲突影响。下面代码用一个简单的程序模拟一个银行系统，用户可以往自己的账号存钱。
对应代码如下：

	from threading import Thread
	from time import sleep
	
	bank = {
	    'byhy' : 0
	}
	
	# 定义一个函数，作为新线程执行的入口函数
	def deposit(theadidx,amount):
	    balance =  bank['byhy']
	    # 执行一些任务，耗费了0.1秒
	    sleep(0.1)
	    bank['byhy']  = balance + amount
	    print(f'子线程 {theadidx} 结束')
	
	theadlist = []
	for idx in range(10):
	    thread = Thread(target = deposit,
	                    args = (idx,1)
	                    )
	    thread.start()
	    # 把线程对象都存储到 threadlist中
	    theadlist.append(thread)
	
	for thread in theadlist:
	    thread.join()
	
	print('主线程结束')
	print(f'最后我们的账号余额为 {bank["byhy"]}')
上面的代码中，一起执行

开始的时候， 该帐号的余额为0，随后我们启动了10个线程， 每个线程都deposit函数，往帐号byhy上存1元钱。

可以预期，执行完程序后，该帐号的余额应该为 10。

然而，我们运行程序后，发现结果如下

	子线程 0 结束
	子线程 3 结束
	子线程 2 结束
	子线程 4 结束
	子线程 1 结束
	子线程 7 结束
	子线程 5 结束
	子线程 9 结束
	子线程 6 结束
	子线程 8 结束
	主线程结束

最后我们的账号余额为 1
为什么是 1 呢？ 而不是 10 呢？

如果在我们程序代码中，只有一个线程，如下所示

	from time import sleep
	
	bank = {
	    'byhy' : 0
	}

	# 定义一个函数，作为新线程执行的入口函数
	def deposit(theadidx,amount):
	    balance =  bank['byhy']
	    # 执行一些任务，耗费了0.1秒
	    sleep(0.1)
	    bank['byhy']  = balance + amount
	
	for idx in range(10):
	    deposit (idx,1)

	print(f'最后我们的账号余额为 {bank["byhy"]}')

代码都是 串行 执行的。 不存在多线程同时访问 bank对象 的问题，运行结果一切都是正常的。

现在我们程序代码中，有多个线程，并且在这个几个线程中都会去调用 deposit，就有可能同时操作这个bank对象，就有可能出一个线程覆盖另外一个线程的结果的问题。

这时，可以使用 threading库里面的锁对象 Lock 去保护。

我们修改多线程代码，如下：

	from threading import Thread,Lock
	from time import sleep
	
	bank = {
	    'byhy' : 0
	}
	
	bankLock = Lock()
	
	# 定义一个函数，作为新线程执行的入口函数
	def deposit(theadidx,amount):
	    # 操作共享数据前，申请获取锁
	    bankLock.acquire()
	    
	    balance =  bank['byhy']
	    # 执行一些任务，耗费了0.1秒
	    sleep(0.1)
	    bank['byhy']  = balance + amount
	    print(f'子线程 {theadidx} 结束')
	    
	    # 操作完共享数据后，申请释放锁
	    bankLock.release()
	
	theadlist = []
	for idx in range(10):
	    thread = Thread(target = deposit,
	                    args = (idx,1)
	                    )
	    thread.start()
	    # 把线程对象都存储到 threadlist中
	    theadlist.append(thread)
	
	for thread in theadlist:
	    thread.join()
	
	print('主线程结束')
	print(f'最后我们的账号余额为 {bank["byhy"]}')

执行一下，结果如下

	子线程 0 结束
	子线程 1 结束
	子线程 2 结束
	子线程 3 结束
	子线程 4 结束
	子线程 5 结束
	子线程 6 结束
	子线程 7 结束
	子线程 8 结束
	子线程 9 结束
	主线程结束
	最后我们的账号余额为 10

正确了。

Lock 对象的acquire方法 是申请锁。

每个线程在 操作共享数据对象之前，都应该 申请获取操作权，也就是 调用该 共享数据对象对应的锁对象的acquire方法。

如果线程A 执行如下代码，调用acquire方法的时候，

	bankLock.acquire()

别的线程B 已经申请到了这个锁， 并且还没有释放，那么 线程A的代码就在此处 等待 线程B 释放锁，不去执行后面的代码。

直到线程B 执行了锁的 release 方法释放了这个锁，

	bankLock.release()

 线程A 才可以获取这个锁，就可以执行下面的代码了。

如果这时线程B 又执行 这个锁的acquire方法， 就需要等待线程A 执行该锁对象的release方法释放锁， 否则也会等待，不去执行后面的代码。