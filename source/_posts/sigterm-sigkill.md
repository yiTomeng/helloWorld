title: SIGTERM と SIGKILLの何年ぶりの初対面
date: 2015-02-27 10:02:29
tags: linux開発
---

linuxにはたくさんシグナルがあるだろう。その中にSIGTERMとSIGKILLはどこが違うのかを気になってて、とりあえず簡単で記載しとく。

SIGTERMはアプリを終了させるため、アプリに送る。アプリはSIGTERMシグナルを捉えてから、ハンドラで処理を行う。もしアプリ側でハンドラを設定しない場合、デフォルト処理を行う。それはアプリを終了処理とのことだ。でもSIGTERMの全体を把握できない状態で、デフォルト処理はどういうふうにアプリを閉じさせるかは知らないだろう。アプリの特別内容を保存せずにアプリを閉じちゃうかもしれない。もしDBとやりとり途中でアプリはSIGTERMを捉えると、DBのデータが混乱する可能性が高いだろう。そのため、個人的にはデフォルト処理を行うことはおすすめではない。

ざっと見ると、SIGKILLとSIGTERMはだいたい同じけど、両方ともはアプリを終了させるものだが、実際には致命的な差がある。それは、SIGKILLのシグナルは捉えないまま、強制終了との機能だ。

一般的にはSIGTERMは「kill pid」の形で、SIGKILLは「kill -9 pid」の形である。

以下の例を挙げろう:

```C
#include <stdio.h>
#include <signal.h>

void SigTerm(void);
void SigKill(void);

int main(int argc, char *argv[])
{
	
	if ( signal(SIGTERM,(void *)SigTerm) == SIG_ERR ){          //ソフトウェア終了シグナルハンドラの設定
		printf("sigterm\n");
        return(-1);
    }

    #if 0				//コメントアウトしないと、ifに入る
    if ( signal(SIGKILL,(void *)SigKill) == SIG_ERR ){          //ソフトウェア終了シグナルハンドラの設定
    	printf("sigkill\n");
        return(-2);
    }
    #endif
    int a = 0;
    while(1)
    {
    	a++;
    	sleep(1);
    }
	return 0;
}

void SigTerm(void)
{
	printf("SigTerm\n");
	return;
}

void SigKill(void)
{
	printf("SigKill\n");
	return;
}

```

上の例の結果：
kill　pidをしたら、「SigTerm」がターミナルに表示され、アプリはそのままで動き続ける。
kill -9 pidを入力したら、「強制終了」が表示され、アプリ終了される。
もしデフォルトSIGTERM処理であれば、kill pidを入力すると、「終了しました」とのメッセージが表示され、アプリは終了される。