---
layout: base
title: エンジンの設定
eleventyNavigation:
  key: エンジンの設定
  order: 6
tags: tutorial
---

## {{ title }}

### ServiceProviderの設定

Mayaa エンジンの設定には、`org.seasar.mayaa.provider.ServiceProvider` というファイル名のXMLファイルを使います。
v1.3.0以降はこのファイルは下記の順序で配置先を確認して最初に見つかった場所のものが採用されます。
v1.2 までは存在しているファイルを逆の順序ですべて読み込み、最後に読み込んだものだけが残ります。

1. `WEB-INF` の直下
2. `META-INF` の下
3. ビルトイン（mayaaのJARファイル内に含まれるもの）

ファイルを置く場所
```
WEB-INF/
 |-- org.seasar.mayaa.provider.ServiceProvider   ①
 +-- classes/
 |    +-- META-INF/
 |    |    |
 |    |    +-- org.seasar.mayaa.provider.ServiceProvider ②
 |    |
 |    +-- (クラスファイルなど)
 |
 +-- lib/
 |    +-- xxxxxxx.jar
 |    |    +--　META-INF/
 |    |         +-- org.seasar.mayaa.provider.ServiceProvider ②
 |    +-- mayaa-[version].jar　③
 |
 (省略)
```

#### ■何もしない設定ファイル

これは何もしない設定ファイルです。この中に各種の設定を書くことで、Mayaa の動作を設定できます。

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE provider
    PUBLIC "-//The Seasar Foundation//DTD Mayaa Provider 1.0//EN"
    "http://mayaa.seasar.org/dtd/mayaa-provider_1_0.dtd">
<provider>

</provider>
```

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE provider
    PUBLIC "-//The Seasar Foundation//DTD Mayaa Provider 1.0//EN"
    "http://mayaa.seasar.org/dtd/mayaa-provider_1_0.dtd">
<provider>
    <engine>
        <!-- Mayaa Engineに関する設定 -->
    </engine>
    <scriptEnvironment>
        <!-- Mayaa内のスクリプトエンジンから参照できるプロパティのスコープ定義など -->
    </scriptEnvironment>
    <specificationBuilder>
        <!-- Mayaa内のスクリプトエンジンから参照できるプロパティのスコープ定義など -->
    </specificationBuilder>
    <libraryManager>
    <templateBuilder class="org.seasar.mayaa.impl.builder.TemplateBuilderImpl">
    <pathAdjuster class="org.seasar.mayaa.impl.builder.PathAdjusterImpl">
    <templateAttributeReader>
    <parentSpecificationResolver class="org.seasar.mayaa.impl.engine.specification.ParentSpecificationResolverImpl">

</provider>
```

