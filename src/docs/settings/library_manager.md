---
layout: base
title: タグライブラリ(JSP, Mayaa)
eleventyNavigation:
  key: タグライブラリ(JSP, Mayaa)
  subtitle: LibraryManager
  parent: エンジンの設定
  order: 9
---
##  タグライブラリ(JSP, 独自Mayaaプロセッサ)

`LibraryManager`クラスはJSPタグライブラリをMayaa内から使えるようにしたり、独自定義したMayaaプロセッサを利用できるように管理します。
`LibraryManager`はスキャナークラスを用いて指定したディレクトリやファイルからタグライブラリのファイルを検索して登録します。

### スキャナー

JSPタグライブラリは、*.tld、Mayaaプロセッサは *.mld の拡張子をもつファイルをLibraryManagerがロードされた時にスキャナークラスを用いて検索します。

下表のパッケージ名の`~`は`org.seasar.mayaa.`を表します。
|スキャナクラス名|説明|
|-------------------------------------------|---------|
|[~impl.builder.library.scanner.WebInfSourceScanner](#webInfSourceScanner)| (1.3.0以降) WEB-INF内の *.tld,mld および WEB-INF/lib のJARファイル内のMETA-INF配下の *.tld,mldを検索する。このスキャナでFolderSourceScannerおよびMetaInfSourceScannerの両方を代替できる。 |
|[~impl.builder.library.scanner.FolderSourceScanner](#folderSourceScanner)| アプリケーションのコンテキストルート配下またはファイルシステム内に配置されたファイルを検索する。 |
|[~impl.builder.library.scanner.MetaInfSourceScanner](#metaInfSourceScanner)| JARファイル内のMETA-INFフォルダに配置されたファイルを検索する。 |
|[~impl.builder.library.scanner.ResourceScanner](#resourceScanner)| クラスパス内のリソースから条件に合致したファイルを検索する。 |
|[~impl.builder.library.scanner.DefaultSourceScanner](#defaultSourceScanner)| Mayaaのビルトインのプロセッサを定義している mayaa.mld を読み込む。 |
|[~impl.builder.library.scanner.WebXMLTaglibSourceScanner](#webXMLTaglibSourceScanner)| web.xml内のtaglibエレメントにて定義されたライブラリを読み込む。 |


#### WebInfSourceScanner {#webInfSourceScanner}

Webアプリケーションのコンテキスト内のリソースを走査してライブラリファイルを検出するための`SourceScanner`実装です。
走査対象は WEB-INF配下のリソースで、lib内のJARファイルも走査対象となります。

ServiceProviderの設定ファイルで以下のように設定することで、走査対象のライブラリファイルを指定することができます。
設定は記述順で評価され、最初に合致したものが採用されます。

`include`と`exclude`は `/WEB-INF/`を基準としてライブラリファイル(tld,mld)を検出対象(include)または除外対象(exclude)するためのGlobパターンを指定します。
記述順に評価され最初に合致したもので判定されます。
どちらのパターンも指定されない場合は`include="**/*.{tld,mld}"`のみが指定されたものとみなします。

`includeJar`と`excludeJar`で走査対象JARファイルのWEB-INF/libを基準としたファイル名部分のGlobパターンを指定します。
どちらのパターンも指定されない場合は `includeJar="*.jar"` のみが指定されたものとみなします。


`includeInJarMetaInf`と`excludeInJarMetaInf`で読み込み対象とするライブラリファイルパスのJARファイル内のMETA-INFを基準としたGlobパターンを指定します。
どちらのパターンも指定されない場合は `includeInJarMetaInf="*.{tld,mld}"` のみが指定されたものとみなします。

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<provider>
    <libraryManager>
       :
        <scanner class="org.seasar.mayaa.impl.builder.library.scanner.WebInfSourceScanner">
            <parameter name="exclude" value="{classes,lib}/"/>
            <parameter name="includeJar" value="taglibs-*.jar"/>
            <parameter name="excludeInJarMetaInf" value="{x,sql,scriptfree,fn,permittedTaglibs}*"/>
            <parameter name="includeInJarMetaInf" value="*.tld"/>
        </scanner>
```

|パラメータ名{style="width:8rem"}|必須{style="width:4rem"}|規定値{style="width:8rem"}|値の例{style="width:8rem"}|説明|
|-------|-----|--|--------------------------|---------|
|include    | - |**/*.\{tld,mld\}||読み込み対象とするライブラリファイルパスのWEB-INFを基準としたGlobパターン |
|exclude    | - | - |\{classes,lib\}|読み込み対象外とするライブラリファイルパスのWEB-INFを基準としたGlobパターン |
|includeJar | - |*.jar|taglibs-*.jar|走査対象JARファイルのWEB-INF/libを基準としたファイル名部分のGlobパターン |
|excludeJar | - | - ||走査対象外のJARファイルのWEB-INF/libを基準としたファイル名部分のGlobパターン |
|includeInJarMetaInf | - |*.\{tld,mld\}||読み込み対象とするライブラリファイルパスのJARファイル内のMETA-INFを基準としたGlobパターン |
|excludeInJarMetaInf | - | - |\{x,sql,fn\}-*.jar|読み込み対象外とするライブラリファイルパスのJARファイル内のMETA-INFを基準としたGlobパターン |

#### 　FolderSourceScanner {#folderSourceScanner}
アプリケーションのコンテキストルート配下またはファイルシステム内に配置されたファイルを検索する。

|パラメータ名{style="width:8rem"}|必須{style="width:4rem"}|規定値{style="width:5rem"}|値の例{style="width:8rem"}|説明|
|-------|-----|--|--------------------------|---------|
|folder| ○ |　- | /WEB-INF | 検索対象のフォルダ名を指定する。検索されるファイルのsystemIDはfolderに指定したパスからの相対パスが設定される。 |
|extension| - | - | .tld | 検索するファイル名の拡張子を指定する。extensionパラメータを複数回記述することで対象の拡張子を複数指定可能。|
|absolute| - | false | false | `true`にするとOSのファイルシステムのルートからの絶対パスと解釈する。falseの場合はWebアプリケーションのコンテキストルートを起点としたパスと解釈される。|
|recursive| - |false| true | `true`にすると指定したフォルダは以下のサブフォルダ配下も再帰的に検索する。|

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<provider>
    <libraryManager>
       :
        <scanner class="org.seasar.mayaa.impl.builder.library.scanner.FolderSourceScanner">
            <parameter name="folder" value="/WEB-INF"/>
            <parameter name="recursive" value="true"/>
            <parameter name="extension" value=".tld"/>
            <parameter name="extension" value=".mld"/>
        </scanner>
```

#### MetaInfSourceScanner {#metaInfSourceScanner}
JARファイル内のMETA-INFフォルダに配置されたファイルを検索する。

|パラメータ名{style="width:8rem"}|必須{style="width:4rem"}|規定値{style="width:5rem"}|値の例{style="width:8rem"}|説明|
|-------|-----|--|--------------------------|---------|
|folder| ○ |　- | /WEB-INF/lib | JARファイルの検索対象のフォルダ名を指定する。 |
|extension| - |  | .jar | JARファイルの拡張子を指定する。|
|absolute| - | false | false | `true`にするとOSのファイルシステムのルートからの絶対パスと解釈する。falseの場合はWebアプリケーションのコンテキストルートを起点としたパスと解釈される。|
|recursive| - |false| true | `true`にすると指定したフォルダは以下のサブフォルダ配下も再帰的に検索する。|
|extension| - | - | .jar | 検索するファイル名の拡張子を指定する。extensionパラメータを複数回記述することで対象の拡張子を複数指定可能。|
|ignore| - |　- | commons-beanutils- | (バグ動作しない)folderおよびexensionで指定した条件に合致するファイルのうち、検査対象から除外するファイル名のパターンを前方一致で指定する。|
|jar.folder| - |　META-INF/ | META-INF/ | 検索対象のフォルダ名を指定する。検索されるファイルのsystemIDはfolderに指定したパスからの相対パスが設定される。 |
|jar.extension| - | .tld<br>.mld | .tld | 検索するファイル名の拡張子を指定する。extensionパラメータを複数回記述することで対象の拡張子を複数指定可能。|
|jar.ignore| - | META-INF/MANIFEST.MF | - | 検索の起点となるフォルダを指定する。検索されるファイルのsystemIDはrootに指定したパスからの相対パスが設定される。 |

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<provider>
    <libraryManager>
       :
        <scanner class="org.seasar.mayaa.impl.builder.library.scanner.MetaInfSourceScanner">
            <parameter name="folder" value="/WEB-INF/lib"/>
            <parameter name="extension" value=".jar"/>
            <parameter name="ignore" value="commons-beanutils-"/>
            <parameter name="ignore" value="commons-collections-"/>
            <parameter name="ignore" value="commons-logging-"/>
            <parameter name="ignore" value="nekohtml-"/>
            <parameter name="ignore" value="jaxen-"/>
            <parameter name="ignore" value="xml-apis-"/>
            <parameter name="ignore" value="xercesImpl-"/>
            <parameter name="ignore" value="rhino-"/>
            <parameter name="jar.ignore" value="META-INF/MANIFEST.MF"/>
            <parameter name="jar.extension" value=".mld"/>
            <parameter name="jar.extension" value=".tld"/>
        </scanner>
```


#### ResourceScanner {#resourceScanner}

クラスパスに指定されたディレクトリやJARファイル内からライブラリファイルを検索します。
Webアプリケーションサーバ内では通常はサーバ起動時のJVMへの指定クラスパスとなっているため多くの場合は
このスキャナは指定する必要はありません。

|パラメータ名{style="width:8rem"}|必須{style="width:4rem"}|規定値{style="width:5rem"}|値の例{style="width:14rem"}|説明|
|-------|-----|--|--------------------------|---------|
|root| - |　- | META-INF/ | クラスパスがディレクトリだった時に走査するサブディレクトリを指定する。 |
|ignore| - | - | - | クラスパスがJarファイルのとき、Jar内でライブラリとして読み込まないエントリをルートから前方一致で指定。|
|extension| - | - | .tld | 検査対象となった（Jar）ファイル内でライブラリとして認識するエントリのファイル拡張子。複数可能。|
|includeJar| - | - | taglibs-standard-impl-*.jar | 走査対象とするJarファイル名Globパターン。|
|excludeJar| - | - | - | 走査除外とするJarファイル名のGlobパターン。|

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<provider>
    <libraryManager>
       :
        <scanner class="org.seasar.mayaa.impl.builder.library.scanner.ResourceScanner">
            <parameter name="root" value="META-INF/"/>
            <parameter name="ignore" value="META-INF/MANIFEST.MF"/>
            <parameter name="extension" value=".mld"/>
            <parameter name="extension" value=".tld"/>
        </scanner>
```

#### DefaultSourceScanner {#defaultSourceScanner}
Mayaaのビルトインのプロセッサを定義している mayaa.mld を読み込む。パラメータはありません。

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<provider>
    <libraryManager>
       :
        <scanner class="org.seasar.mayaa.impl.builder.library.scanner.DefaultSourceScanner"/>
```

#### WebXMLTaglibSourceScanner {#webXMLTaglibSourceScanner}
web.xml内のtaglibエレメントにて定義されたライブラリを読み込みます。パラメータはありません。

```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
<provider>
    <libraryManager>
       :
        <scanner class="org.seasar.mayaa.impl.builder.library.scanner.WebXMLTaglibSourceScanner"/>
```

### ビルダー

検出したTLDファイルやMLDファイルを読み込んで内部のタグ定義として格納するものです。
通常は変更する必要はありません(変更しないでください)。

|ビルダクラス名|
|-------------------------------------------|
|~impl.builder.library.MLDDefinitionBuilder|
|~impl.builder.library.TLDDefinitionBuilder|

### コンバーター

MLDファイルでクラス変換を行う際に使用するカスタムコンバータを登録します。
通常は変更する必要はありません(変更しないでください)。

|コンバータ名|コンバータクラス名|
|-----------------|-------------------------------------------|
|ProcessorProperty|~impl.builder.library.converter.ProcessorPropertyConverter|
|PrefixAwareName|~impl.builder.library.converter.PrefixAwareNameConverter|


### LibraryMangerの設定の全体感
```xml {data-filename=org.seasar.mayaa.provider.ServiceProvider}
※レイアウトの都合で改行しています。
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE provider
    PUBLIC "-//The Seasar Foundation//DTD Mayaa Provider 1.0//EN"
    "http://mayaa.seasar.org/dtd/mayaa-provider_1_0.dtd">
<provider>
   <libraryManager class="org.seasar.mayaa.impl.builder.library.LibraryManagerImpl">
        <converter name="ProcessorProperty" class="org.seasar.mayaa.impl.builder.library.converter.ProcessorPropertyConverter"/>
        <converter name="PrefixAwareName" class="org.seasar.mayaa.impl.builder.library.converter.PrefixAwareNameConverter"/>

        <scanner class="org.seasar.mayaa.impl.builder.library.scanner.WebInfSourceScanner">
            <!-- <parameter name="include" value="**/*.{tld,mld}"/> -->
            <parameter name="excludeJar" value="commons-beanutils,commons-collections,commons-logging,nekohtml,jaxen,xml-apis,xercesImpl,rhino}-*.jar"/>
            <parameter name="includeJar" value="*.jar"/>
            <!-- <parameter name="includeInJarMetaInf" value="*.tld"/> -->
        </scanner>

        <scanner class="org.seasar.mayaa.impl.builder.library.scanner.DefaultSourceScanner"/>

        <!-- after scan jars -->
        <!-- <scanner class="org.seasar.mayaa.impl.builder.library.scanner.WebXMLTaglibSourceScanner"/> -->

        <builder class="org.seasar.mayaa.impl.builder.library.MLDDefinitionBuilder"/>
        <builder class="org.seasar.mayaa.impl.builder.library.TLDDefinitionBuilder"/>
    </libraryManager>
</provider>
```

