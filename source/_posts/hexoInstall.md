title: hexo Install
date: 2015-01-27 09:58:36
tags:　hexo Install
---
こんにちは、エンジニアののびすけです。静的サイトをいやらしい風にtypoする今日この頃です。

前回、Github Pagesを使ったWebページ公開の記事「Git初心者でも大丈夫！完全無料でGithub PagesにWebページを公開する方法」を書きました。

Github Pagesではデータベースやサーバーサイドプログラムを使うことができないので、弊社ブログのようにWordPressを使ってブログ構築をすることはできません。

そこで今回は静的サイトジェネレータとGithub Pagesを組み合わせることで無料でブログを公開する方法を紹介します。
静的サイトジェネレータ

静的サイトジェネレータとはコマンドラインでのカンタンな操作でHTML/CSS/JavaScriptなどを生成し、Webページ作成を少ない手間で作ることができるツールの総称です。

静的なページにすることで、

    セキュリティリスクが少ない
    SEOに強い
    表示速度が高速

といったメリットもあります。

JekyllというRuby製のツールが今のところ最も有名ですが、静的サイトジェネレータは色々な人が作っていて、乱立している状態です。

StaticSiteGeneratorsというサイトでは、現時点での静的サイトジェネレータ一覧を見ることができます。2014/7/29現在では289種類ものジェネレータが存在するみたいです。

これらのツールはGithub上でソースコードを公開していて、Githubのスター(ブックマークのようなもの)の数でのランキングを見てみると、

    Jekyll
    Octopress
    Pelican
    Middleman
    Hexo

という順番で人気があることが分かります。今回はこの中でもJavaScript製のHEXOを利用してみたいと思います。

参考：Static Site Generators

http://staticsitegenerators.net/
HEXO

人気上位のJekyllやOctopressはRuby製のツールですが、私は全てJavaScriptで完結させたいと思っているので、Node.js製のHEXOを利用してみます。
（HEXO以外にもmetalsmith、DocPad、Harp、Wintersmith、assembleなどのツールもNode.jsで作られていますが、今回は割愛します。）

HEXOは数ある静的サイトジェネレータの中でもブログ生成に特化しています。記事はMarkDown方式で書くことができ、カンタンなコマンドでページの追加やテーマ、プラグインのインストールができます。

参考：HEXO

http://hexo.io/

参考：JekyllからHexoに移行した｜じまぐてっく

http://nakajmg.github.io/blog/2014-07-21/change-generator.html
HEXOで作ったブログを2分でGithub Pagesへ公開
準備

まずは、以下の環境を揃えてください。

    Node.jsが動く環境
    Github Pagesのリポジトリ

Node.jsのインストールをしていない人は、こちらの記事を参考にしてNode.jsをインストールしてください。

「いまアツいJavaScript！ゼロから始めるNode.js入門〜5分で環境構築編〜」

Github PagesでWebページ公開をしたことがない人は、こちらの記事を参考にしてGithub Pagesのリポジトリを作ってみましょう。

参考：Git初心者でも大丈夫！完全無料でGithub PagesにWebページを公開する方法
http://liginc.co.jp/web/html-css/html/96453
HEXOを使い1分でブログ生成

HEXOをインストールしましょう。npm installで簡単にインストールできます。

	
$ npm install -g hexo

次にブログを作ってみます。hexo init [プロジェクト名]でブログのひな形を生成できます。今回はmyblogという名前で作ってみます。

	
$ hexo init myblog

myblogというフォルダが生成されます。

myblogフォルダへ移動して必要なモジュールをインストールします。

	
$ cd myblog
$ npm install

たったこれだけで準備は完了です。

ローカルサーバーを立ち上げてみましょう。

	
$ hexo server
[info] Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

この状態でブラウザからhttp://localhost:4000/にアクセスしてみましょう。

こんな感じでブログができ上がっていると思います。

めちゃ簡単ですね！ そして爆速！

確認ができたら、

Ctrl ＋C

でサーバーを止めることができます。
HEXOで作ったブログをGithub Pagesで公開する

上記の手順でブログ生成ができたので、このブログを世界に向けて公開してみましょう。

lsコマンドでmyblogフォルダの中を確認すると_config.ymlというファイルが生成されていることがわかります。

	
$ ls
_config.yml  node_modules scaffolds    themes
db.json      package.json source

HEXOではこの_config.ymlが設定ファイルになっていて、このファイルにGithub Pagesの設定を記述していきます。

エディタで_config.ymlを編集しましょう。

	
$ vi _config.yml

デフォルトの状態はこのような記述になっています。

	
・・・(省略)・・・
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type:

