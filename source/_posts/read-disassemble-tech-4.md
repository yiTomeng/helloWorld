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
除法对应的指令是idiv , div

5.如果除数是变量，编译器只能使用除法指令，如果除数是常量，则编译器会根据情况（负数，2的幂，非2的幂）来进行优化。
除法时汇编中会出现魔数，其结果为 2^n / o。 使用时一般采用变量乘以魔数后再移位的操作。如果变量乘以魔数后超出32位的范畴则会采取一定的措施，如再减去一个值。

6.
mov eax, MagicNumber
imul ...
sar edx, ...
mov reg, edx
shr reg, 1fh
add edx, reg

当遇上以上指令时，基本可以判定是除法优化的代码，其除法原型为a除以常量o,imul表示有符号计算，其操作数是优化前的被除数，接下来统计右移的总次数以确定公式中的n值，然后使用公式o=2^n / c，将MagicNumber作为c带入求解，四舍五入可以恢复除法原型。

mov eax, MagicNumber
mul reg
sub reg, edx
shr reg, 1
add edx, reg
shr reg, A

当遇上以上指令时，基本可以判定是除法优化的代码，其除法原型为a除以常量o,mul表示无符号计算，其操作数是优化前的被除数a，接下来统计右移的总次数以确定公式中的n值，然后使用公式o=2^(n+32) / ((2^32) + c)，将MagicNumber作为c带入求解，四舍五入可以恢复除法原型。


mov eax, MagicNumber(大于7fffffffh)
imul reg
add edx, reg
sar edx
mov reg, edx
shr reg, 1Fh
add edx, reg


当遇上以上指令时，基本可以判定是除法优化的代码，其除法原型为a除以常量o,imul表示有符号计算，其操作数是优化前的被除数a，接下来统计右移的总次数以确定公式中的n值，然后使用公式o=2^n / c，将MagicNumber作为c带入求解，四舍五入可以恢复除法原型。


mov eax, MagicNumber(大于7fffffffh)
imul reg
sar edx, ...
mov reg, edx
shr reg, 1Fh
add edx, reg


当遇上以上指令时，基本可以判定是除法优化的代码，其除法原型为a除以常量o,imul表示有符号计算，其操作数是优化前的被除数a，由于MagicNumber取值大于7fffffffh,而imul和sar之间未见任何调整代码，故可认定除数为负，且MagicNumber为补码形式。接下来统计右移的总次数以确定公式中的n值，然后使用公式|o|=2^n / (2^(32) - c)，将MagicNumber作为c带入求解，四舍五入可以恢复除法原型。


mov eax, MagicNumber(小于等于7fffffffh)
imul reg
sub edx, reg
sar edx, ...
mov reg, edx
shr reg, 1Fh
add edx, reg


当遇上以上指令时，基本可以判定是除法优化的代码，其除法原型为a除以常量o,imul表示有符号计算，其操作数是优化前的被除数a，由于MagicNumber取值小于等于7fffffffh,而imul和sar之间有sub指令来调整乘积，故可认定除数为负，且MagicNumber为补码形式。接下来统计右移的总次数以确定公式中的n值，然后使用公式|o|=2^n / (2^(32) - c)，将MagicNumber作为c带入求解，四舍五入可以恢复除法原型。


7.
无分之求绝对值

mov eax, [esp + argc]
cdq
!!!!xor eax, edx
!!!!sub eax, edx
push eax
...

扩展高位到edx，若eax为负数，则edx为-1,在机器中为全1，若eax为整数，则edx为0；
与全1异或则为取反，去全0异或则为本身。
则如上代码的意思为若为负数取反，然后减去-1即加1；
若为整数则本身减0。

C语言的测试代码：这种方法不用if语句判断正负来得到变量的绝对值

