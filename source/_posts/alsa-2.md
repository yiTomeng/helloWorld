title: alsaの勉強（2）ーー alsaについて簡単な説明
date: 2015-03-01 16:01:59
tags: linux開発
---

##一、はじめに
linux音声再生の一つOSSは前編で説明した。確かにOSSを利用して、いろいろな処理を行うことができるけど、直接デバイスを扱うなんか、違和感が感じられる。だから、ほかのできたものがあるかどうかを探す必要がある(OSSを使うなら特に問題ないと思うけど、日本でソフト開発に向けて、一番重要なのは安全性だから、OSSを使ったら、OSSの仕組みちゃんと理解していない状況で、心配をかけちゃう）。そのため、調査した結果はalsaを用いることだ。

##二、alsaは何でしょう？
alsaは「Advanced Linux Sound Architecture」の略称である。これはサウンドカードのデバイスドライバを提供するOpen Sound System (OSS)を置き換えるために開発されたLinuxカーネルコンポーネントである。ALSAプロジェクトの初期の目標は、サウンドカードハードウェアの自動設定や、複数のサウンドデバイスのスマートな取扱いなどであったが、それらは概ね達成された。開発になかなか関係なく詳しい内容は[alsa wiki](http://ja.wikipedia.org/wiki/Advanced_Linux_Sound_Architecture)

##三、alsaの仕組みにおいて、簡単な説明
alsaはlinuxのコンポーネントであることを特に知っているはずなので、linuxの特徴を持っているに決まっているなのでしょう。linuxには、ハードウェアに対して、ドライバーが必要となるのは当然だ(ほかのOSも同じだろうね）。だから、alsa driverを提供している(open source）。なぜalsaは人気になるのか？その１つはアプリAPIを提供すること。alsa libをインストールしてから、alsaのAPIを使うことができる。(僕の環境「asianux」には、最初から搭載しているので、わざわざインストール必要はない。alsa libのインストールに関しては、別途記載するかもしれない）。alsaの仕組みは以下の画像のように：
![alsa arch](/pic/alsa-arch1.png)
そのため、アプリを作る時は、lib中のAPI関数を呼び出したら、driverを通じて、ハードウェアをコントロールできる。
以上の内容は大雑把で見るものだ。詳しい内容を知るため、深く追求しないとわからない。時間はないので、それは後回しにする。

##四、四つ基本な操作
linuxにはデバイスにとって、ファイルと同じように扱うので、基本的にはopen(),close(),write(),read()四つ基本な操作があるだろう。今からちょっと詳しく説明しましょう。
①　open()、close()
この二つはよくペアとして現れるだろう。まずopen()関数を呼び出して、デバイスのハンドルを取得する。それから、このデバイスに対しての処理を行える。もちろん、open()関数の戻り値の判断が必要なのだ。処理は終わってから、close()関数を呼び出すことは絶対必要なのだ。
OSSについて、/dev/dspデバイスは一回しか開けない(一つアプリに複数openした、できない)。つまり、open()を一度呼び出すと、close()関数を呼び出す前に、ほかの場所でopen()関数は効かない。もし多数音声ファイルを同時に放送したい場合、/dev/mixerを使ったら、目的を達成することができる。それにしても、設定はちょっとややこしい。
ALSAでは、何回open()しても、問題にならないです(別々のアプリでopen()関数を呼び出すと、同時に放送できた。)。libがある限り、ハードウェアの方は考えなくてもいい。

②　read()、write()
read()はデバイスからデータを読み込む関数だ。つまり、録音用な関数だ。write()とread()関数は逆だ。write()はデバイスに書き込む関数だ。
今回のプロジェクトについては、read()はつかわない、放送だけだから、write()を使っている。そのため、write()関数を使ったとき仕組みを説明しましょう（自分の理解だね）。
サウンドカードにはハードバッファがある。それはring bufferである。
理解ⅰ「いろいろな資料から理解した内容」：write()関数を呼び出すときは、まず、データをもう一つバッファ(名前はランタイムバッファ）に入れる。ハードバッファに空き領域があれば、DMA経由で、ハードバッファに入れる。もし、今write()で一気に500バイトを書き込もうとしたら、そのときハードバッファに空き領域は400バイト、そうすると、戻り値は400バイト、残り100バイトはランタイムバッファに残っている。もし1200バイトを書き込もうとしたら、ハードウェアバッファには400バイト空き領域しかない、そのとき、戻り値は800かもしれない、原因としては、書き込むには時間がかかる、その途中で、デバイスはどんどん放送するため、空き領域増えている、限界(周期サイズというもの、今後説明する予定）になってから、まだランタイムに通知し、パッファに書き込むから。

理解ⅱ「先見たばかり」：[new opinion](http://www.linuxidc.com/Linux/2012-07/65903p2.htm)

read()とwrite()はもう一つ問題がある。それはアプリ側で操作スピードとハードウェアの処理スピードが合うか合わないかの問題だ。
ちょっと想像しろ：
read()の場合、アプリ側で読み込むスピードが早すぎると、ハードウェアバッファに昔のデータはもう読み込んだ、最新のデータまだこない、この場合はアプリ側読み込むことができないだろう。alsaにそれはoverrunと呼ぶ。
write()の場合、アプリ側の書き込むスピードが遅すぎると、ハードウェアバッファに前書き込んだデータを特に放送終わっちゃったから、ハードウェア現在放送できなくなる、もしくは、変なデータが放送しちゃう(ring bufferのため、前のデータ再放送するかもしれない）。alsaにそれはunderrunと呼ぶ。
overrunはunderrunとともにXRUNを呼ぶ。

##五、なぜalsaを使う
一つネットwebsiteを挙げましょう：[alsa advantage](http://www.linuxjournal.com/article/7391?page=0,0)
alsaはossにより、レベル高いlibを用意したので、ハードの方は考えずに使える。
ossで書き込むと読み込むのスピードをコントロールしにくい。alsaにはコントロールすると、周期サイズを設定すれば問題なくなる。そしたら、放送や録音には一番厄介なものが解決された。

以上にはalsaに対して簡単で説明した。私にとって、全体(libやdriverソース）を見ないと、断言できないと思う。ざっと見たら、以上のような動きだ。今後は証明するかもしれない。とりあえず、今は記載しとく、深く内容はまだ知らないけど、特に問題ないはずだ。