これを以下のように編集しましょう。

	
・・・(省略)・・・
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: github
  repo: git@github.com:n0bisuke/n0bisuke.github.io.git
  branch: master

repoの部分は自分のGithub Pagesのリポジトリに合わせて適宜変更してください。
設定が完了したら、以下のコマンドでデプロイをします。
generateというコマンドで静的HTMLをpublicフォルダ内に生成することができるのですが、deployコマンドに-gオプションを付けることで、これを同時に行ってくれます。

	
$ hexo deploy -g

エラーが出なければ成功です。自分のGithub Pagesのページを見てみましょう。

私のGithub PagesでもHEXOでブログを作ってみたので、デモ画面として確認してみてください。
記事を追加してみる

HEXOでは記事の追加も凄まじく簡単です。

	
$ hexo new &quot;新規ページタイトル&quot;

これだけで記事の追加ができます。

例えば “hexoについて”という記事ページを追加する場合は以下の通りです。

	
hexo new &quot;hexoについて&quot;
[info] File created at /Users/nobisuke/Sites/myblog/source/_posts/hexoについて.md

source/_postsフォルダに”hexoについて.md”というMarkDownファイルが生成されていると思います。
あとはこのMarkDownファイルを編集すれば記事の詳細を書くことができます。

	
$ vi source/_posts/hexoについて.md

MarkDownファイルを開くと以下のように記述されています。

	
title: hexoについて
date: 2014-07-29 19:29:05
tags:
---
(ここに本文を記述)

titleとdateにはあらかじめメタ情報が付与されていると思います。
tagsには任意のタグを設定することができます。今回はhexoというタグを付けてみました。
本文は—の下にMarkDownで書いていきます。HTMLでも記述可能です。

	
title: hexoについて
date: 2014-07-29 19:29:05
tags: hexo
---
#hexoとは
静的サイトジェネレータの1つです。
 
#シンタックス
[Qiita](http://qiita.com)のようにシンタックスハイライトされます。
 
```server.js
 
var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(1337, '127.0.0.1');
console.log('Server running at http://127.0.0.1:1337/');
 
```
&amp;amp;amp;amp;lt;p&amp;amp;amp;amp;gt;
記事の編集が終わったら、ローカル環境で確認してみましょう。
&amp;amp;amp;amp;lt;/p&amp;amp;amp;amp;gt;
&amp;amp;amp;amp;lt;p&amp;amp;amp;amp;gt;
先程と同じように&amp;amp;amp;amp;lt;code class=&quot;command&quot;&amp;amp;amp;amp;gt;hexo server&amp;amp;amp;amp;lt;/code&amp;amp;amp;amp;gt;コマンドでローカルサーバーを立ち上げて確認しましょう。
&amp;amp;amp;amp;lt;/p&amp;amp;amp;amp;gt;
&amp;amp;amp;amp;lt;p&amp;amp;amp;amp;gt;
実は&amp;amp;amp;amp;lt;code class=&quot;command&quot;&amp;amp;amp;amp;gt;hexo server&amp;amp;amp;amp;lt;/code&amp;amp;amp;amp;gt;は&amp;amp;amp;amp;lt;code class=&quot;command&quot;&amp;amp;amp;amp;gt;hexo s&amp;amp;amp;amp;lt;/code&amp;amp;amp;amp;gt;としても同じ動作をするのでショートカットとして使うと良いと思います。
同じように&amp;amp;amp;amp;lt;code class=&quot;command&quot;&amp;amp;amp;amp;gt;hexo deploy&amp;amp;amp;amp;lt;/code&amp;amp;amp;amp;gt;は&amp;amp;amp;amp;lt;code class=&quot;command&quot;&amp;amp;amp;amp;gt;hexo d&amp;amp;amp;amp;lt;/code&amp;amp;amp;amp;gt;だけで大丈夫です
&amp;amp;amp;amp;lt;/p&amp;amp;amp;amp;gt;
 

$ hexo s

http://localhost:4000 で確認が終わったらGithub Pagesへデプロイしましょう。

	
$ hexo d -g

これで記事の追加や編集の流れが一通りできるようになり、ブログ構築ができました。
ホントに一瞬だったと思います。

完成したページはこちらです。
補足: HEXO Tips

ここからは、HEXOについてここまでで紹介しなかった、でも役立ちそうなその他のTipsを紹介します。
テーマ

HEXOでは色々なテーマが公開されていて、自由に使うことができます。公開されているテーマを編集してオリジナルテーマの作成も簡単です。この辺りもHEXOを使うメリットだと思います。
デフォルトではLandscapeというテーマが適用されています。中国でHEXOが人気なのか、なぜか中国語のものが多いようです。

参考：公開テーマ一覧

https://github.com/hexojs/hexo/wiki/Themes