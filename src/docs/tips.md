---
layout: base
title: その他の注意点
eleventyNavigation:
  key: その他の注意点
  subtitle: その他のTIPSなど
  parent: 高度な使い方
  order: 15
---

## その他の注意点

* [テンプレートに ${ } を書きたい](#dollar)
* [出力する文字エンコーディングを指定したい](#charset)
* [Java から出力した "～" が文字化けする](#japanese)
* [<, >, &amp; などを書きたい](#escape)


### テンプレートに ${ } を書きたい {#dollar}

Mayaa ではスクリプトを書くときに `${ }` という表記を使います。テンプレート上でも mayaa ファイル上でも、この表記をした場合にはスクリプトとして扱います。

そのためテンプレート上に `${ }` という文字列を書きたい場合、`&amp;#36;{ }` とエスケープして書く必要があります。



### 出力する文字エンコーディングを指定したい {#charset}

Mayaa で出力する文字エンコーディングとテンプレート読み込み時の文字エンコーディングを決定するのは、テンプレートの文字エンコーディング指定です。HTML テンプレートであれば META タグでの指定、XML テンプレートであれば XML としての文字エンコーディング指定です。

文字エンコーディングを指定しない場合は `UTF-8` として扱います。文字エンコーディングはトラブルの元となりやすいため、基本的には明示的に文字エンコーディング指定をするようにしてください。

`Shift_JIS` のテンプレートを使う場合、Windows で編集したファイルであれば<a href="/docs/settings/" title="エンジン設定方法">エンジン設定</a>でパラメータ "`convertCharset`" の値を `true` に設定し、テンプレートの META タグで "`Windows-31J`" を指定するようにしてください。これにより文字化けを防ぎ、かつ "`Shift_JIS`" として出力するため IE や携帯電話でも正しく文字エンコーディングを認識できるようになります。

*※1.1.3 で変更しました。* <strike>文字エンコーディングを指定しない場合、HTML と XML で動作が異なります。HTML の場合、`UTF-8` と自動判定できる場合は `UTF-8` として、それ以外は `ISO-8859-1` として扱います。XML の場合は `UTF-8` として扱います。</strike>


### Java から出力した "～" が文字化けする {#japanese}

日本語を扱う場合、文字エンコーディングの指定によっては文字化けが発生することがあります。文字化けを回避するには、Java ソースコードや他のリソースと同じ文字エンコーディングを使うようにしてください。


たとえば Windows で開発する場合、一般的に Java ソースコードの文字エンコーディングは `MS932` です。この場合に HTML テンプレートに指定する文字エンコーディングは `Windows-31J` です。 **Shift_JIS ではありません** 。テンプレートの文字エンコーディングを `Shift_JIS` にした場合、Java ソースコード中に書いた "～" の文字を出力しようとすると文字化けが発生します。

### <, >, & などを書きたい {#escape}

テンプレート、mayaa ファイルの両方とも、`<`, `>`, `&` などを文字として使いたい場合は `&lt;`, `&gt;`, `&amp;` のようにエスケープする必要があります。

ちょっとややこしくなりますが、mayaa ファイルからテンプレートに「エスケープ後の文字列」を出力する場合、もう一段階エスケープしなければなりません。(`&amp;lt;`, `&amp;gt;`, `&amp;amp;`)