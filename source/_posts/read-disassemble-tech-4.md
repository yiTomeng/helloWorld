title: C++反汇编与逆向分析技术揭秘(4)--第四章
date: 2015-03-05 09:42:29
tags: Read Tech
---

1.VC++ 6.0上有几种优化方式：
① O1 release 文件最小化
② O2 release 效率最高化
③ Od + ZI debug优化组 添加一个用于调试的信息，也不是完全不优化

第三种可以得到源代码与汇编代码的一一对应，而O2则汇编代码会有大幅变化。

2.有两种常见的编译器优化方式：常量传播和常量折叠

常量传播：

```C
int main(void)
{
	int a = 1;
	printf("%d\n", a);
	return 0;
}

```

上述代码的情况下，会被编译成类似下面代码：

```C
int main(void)
{
	//int a = 1;
	printf("%d\n", 1);
	return 0;
}

```

常量折叠：
同样举个例子方便理解:

```C
int main(void)
{
	int a = 1 + 2 * 4;
	printf("%d\n", a);
	return 0;
}

```
上述代码会先变成下面代码的感觉：

```C
int main(void)
{
	int a = 9;
	printf("%d\n", a);
	return 0;
}

```

最后变成:

```C
int main(void)
{
	printf("%d\n", 9);
	return 0;
}

```

这样把编译时能计算出结果的都变成常量，这样也可以减少一些变量的使用。

3.复写传播

4.加法对应的汇编指令为ADD
减法对应的汇编指令为SUB，计算机只能计算加法，因此采取加补码的方式：a + a(verse) + 1 = 0-> 0 - a = a(verse) + 1 -> 0 - a = a(buma)
乘法对应的汇编指令为：有符号为imul , 无符号为mul。编译过程中会尽量先转换成加法指令或移位指令，实在不可行才使用乘法指令。


---P56