`~`は`org.seasar.mayaa`を表します。
| エレメント名 | 要求されるインタフェース                      | |
|------------|-------------------------------------------|--------------------|
| engine     | org.seasar.mayaa.engine.Engine            <br>[org.seasar.mayaa.impl.engine.EngineImpl](#engine) ||
| parentSpecificationResolver | org.seasar.mayaa.engine.specification.ParentSpecificationResolver <br> [org.seasar.mayaa.impl.engine.specification.ParentSpecificationResolverImpl](#parentSpecificationResolver) ||
| scriptEnvironment       | org.seasar.mayaa.cycle.script.rhino.ScriptEnvironment <br>[org.seasar.mayaa.impl.cycle.script.rhino.ScriptEnvironmentImpl](#scriptEnvironment) |
| templateAttributeReader | org.seasar.mayaa.builder.library.TemplateAttributeReader <br>[org.seasar.mayaa.impl.builder.library.TemplateAttributeReaderImpl](#templateAttributeReader) |
| specificationBuilder    | org.seasar.mayaa.builder.SpecificationBuilder <br>[org.seasar.mayaa.impl.builder.SpecificationBuilderImpl](#specificationBuilder) |
| templateBuilder         | org.seasar.mayaa.builder.TemplateBuilder<br>[org.seasar.mayaa.impl.builder.TemplateBuilderImpl](#templateBuilder) |
| pathAdjuster            | org.seasar.mayaa.builder.PathAdjuster<br>[org.seasar.mayaa.impl.builder.PathAdjusterImpl](/docs/settings/path_adjust_settings/) |
| libraryManager          | org.seasar.mayaa.builder.library.LibraryManager<br>[org.seasar.mayaa.impl.builder.library.LibraryManagerImpl](/docs/settings/library_manager/) |
| templateAttributeReader | org.seasar.mayaa.builder.library.TemplateAttributeReader<br>[org.seasar.mayaa.impl.builder.library.TemplateAttributeReaderImpl](#templateAttributeReader) |


#### ■Engine のパラメータ {#engine}

Mayaa 標準の Engine に対するパラメータをいくつか説明します。デフォルト設定は下記の通りです。


```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE provider
    PUBLIC "-//The Seasar Foundation//DTD Mayaa Provider 1.0//EN"
    "http://mayaa.seasar.org/dtd/mayaa-provider_1_0.dtd">
<provider>
    <engine>
        <parameter name="defaultSpecification" value="/default.mayaa"/>
        <parameter name="checkTimestamp" value="true"/>
        <parameter name="suffixSeparator" value="$"/>
        <parameter name="welcomeFileName" value="index.html"/>
        <parameter name="requestCharacterEncoding" value="UTF-8"/>
        <parameter name="surviveLimit" value="5"/>
        <parameter name="forwardLimit" value="10"/>
        <parameter name="convertCharset" value="false"/>
    </engine>
</provider>
```

`defaultSpecification`
: <a href="/docs/default/">すべてのページ共通に利用する Mayaa ファイル</a>のパスを指定します。デフォルトは "`/default.mayaa`" です。

`checkTimestamp`
: HTML ファイル、Mayaa ファイルの更新を検知して再ビルドする機能を有効にするかどうかを設定します。"`true`" なら有効、"`false`" なら無効です。デフォルトは "`true`" です。

`suffixSeparator`
: <a href="/docs/template_suffix/">テンプレート切り替え機能</a>でページ名と suffix とを区別するための文字列を指定します。デフォルトは "`$`" です。

`requestedSuffixEnabled`
: <a href="/docs/template_suffix/">テンプレート切り替え機能</a>で suffix を明示したアクセスを許すかどうかを設定します。アクセス時に明示した suffix は最優先になります。デフォルトは "`false`" です。

`welcomeFileName`
: URL の最後が "`/`" でアクセスされた場合にリダイレクトする先のファイル名を指定します。デフォルトは "`index.html`" です。

`requestCharacterEncoding`
: Mayaa でリクエストパラメータを参照する際の文字コードを指定します。デフォルトは "`UTF-8`" です。`MayaaServlet` の前に `HttpServletRequest#setCharacterEncoding(String encoding)` でセットされている場合はこの設定を使用しません。

`surviveLimit`
: ページのビルド結果をガーベジコレクションの対象にするまでの回数を指定します。デフォルトは "`5`" です。ここで指定した回数だけ解放要求が来たときに初めて、ページのビルド結果オブジェクトがガーベジコレクションの対象になります。また値として "`0`" を設定した場合、ビルド結果はガーベジコレクションの対象になりません。

`forwardLimit`
: 一回のレンダリング中に Mayaa の同一パスへ forward できる上限回数を指定します。デフォルトは "`10`" です。このパラメータは記述ミスで無限ループしてしまうのを避けるためにあります。

`convertCharset`
: テンプレートの meta タグで charset を "`Windows-31J`" としたとき、出力する HTML および HTTP レスポンスヘッダの Content-Type の charset として "`Shift_JIS`" を出力するかどうか設定します。"`true`" なら有効、"`false`" なら無効です。デフォルトは "`false`" です。

</dl>


#### TemplateBuilder のパラメータ {#templatebuilder}

Mayaa 標準の `TemplateBuilder` に対するパラメータをいくつか説明します。デフォルト設定は下記の通りです。

この `templateBuilder` は [id属性を無視する](./equals_id_resolver/)でパラメータ以外にも設定を紹介しています。

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE provider
    PUBLIC "-//The Seasar Foundation//DTD Mayaa Provider 1.0//EN"
    "http://mayaa.seasar.org/dtd/mayaa-provider_1_0.dtd">
<provider>
    <templateBuilder>
        <parameter name="outputTemplateWhitespace" value="true"/>
        <parameter name="outputMayaaWhitespace" value="false"/>
        <parameter name="optimize" value="true"/>
        <parameter name="defaultCharset" value="UTF-8"/>
                <!-- 1.1.22 で追加されたパラメータ -->
        <parameter name="replaceSSIInclude" value="true"/>
                <!-- 1.1.25 で追加されたパラメータ -->
    </templateBuilder>
</provider>
```


`outputTemplateWhitespace`
: HTML ファイル上の空白文字(スペース、タブ文字)を出力するかどうか設定します。"`true`" なら出力し、"`false`" なら出力しません。デフォルトは "`true`" です。

`outputMayaaWhitespace`
: Mayaa ファイル上の空白文字(スペース、タブ文字)を出力するかどうか設定します。"`true`" なら出力し、"`false`" なら出力しません。デフォルトは "`false`" です。

`optimize`
: テンプレートビルド後の最適化をするかどうか設定します。最適化をすると描画パフォーマンスが向上します。"`true`" なら有効、"`false`" なら無効です。デフォルトは "`true`" です。

`defaultCharset`
: HTML ファイルを読み込むときのデフォルト文字コードを指定します。デフォルトは "`UTF-8`" です。ここで指定した文字コードで HTML ファイルの先頭部分を読み込み、meta タグによる charset 指定を見つけたときは、meta タグで指定された文字コードで読み込み直します。

`replaceSSIInclude`
: <a href="/docs/settings/include/">SSI の include 記述を insert プロセッサに置き換える機能</a>を有効にするかを指定します。デフォルトは "`false`" です。
