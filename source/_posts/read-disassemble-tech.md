title: C++反汇编与逆向分析技术揭秘(2)--第二章
date: 2015-03-05 09:42:29
tags: Read Tech
---

1.関数を呼び出したら、push ebpは実行されて、今のebpの指す内容は保存された前回のebpです。
  x86のスタックに関しては、普通は大きいアドレスから小さいアドレスへ進んでる。

2.いくつのfloat型に関するx86アセンブリコマンド：
fld fild fldz fld1 fst fstp fist fistp fcom ftst fadd faddp

3.float型は戻り値とする時、eaxには保存できないため、ST(0)に保存する

4.x86に対して、mov後ろの操作数に[]があるときは、その[]中の結果をアドレスとして、そのアドレス指してる内容を取得する。
そういえば、[]は計算できるものという意味だ。
例えば、mov eax, ebx + 16h   はダメだが、
mov eax, [ebx + 16h]は正しい、その意味はebx + 16hのアドレスにあるデータをeaxに保存する。

もう一つはleaのことだ。もしlea eax, [ebx + 16h]の場合、ebx + 16hの値を直接eaxに保存する。

5.まず、ソースを見ましょう：

```C
#include <stdio.h>

int main(void)
{
	const int a = 5;

	//a = 7;					//NG

	int *pa = (int *)&a;

	*pa = 6;

	int ta = a;

	printf("a:%d  pa:%d   ta:%d\n", a, *pa, ta);

	const int b;

	int *pb = (int *)&b;

	*pb = 8;

	int tb = b;

	printf("b:%d  pb:%d   tb:%d\n", b, *pb, tb);

	return 0;

}

```

microsoft visual c++6.0にて、constの処理はこうなると書いてある(結果)：
a:6  pa:6   ta:5
b:8  pb:8   tb:8
taは5の理由はコンパイラにより、int ta = a → int ta = 5に変換したからだ。tbは8の理由は、aは初期化され、bは初期化されてないからだ。
で、私はlinux(asianux)で実行すると、結果はこうなる：
a:6  pa:6   ta:6
b:8  pb:8   tb:8

とりあえず、constを付けても、変数は変数の事実は変わらない。コンバイラはいくら頑張って止めてもポインタを利用して、変更の内容を変えることができる。

ソース上でa=7　　NGと書いてあるんだ。それはコンバイラを通らなく、エラーメッセージが表示された。