```C
#include <stdio.h>

int main(int argc, char *argv[])
{
	int a = -5;
    int b = 0;

    #if 0                   //OK, But this is restricted for param address, for example -4(%ebp) or -8(%ebp)
    asm
    (
        //"mov -4(%ebp), %eax \n\t"                 //vc++6.0 first param is in ebp - 4
        "mov -8(%ebp), %eax \n\t"                   //find first param by assembly that it is in ebp - 8 
        "mov $-1, %edx\n\t"
        "xor %edx, %eax\n\t"
        "sub %edx, %eax\n\t"
        //"mov %eax, -4(%ebp)\n\t"
        "mov %eax, -8(%ebp)\n\t"
    );
    #endif


    #if 0
    /*when using %0,%1...%9, you must add '%%' before registers*/
    /*get -a */
    asm
    (
        "mov %1, %%eax \n\t"                        
        "mov $-1, %%edx\n\t"
        "xor %%edx, %%eax\n\t"
        "sub %%edx, %%eax\n\t"
        "mov %%eax, %0\n\t"
        : "=r"(a) : "m"(a) 
    );
    #endif

    #if 0
    /*if a is positive, get a*/
    asm
    (
        "mov %1, %%eax \n\t"                        
        "mov $0, %%edx\n\t"
        "xor %%edx, %%eax\n\t"
        "sub %%edx, %%eax\n\t"
        "mov %%eax, %0\n\t"
        : "=r"(a) : "m"(a) 
    );
    #endif
    
    /*get the absolute value of a*/
    asm
    (
        "mov %1, %%eax \n\t"   
        "mov %%eax, %%ecx \n\t" 
        "mov $-1, %%edx\n\t"
        "shr $31, %%ecx\n\t" 
        "jnz LP\n\t"                     
        "mov $0, %%edx\n\t"
        "LP:\n\t"
        "xor %%edx, %%eax\n\t"
        "sub %%edx, %%eax\n\t"
        "mov %%eax, %0\n\t"
        : "=r"(a) : "m"(a) 
    );

    /*
    -->
    int temp = a;
    temp = temp >> 31;
    a = a ^ temp;
    a = a - temp;
    */ 
    
    

    printf("a:%p\n", &a);
    printf("b:%p\n", &b);
    printf("%d\n", a);

	return 0;
}


```

8.
a % (2 ^ 5)
->
mov reg, 被除数
and reg, 8000001F
jns LP
dec reg
or  reg, FFFFFFE0
inc reg

遇到以上的指令序列时，基本可判定为取模。其取模原型为被除数（变量）对2^k（常量）的取模。

9.溢出与进位

进位是对无符号数而言，无符号数进位后，CF进位标志位被设置；
溢出是针对有符号数，数据超出最大空间后会导致破坏有符号数的符号位，OF标志位被设置。

9.条件表达式的转化有四种情况：
表达式1？表达式2：表达式3
① 表达式1较为简单，表达式2和表达式3的差值为1 （setne设置al标志位）
例：

```
argc == 5 ? 5 : 6;
->
xor eax, eax
cmp dword ptr [ebp+8], 5
setne al
add eax, 5
```

② 表达式1较为简单，表达式2和表达式3的差值大于1

```
argc == 5 ? 4 : 10;
->
mov eax, dword ptr [ebp + 8]
sub eax, 5
neg eax							##取反(设置CF)
sbb eax, eax					##借位减法 eax - eax -CF
and eax, 6
add eax, 4


遇到
sub reg, A
neg reg
sbb reg, reg
and reg, B
add reg, C
这样的代码块，就可以表明是等值比较了，其判定值为A,可以直接还原为如下形式的高级代码：
reg == A ? C : B + C;
若表达式2大于表达式3，那么最后加的数字为一个负数

```

③ 表达式1较为复杂，表达式2和表达式3的差值大于1

```
argc <= 8 ? 4 : 10;
->
xor eax, eax
cmp dword ptr [ebp + 8], 8
setg al      ##当>8时为1，否则为0
dec eax
and al, 0FAh	##-6 = 4 - 10
add eax, 0Ax	

```

④ 表达式2和表达式3有一个变量，无优化。成分支结构

9.代码优化一般有四个方向：
执行速度优化，内存存储空间优化，磁盘存储空间优化，编译时间优化

中间代码生成的优化方案：（常见的与设备无关的优化方案：）
常量折叠，常量传播，减少变量，公共表达式，复写传播，剪去不可达分支，
顺序语句代替分支，强度削弱，数学变换，代码外提，

目标代码生成阶段优化方案：流水线优化，分支优化，高速缓存优化

流水线优化注意：指令相关性和地址相关性。编译器会调整流水线，但是在保证计算结果正确的前提下优化流水线。

分支优化：目标地址缓存器（BTB）中预测。
因此，嵌套循环时，大循环在内，小循环在外，可以增加命中的几率；

高速缓存优化：
cache优化有以下几种：
①数据对齐：cache不保存二进制低位
②数据集中：将访问次数多的数据或代码尽量安排在一起。一方面cache在抓取命中数据时会抓取周围的其他数据；另一方面是虚拟内存分页的问题，若数据分散，保存在多个分页中，就会导致过多的虚拟地址转换，甚至会导致缺页中断频繁发生，影响效率。
③减少体积：命中率高的代码段应减少体积，尽量放入cache中以提高效率

