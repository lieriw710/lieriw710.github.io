---
post: layout
title: 2020-2-24 Python lesson4
---
目录
	
	1、调用其他程序
	2、

#调用其他程序
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

