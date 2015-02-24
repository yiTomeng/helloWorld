title: localtime和localtime_r及相关时间处理结构体和相关函数
date: 2015-01-28 15:44:47
tags: C
---

## 一、相关结构体
### 1、time_t
time_t实际上是长整数类型，定义为：

```C
typedef long time_t; /* time value */
```
 
### 2、timeval
timeval是一个结构体，在time.h中定义为：

```C
struct timeval
{
     __time_t tv_sec;                /* Seconds. */
     __suseconds_t tv_usec;      /* Microseconds. */
};
```

其中，tv_sec为Epoch（1970-1-1零点零分）到创建struct timeval时的秒数，tv_usec为微秒数，即秒后面的零头。
 
### 3、tm
tm是一个结构体，定义为：

```C
struct tm
{
    int tm_sec;      /*代表目前秒数，正常范围为0-59，但允许至61秒 */
    int tm_min;     /*代表目前分数，范围0-59*/
    int tm_hour;   /* 从午夜算起的时数，范围为0-23 */
    int tm_mday;  /* 目前月份的日数，范围01-31 */
    int tm_mon;   /*代表目前月份，从一月算起，范围从0-11 */
    int tm_year;   /*从1900 年算起至今的年数*/
    int tm_wday;   /* 一星期的日数，从星期一算起，范围为0-6。*/
    int tm_yday;   /* Days in year.[0-365] */
    int tm_isdst;   /*日光节约时间的旗标DST. [-1/0/1]*/
};
 ```

## 二、具体操作函数

time()函数
　　原 型：time_t time(time_t * timer)
　　功 能: 获取当前的系统时间，返回的结果是一个time_t类型，其实就是一个大整数，其值表示从CUT（Coordinated Universal Time）时间1970年1月1日00:00:00（称为UNIX系统的Epoch时间）到当前时刻的秒数。然后调用localtime将time_t所表示的CUT时间转换为本地时间（我们是+8区，比CUT多8个小时）并转成struct tm类型，该类型的各数据成员分别表示年月日时分秒。
　程序例1:
　　time函数获得日历时间。日历时间，是用“从一个标准时间点到此时的时间经过的秒数”来表示的时间。这个标准时间点对不同的编译器来说会有所不同，但对一个编译系统来说，这个标准时间点是不变的，该编译系统中的时间对应的日历时间都通过该标准时间点来衡量，所以可以说日历时间是“相对时间”，但是无论你在哪一个时区，在同一时刻对同一个标准时间点来说，日历时间都是一样的。

```C
　　#include <time.h>
　　#include <stdio.h>
　　#include <dos.h>
　　int main(void)
　　{
　　time_t t; t = time(NULL);
　　printf("The number of seconds since January 1, 1970 is %ld",t);
　　return 0;
　　}
```

　程序例2：
　　//time函数也常用于随机数的生成，用日历时间作为种子。

```C
　　#include <stdio.h>
　　#include <time.h>
　　#include<stdlib.h>
　　int main(void)
　　{
　　int i;
　　srand((unsigned) time(NULL));
　　printf("ten random numbers from 0 to 99\n\n");
　　for(i=0;i<10;i++)
　　　printf("%d\n",rand()%100);
　　return 0;
　　}
```

　程序例3：
　　用time()函数结合其他函数（如：localtime、gmtime、asctime、ctime）可以获得当前系统时间或是标准时间。

```C
　　#include <stdio.h>
　　#include <stddef.h>
　　#include <time.h>
　　int main(void)
　　{
　　time_t timer;//time_t就是long int 类型
　　struct tm *tblock;
　　timer = time(NULL);//这一句也可以改成time(&timer);
　　tblock = localtime(&timer);
　　printf("Local time is: %s\n",asctime(tblock));
　　return 0;
　　}
 ```

gmtime()函数
　　原 型：struct tm *gmtime(long *clock);
　　功 能：把日期和时间转换为格林威治(GMT)时间的函数。将参数timep 所指的time_t 结构中的信息转换成真实世界所使用的时间日期表示方法，然后将结果由结构tm返回。 
　　说 明：此函数返回的时间日期未经时区转换，而是UTC时间。
　　返回值：返回结构tm代表目前UTC 时间
程序例

```C
　　#include "stdio.h"
　　#include "time.h"
　　#include "stdlib.h"
　　int main(void)
　　{
　　time_t t;
　　struct tm *gmt, *area;
　　tzset(); /* tzset()设置时区*/
　　t = time(NULL);
　　area = localtime(&t);
　　printf("Local time is: %s", asctime(area));
　　gmt = gmtime(&t);
　　printf("GMT is: %s", asctime(gmt));
　　return 0;
　　}
```

localtime()函数
　　功 能: 把从1970-1-1零点零分到当前时间系统所偏移的秒数时间转换为日历时间 。
　　说 明：此函数获得的tm结构体的时间，是已经进行过时区转化为本地时间。
　　用 法: struct tm *localtime(const time_t *clock);
　　返回值：返回指向tm 结构体的指针.tm结构体是time.h中定义的用于分别存储时间的各个量(年月日等)
的结构体.
 程序例1:

```C
　　#include <stdio.h>
　　#include <stddef.h>
　　#include <time.h>
　　int main(void)
　　{
　　time_t timer;//time_t就是long int 类型
　　struct tm *tblock;
　　timer = time(NULL);
　　tblock = localtime(&timer);
　　printf("Local time is: %s\n",asctime(tblock));
　　return 0;
　　}
```

