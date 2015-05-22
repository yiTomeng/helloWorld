title: SimpleDateFormat
date: 2015-05-23 09:42:29
tags: java
---

今日spring junitを使って、単体テストした。最初はeclipseでjunitを使って、無事で終わったが、
jenkins上で自動単体テストするときは、エラー異常発生した。なぜと言うと、それは取得した日付と正しい日付は一致しないのだ。

eclipseでgradleによりテストを試した結果とjenkinsは一緒（それはもちろん、同じbuild.gradleを参照し、gradleでビルドしたからだ。）
そして、いろいろ試したり、調べたり、やっと問題点が発見した。その張本人はSimpleDateFormat関数だ。

修正前のソースは以下の通り：

```

SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");

```

そのロケール設定しないので、日付を"20150523"設定し、結果はFri May 23 00:00:00 JST 4003になってしまった。
なぜ年は変えただろう？今までまだ知らないんだ。
知っているのは、ロケールを追加し、問題が解決できる。

修正後のソースは以下の通り：

```

SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd", Locale.JAPAN);

```

修正後の結果は：
Sat May 23 00:00:00 JST 2015

具体的原因はまだわからないけど、とりあえず、記載しておく。