title: 共有メモリ
date: 2015-02-26 12:45:36
tags: linux
---

現在linux上で多数プロセスと多数スレッドのアプリがどんどん現れる。そうすると、関連プロセス間に通信必要な場合がおおい。最近のプロジェクトで目に遭うので、とりあえずメモしとく。
linuxにおいてプロセス間の通信といえば、IPCはよく聞こえる。それはlinuxのプロセス間通信との意味だ。IPC通信方法はいくつある。例えば、共有メモリ、パイプ、ソケット通信など。最近使っているのは共有メモリとソケット通信二つ。ここは共有メモリをメモしとく。(ソケットは別途でメモするかも）

簡単にいえば、共有メモリ通信は各プロセスアクセスできるメモリであるため、プロセス間に通信できる。
linuxには共有メモリに関連するapi関数は:shmget(),shmat(),shmdt(),shmctl();
shmget関数、shmat関数、shmdt関数、shmctl関数は、共有メモリの操作を行ないます。共有メモリとは、システムが管理しているメモリの一部を共有して、複数のプロセスがプロセス間通信を行う技法です。複数のプロセスが共有しますので、プロセス間の待ち合わせや排他制御が必要な場合がありますが、その場合は[セマフォ](http://www.c-lang.net/semop/index.html)を使用するとよいでしょう。

共有メモリは、次の手順で操作します。
関数を使う前に、必ず<sys/shm.h>をincludeすること（<sys/ipc.h>は必要かどうかテストしてない）。
shmget関数で共有メモリ・セグメントの識別子（セグメントID）を取得します。なお、共有メモリ・セグメントは新規作成もできます。
それぞれのプロセスは、shmat関数で自プロセスのアドレス空間に共有メモリ・セグメントを付加（アタッチ）します。これで、共有メモリへのデータの読み書きが可能になります。
それぞれのプロセスで共有メモリが不要になったら、shmdt関数で自プロセスのアドレス空間に配置された共有メモリ・セグメントを分離（デタッチ）します。
shmctl関数で共有メモリ・セグメントを破棄します。
これらの関数は、Ｃ言語のライブラリ関数（標準関数）ではありませんので、コンパイラにより、使えない場合があります。

```C
#include <sys/shm.h>
int shmget(key_t key， size_t size， int shmflg);
```

keyは共有メモリ・セグメントに対するキーを指定します。0(IPC_PRIVATE)の場合、新しい共有メモリを作る。もし0以上の場合、shmflg次第だ。普通はftok()の返却値である。
sizeは作成する共有メモリ・セグメントの大きさをバイト単位で指定します。なお、指定した値はヘッダファイルに定義してあるPAGE_SIZEの値の倍数に切り上げた（round up）大きさになります。0以上の場合、新規のサイズである。0の場合は共有メモリを取得するだけの意味だ。
shmflgはオプションを指定します。0：共有メモリ識別子を取得、なければエラー発生する。IPC_CREAT:もしshmflg&IPC_CREATはTRUEの場合、カーネルにはkeyと同じキーがなければ作る、あれば、共有メモリの識別子を返す。IPC_CREAT|IPC_EXCL：カーネルにはkeyと同じキーがなければ共有メモリを作る、あれば、エラー発生する。

戻り値として、正常終了した場合はセグメント識別子（セグメントID）を、エラーの場合は-1を返します。なお、セグメントIDは正の整数値です。

以下はほかのところからコピーしてきた内容を見せる(参照できると思う）：
{
新規に共有メモリ・セグメントを作成するには、次の２つの方法があります。

第１引数のkeyにIPC_PRIVATEを指定します。
第１引数のkeyにユニークな値を指定し、第３引数のshmflgにIPC_CREATを指定します。なお、shmflgにIPC_CREATとIPC_EXCLを指定すると、keyに対する共有メモリ・セグメントが既に存在していた場合にエラーになります。
第３引数のshmflgの下位９ビットは、その共有メモリ・セグメントの所有者、グループ、他人に対するアクセス許可の定義として使用します。


```C
#include <sys/shm.h>
void *shmat(int shmid， const void *shmaddr， int shmflg);
int shmdt(const void *shmaddr);
```

shmidは共有メモリ・セグメントに対するセグメント識別子（セグメントID）を指定します。
*shmaddrは共有メモリ・セグメントを付加（アタッチ）するアドレスを指定します。
shmflgはオプションを指定します。SHM_RDONLYを指定すると、共有メモリ・セグメントを読み込み専用で付加します。

戻り値として、正常終了した場合は付加した共有メモリ・セグメントのアドレスを、エラーの場合は(void *)-1を返します。

第２引数の*shmaddrにNULLを指定すると、システムは共有メモリ・セグメントを適切な（使用されていない）アドレスに付加します。また、NULLでないアドレスを指定して、第３引数のshmflgにSHM_RNDを指定すると、指定したアドレスはセグメントの境界アドレスの最小倍数（SHMLBA）に切り捨てた（rounding down）アドレスになります。その他の場合は、ページ境界のアドレスでなければなりません。

```C
#include <sys/shm.h>
int shmctl(int shmid， int cmd， struct shmid_ds *buf);
```

shmidは共有メモリ・セグメントに対するセグメント識別子（セグメントID）を指定します。
cmdは共有メモリ・セグメントに対する制御命令を指定します。
*bufは共有メモリ・セグメントに関する情報が格納されているhmid_ds構造体へのポインタを指定します。

戻り値として、正常終了した場合は０を、エラーの場合は-1を返します。

第２引数のcmdには、次のようなコマンドを指定できます。

コマンド	内容
IPC_STAT	カーネルデータ構造体の情報を第３引数の*bufで指定されたshmid_ds構造体にコピーします。
IPC_SET	第３引数の*bufで指定されたshmid_ds構造体のいくつかのメンバの値を、カーネルデータ構造体に設定します。
IPC_RMID	共有メモリ・セグメントを破棄します。実際には破棄済みのマークだけを付けておき、関連する全てのプロセスが共有メモリ・セグメントを分離（デタッチ）したら破棄します。
次の例題プログラムは、子プロセス１が共有メモリにメッセージを書き込み、子プロセス２がそれを表示しています。

プログラム　例

```C
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <sys/shm.h>
#include <string.h>

int main()
{
int                shmid;       /* セグメントID */
int                child_cnt;

/* 共有メモリ・セグメントを新規作成 */
if ((shmid = shmget(IPC_PRIVATE， 100， 0600)) == -1){
perror('main : shmget ');
exit(EXIT_FAILURE);
}

/* １つ目の子プロセスを生成 */
if (fork() == 0) {
/* 子プロセス */
char      *shmaddr;

printf('子プロセス１開始\n');

/* 共有メモリ・セグメントをプロセスのアドレス空間に付加 */
if ((shmaddr = shmat(shmid， NULL， 0)) == (void *)-1) {
perror('子プロセス１ : shmat ');
exit(EXIT_FAILURE);
}

/* 共有メモリ・セグメントに文字列をコピー */
strcpy(shmaddr， 'Hello.\nBy.\n');

/* 共有メモリ・セグメントを分離 */
if (shmdt(shmaddr) == -1) {
perror('子プロセス１ : shmdt ');
exit(EXIT_FAILURE);
}

printf('子プロセス１終了\n');
exit(EXIT_SUCCESS);
}

/* ２つ目の子プロセスを生成 */
if (fork() == 0) {
/* 子プロセス */
char      *shmaddr;

printf('子プロセス２開始\n');

/* 待ち合わせ */
sleep(1);

/* 共有メモリ・セグメントをプロセスのアドレス空間に付加 */
if ((shmaddr = shmat(shmid， NULL， SHM_RDONLY)) == (void *)-1) {
perror('子プロセス２ : shmat ');
exit(EXIT_FAILURE);
}

printf('子プロセス２：共有メモリの内容を表示します\n');
/* 共有メモリ・セグメントの内容を表示 */
printf('%s'， shmaddr);

/* 共有メモリ・セグメントを分離 */
if (shmdt(shmaddr) == -1) {
perror('子プロセス２ : shmdt ');
exit(EXIT_FAILURE);
}

printf('子プロセス２終了\n');
exit(EXIT_SUCCESS);
}

/* 親プロセス。子プロセスの終了を待つ */
for (child_cnt = 0; child_cnt < 2; ++child_cnt) {
wait(NULL);
}

/* 共有メモリ・セグメントを破棄 */
if (shmctl(shmid， IPC_RMID， NULL) == -1){
perror('main : shmctl ');
exit(EXIT_FAILURE);
}

printf('親プロセス終了\n');
return EXIT_SUCCESS;
}
例の実行結果

$ ./shm.exe
子プロセス１開始
子プロセス１終了
子プロセス２開始
子プロセス２：共有メモリの内容を表示します
Hello.
By.
子プロセス２終了
親プロセス終了
$
}
```
