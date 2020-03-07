---
post: layout
titile:  2020-3-7 CPP Lesson2
---
### C++ static
window有具体api函数来操作这些控制台输出输入缓冲区。可以在指定的位置进行输出，还可以设置字符的颜色等。在 《intel汇编语言程序设计》这本书里看见过，从外国翻译过来的，罗云彬 申校。
###一、控制台窗口操作
用于控制台窗口操作的API函数如下：

	GetConsoleScreenBufferInfo 获取控制台窗口信息
	GetConsoleTitle 获取控制台窗口标题
	ScrollConsoleScreenBuffer 在缓冲区中移动数据块
	SetConsoleScreenBufferSize 更改指定缓冲区大小
	SetConsoleTitle 设置控制台窗口标题
	SetConsoleWindowInfo 设置控制台窗口信息
此外，还有窗口字体、显示模式等控制函数，这里不再细说。下列举一个示例，程序如下：

	#include
	#include
	#include
	void main()
	{
	HANDLE hOut = GetStdHandle(STD_OUTPUT_HANDLE);
	// 获取标准输出设备句柄
	CONSOLE_SCREEN_BUFFER_INFO bInfo; // 窗口缓冲区信息
	GetConsoleScreenBufferInfo(hOut, bInfo );
	// 获取窗口缓冲区信息
	char strTitle[255];
	GetConsoleTitle(strTitle, 255); // 获取窗口标题
	printf("当前窗口标题是：%s\n", strTitle);
	_getch();
	SetConsoleTitle("控制台窗口操作"); // 获取窗口标题
	_getch();
	COORD size = {80, 25};
	SetConsoleScreenBufferSize(hOut,size); // 重新设置缓冲区大小
	_getch();
	SMALL_RECT rc = {0,0, 80-1, 25-1}; // 重置窗口位置和大小
	SetConsoleWindowInfo(hOut,true ,&rc);
	CloseHandle(hOut); // 关闭标准输出设备句柄
	}
需要说明的是，控制台窗口的原点坐标是(0, 0)，而最大的坐标是缓冲区大小减1，例如当缓冲区大小为80*25时，其最大的坐标是(79, 24)。
###二、文本属性操作

与DOS字符相似，控制台窗口中的字符也有相应的属性。这些属性分为：文本的前景色、背景色和双字节字符集(DBCS)属性三种。事实上，我们最关心是文本颜色，这样可以构造出美观的界面。颜色属性都是一些预定义标识：

	FOREGROUND_BLUE 蓝色
	FOREGROUND_GREEN 绿色
	FOREGROUND_RED 红色
	FOREGROUND_INTENSITY 加强
	BACKGROUND_BLUE 蓝色背景
	BACKGROUND_GREEN 绿色背景
	BACKGROUND_RED 红色背景
	BACKGROUND_INTENSITY 背景色加强
	COMMON_LVB_REVERSE_VIDEO 反色

与文本属性相关的主要函数有：

	BOOL FillConsoleOutputAttribute( // 填充字符属性
	HANDLE hConsoleOutput, // 句柄
	WORD wAttribute, // 文本属性
	DWORD nLength, // 个数
	COORD dwWriteCoord, // 开始位置
	LPDWORD lpNumberOfAttrsWritten // 返回填充的个数
	);

	BOOL SetConsoleTextAttribute( // 设置WriteConsole等函数的字符属性
	HANDLE hConsoleOutput, // 句柄
	WORD wAttributes // 文本属性
	);

	BOOL WriteConsoleOutputAttribute( // 在指定位置处写属性
	HANDLE hConsoleOutput, // 句柄
	CONST WORD *lpAttribute, // 属性
	DWORD nLength, // 个数
	COORD dwWriteCoord, // 起始位置
	LPDWORD lpNumberOfAttrsWritten // 已写个数
	);
另外，获取当前控制台窗口的文本属性是通过调用函数GetConsoleScreenBufferInfo后，在CONSOLE_SCREEN_ BUFFER_INFO结构成员wAttributes中得到。
### 三、文本输出
文本输出函数有：

	BOOL FillConsoleOutputCharacter( // 填充指定数据的字符
	HANDLE hConsoleOutput, // 句柄
	TCHAR cCharacter, // 字符
	DWORD nLength, // 字符个数
	COORD dwWriteCoord, // 起始位置
	LPDWORD lpNumberOfCharsWritten // 已写个数
	);

	BOOL WriteConsole( // 在当前光标位置处插入指定数量的字符
	HANDLE hConsoleOutput, // 句柄
	CONST VOID *lpBuffer, // 字符串
	DWORD nNumberOfCharsToWrite, // 字符个数
	LPDWORD lpNumberOfCharsWritten, // 已写个数
	LPVOID lpReserved // 保留
	);

	BOOL WriteConsoleOutput( // 向指定区域写带属性的字符
	HANDLE hConsoleOutput, // 句柄
	CONST CHAR_INFO *lpBuffer, // 字符数据区
	COORD dwBufferSize, // 数据区大小
	COORD dwBufferCoord, // 起始坐标
	PSMALL_RECT lpWriteRegion // 要写的区域
	);

	BOOL WriteConsoleOutputCharacter( // 在指定位置处插入指定数量的字符
	HANDLE hConsoleOutput, // 句柄
	LPCTSTR lpCharacter, // 字符串
	DWORD nLength, // 字符个数
	COORD dwWriteCoord, // 起始位置
	LPDWORD lpNumberOfCharsWritten // 已写个数
	);

可以看出：WriteConsoleOutput函数功能相当于SetConsoleTextAttribute和WriteConsole的功能。而WriteConsoleOutputCharacter函数相当于SetConsoleCursorPosition(设置光标位置)和WriteConsole的功能。不过在具体使用要注意它们的区别。