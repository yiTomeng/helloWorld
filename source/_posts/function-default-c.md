title: C语言中默认参数的实现及相关define
date: 2015-02-02 19:13:50
tags: C
---

C++中的函数可以设置默认形参，不过也得注意设置形参默认值的位置。
C++中的函数默认形参设置有三种类型：
1.只在声明函数时定义；
2.只在定义函数时定义
3.声明时和定义时都定义

第一种情况：只在声明函数时定义，代码如下：

```C
	#include <iostream>
	using namespace std;

	int testFunc(int a, int b = 10);

	int main(int, char**)
	{
		int a = testFunc(5);
	
		cout << a << endl;
	
		return 0;
	}

	int testFunc(int a, int b)
	{
		return (a + b);
	}
```

这种情况下运行正常。

第二种情形：只在定义函数时定义。这种情况又可以分为两种：
--1.函数定义在函数调用之前，代码如下：

```C
	#include <iostream>
	using namespace std;

	int testFunc(int a, int b);

	int testFunc(int a, int b = 10)
	{
		return (a + b);
	}

	int main(int, char**)
	{
		int a = testFunc(5);
		
		cout << a << endl;
	
		return 0;
	}	
```

这时运行是没有问题，调用的时候可以知道定义时有设置默认的形参。

--2.函数定义在函数调用之后，代码如下：

```C
	#include <iostream>
	using namespace std;

	int testFunc(int a, int b);

	int main(int, char**)
	{
		int a = testFunc(5);
		
		cout << a << endl;
	
		return 0;
	}	

	int testFunc(int a, int b = 10)
	{
		return (a + b);
	}
```

这样的代码执行的时候，由于定义在函数调用之后，会出现函数参数不足的错误。

第三种情形是两者同时定义，代码如下：

```C
	#include <iostream>
	using namespace std;

	int testFunc(int a, int b= 10);

	int main(int, char**)
	{
		int a = testFunc(5);
		
		cout << a << endl;
	
		return 0;
	}	

	int testFunc(int a, int b = 10)
	{
		return (a + b);
	}
```

这几行代码会导致函数定义处报错，提示之前声明处已定义。

从上可以看出，虽然C++可以轻松实现带默认形参值的函数，可是必须得遵循一些准则：声明和定义必须只能有一方定义，且定义形参默认值的一方必须在调用函数之前。

C语言就没那么好运了，它默认不支持对函数的形参设置默认值，那么该如何实现呢？
在网上调查的结果就是：用宏来实现，代码如下：

```C
#define DEFAULT_PLAY_TIMES 1														//デフォルト再生回数：1回

#define play_wave_default(filename) play_wave(filename, DEFAULT_PLAY_TIMES)			//デフォルト再生回数の関数

extern int play_wave(const char *filename, int refresh_times);						//wavファイルを再生する関数
```

上面的例子可能有些牵强，只是替换了函数而已，调用的时候变成调用别的函数，下面摘取一个网上的例子：

```C
	#include <stdio.h>
	#define DEFAULT 40      /*默认参数值*/
	#define FUN(A) fun(#A##"-")    /*用于实现默认参数的宏*/
	
	int f(int n)  /*用于实验默认参数的函数*/
	{
	 return printf("%d\n",n);
	}
	int fun(const char *a)    /*确定函数调用的函数,返回值类型要和实际需要调用的f()函数返回值类型一致*/
	{
	 int n; /*变量的类型要和f()函数参数的类型一样*/
	 if (a[0]=='-') n=DEFAULT;
	 else sscanf(a,"%d",&n);
	
	 return f(n);
	
	}
	int main(void)
	{
	 FUN();
	 FUN(67);
	 return 0;
	}
```

