title: 関数の引数の特別な書き方
date: 2015-03-04 09:42:29
tags: C
---

今日はglibcのソースにこんなものを見た：

```C
char *
_itoa (value, buflim, base, upper_case)
     unsigned long long int value;
     char *buflim;
     unsigned int base;
     int upper_case;
{
	... ...
}
```

見た時自分でもびっくりした。これって、できるかなと思ったんだけど、いつ見たことある気がする。それって、一回テストソースを作ってやてみたら、やっぱり問題ない。今後びっくりさせないため、記載しとく。
テストソースは以下のように：

```C
#include <stdio.h>

int add(a,b)
	int a; int b;
	{
		int sum = a + b;
		return sum;
	}


int main(void)
{
	int a = 10;
	int b = 20;

	printf("%d\n", add(a, b));
	return 0;
}

```