　　执行结果：
　　Local time is: Mon Feb 16 11:29:26 2009
程序例2：
　　上面的例子用了asctime函数，下面这个例子不使用这个函数一样能获取系统当前时间。
        需要注意的是年份加上1900，月份加上1。

   ```C
　　#include<time.h>
　　#include<stdio.h>
　　int main()
　　{
　　struct tm *t;
　　time_t tt;
　　time(&tt);
　　t=localtime(&tt);
　　printf("%4d年%02d月%02d日 %02d:%02d:%02d\n",
               t->tm_year+1900,t->tm_mon+1,t->tm_mday,t->tm_hour,t->tm_min,t->tm_sec);
　　return 0;
　　}
```

localtime()和gmtime()的区别：
　　gmtime()函数功能类似获取当前系统时间，只是获取的时间未经过时区转换。
　　localtime函数获得的tm结构体的时间，是已经进行过时区转化为本地时间。
 
localtime_r()和gmtime_r()函数

```C
　　struct tm *gmtime_r(const time_t *timep, struct tm *result); 
　　struct tm *localtime_r(const time_t *timep, struct tm *result);
```

　　gmtime_r()函数功能与此相同，但是它可以将数据存储到用户提供的结构体中。
　　localtime_r()函数功能与此相同，但是它可以将数据存储到用户提供的结构体中。它不需要设置tzname。
　　使用gmtime和localtime后要立即处理结果，否则返回的指针指向的内容可能会被覆盖。
       一个好的方法是使用gmtime_r和localtime_r，由于使用了用户分配的内存，这两个函数是不会出错的。
asctime()函数
　　功 能: 转换日期和时间为相应的字符串（英文简写形式，形如：Mon Feb 16 11:29:26 2009）
　　用 法: char *asctime(const struct tm *tblock);
 
ctime()函数
　　功 能: 把日期和时间转换为字符串。（英文简写形式，形如：Mon Feb 16 11:29:26 2009）
　　用 法: char *ctime(const time_t *time);
　　说 明：ctime同asctime的区别在于，ctime是通过日历时间来生成时间字符串，
                  而asctime是通过tm结构来生成时间字符串。
 
mktime()函数
　　功 能：将tm时间结构数据转换成经过的秒数（日历时间）。
　　原 型：time_t mktime(strcut tm * timeptr);。
　　说 明：mktime()用来将参数timeptr所指的tm结构数据转换成
                   从公元1970年1月1日0时0分0 秒算起至今的UTC时间所经过的秒数。
　　返回值：返回经过的秒数。

difftime()函数
　　功 能：计算时间间隔才长度，以秒为单位，且只能精确到秒。
　　原 型：double difftime(time_t time1, time_t time0);
　　说 明：虽然该函数返回值是double类型的，但这并不说明该时间间隔具有同double一样的精度，
                   这是由它的参数决定的。

strftime()函数
　　功 能：将时间格式化，或者说：格式化一个时间字符串。我们可以使用strftime()函数将时间格式化为我们想要的格式。
　　原 型：size_t strftime(char *strDest,size_t maxsize,const char *format,const struct tm *timeptr);
　　参 数：我们可以根据format指向字符串中格式命令把timeptr中保存的时间信息放在strDest指向的字符串中，
                   最多向strDest中存放maxsize个字符。
　　返回值：该函数返回向strDest指向的字符串中放置的字符数。
　　类似于sprintf()：识别以百分号(%)开始的格式命令集合，格式化输出结果放在一个字符串中。
                    格式化命令说明串strDest中各种日期和时间信息的确切表示方法。格式串中的其他字符原样放进串中。
                    格式命令列在下面，它们是区分大小写的。
　　%a 星期几的简写
　　%A 星期几的全称
　　%b 月份的简写
　　%B 月份的全称
　　%c 标准的日期的时间串
　　%C 年份的后两位数字
　　%d 十进制表示的每月的第几天
　　%D 月/天/年
　　%e 在两字符域中，十进制表示的每月的第几天
　　%F 年-月-日
　　%g 年份的后两位数字，使用基于周的年
　　%G 年份，使用基于周的年
　　%h 简写的月份名
　　%H 24小时制的小时
　　%I 12小时制的小时
　　%j 十进制表示的每年的第几天
　　%m 十进制表示的月份
　　%M 十时制表示的分钟数
　　%n 新行符
　　%p 本地的AM或PM的等价显示
　　%r 12小时的时间
　　%R 显示小时和分钟：hh:mm
　　%S 十进制的秒数
　　%t 水平制表符
　　%T 显示时分秒：hh:mm:ss
　　%u 每周的第几天，星期一为第一天 （值从0到6，星期一为0）
　　%U 第年的第几周，把星期日作为第一天（值从0到53）
　　%V 每年的第几周，使用基于周的年
　　%w 十进制表示的星期几（值从0到6，星期天为0）
　　%W 每年的第几周，把星期一做为第一天（值从0到53）
　　%x 标准的日期串
　　%X 标准的时间串
　　%y 不带世纪的十进制年份（值从0到99）
　　%Y 带世纪部分的十制年份
　　%z，%Z 时区名称，如果不能得到时区名称则返回空字符。
　　%% 百分号
　　提示：与 gmstrftime() 的行为相同，不同的是返回时间是本地时间。



localtime是不可重入函数，它直接返回strcut tm*指针（如果成功的话）；
这个指针是指向一个静态变量的；
因此，返回的指针所指向的静态变量有可能被其他地方调用的localtime改掉，例如多线程使用的时候。

localtime_r则是由调用者在第二个参数传入一个struct tm result指针，该函数会把结果填充到这个传入的指针所指内存里面；成功的返回值指针也就是struct tm result。localtme_r是可重入的。

其他的时间函数，如asctime，asctime_r；ctime，ctime_r；gmtime，gmtime_r都是类似的，所以，时间函数的 _r 版本都是线程安全的。