# Spring MVC(+Spring Boot)上での静的リソースへのアクセスを理解する

今回は、Spring MVC上で静的リソース（HTML、JavaScript、CSS、画像など）にアクセスする方法について説明します。ここでは、「Java EEのアプリケーションサーバーの機能を使用してアクセスする方法」と「Spring MVCの独自機能を使用してアクセスする方法」について説明します。
また、最後にSpring Bootアプリでアクセスする方法についても紹介します。

## 動作検証バージョン
* Spring Framework 4.2.5.RELEASE
* Spring Boot 1.3.3.RELEASE


## アプリケーションサーバーの機能を使用した静的リソースへのアクセス

まずは、Java EE準拠のWebアプリケーションでどのように静的リソースを扱うか説明（おさらい）しておきましょう。
Java EE準拠のWebアプリケーションでは、静的リソースはドキュメントルート上の任意のディレクトリに格納します。 
ドキュメントルートは、MavenやGradleプロジェクトではあればsrc/main/webappになります。

* Webアプリケーションのドキュメントルート上への格納例

Project Root
└─src
    └─ main
        └─ webapp  # Webアプリケーションのドキュメントルート
            └─ static
                └─ css
                    └─ app.css
例えば、上記のようなディレクトリにCSSファイルを格納した場合は、
http://localhost:8080/(context-path/)static/css/app.cssというパスでアクセスすることができます。
 使用するアプリケーションサーバーがTomcatの場合は、org.apache.catalina.servlets.DefaultServletというサーブレットを介して
 Webアプリケーション内の静的リソースにアクセスします。
 Tomcat以外の主要なアプリケーションサーバーでも類似のサーブレットが用意されており、
 これらのサーブレットのことを「デフォルトサーブレット」と呼びます。
なお、「デフォルトサーブレット」はルートパス(「/」)にマッピングされています。

> Note: Servlet 3.0+ でのドキュメントルート
> Servlet 3.0より、warファイル内のjarファイルの中にある「META-INF/resources」ディレクトリもWebアプリケーションのドキュメントルートとして扱われるようになっています。
> 
> 
> warファイル
> └─ WEB-INF
>     └─ lib
>         └─ foo.jar
>             └─ META-INF
>                 └─ resources # Webアプリケーションのドキュメントルート
>                     └─ static
>                         └─ css
>                             └─ foo.css
> 
> 
> 例えば、上記のようなディレクトリにCSSファイルを格納した場合は、http://localhost:8080/(context-path/)static/css/foo.cssというURLでアクセスすることができます。

## Spring MVCのDispatcherServletとデフォルトサーブレットの関係

Spring MVCでは、org.springframework.web.servlet.DispatcherServletで全てのリクエストを受け、
リクエスト内容に対応するHandler(ControllerのHandlerメソッド)にディスパッチする仕組みになっており、
多くのSpring MVCアプリケーションでDispatcherServletをルートパスにマッピングしています。


```src/main/webapp/WEB-INF/web.xml
 
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- ... -->
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern> <!-- ルートパスにマッピング -->
</servlet-mapping>
```

DispatcherServletをルートパスにマッピングすると、アプリケーションサーバーが提供している「デフォルトサーブレット」が呼び出されなくなり、
その結果として、Webアプリケーションのドキュメントルート上から静的リソースを取得できなくなります。

DispatcherServletとデフォルトサーブレットの併用

DispatcherServletをルートパスにマッピングしつつ、デフォルトサーブレット経由でドキュメントルート上の静的リソースにアクセスすることはできないのでしょうか？
 （もちろん）できます！！ DispatcherServletとデフォルトサーブレットを併用したい場合は、DispatcherServletで受けたリクエストをデフォルトサーブレットにフォワードする機能(org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler)を有効にしてください。

```java
@Configuration
@EnableWebMvc // 注意：Spring Bootの場合は、@EnableWebMvcはつけちゃダメ！！
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    // ...
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```
DispatcherServletで受けたリクエストをデフォルトサーブレットにフォワードする機能を有効にすると、以下のような流れで静的リソースにアクセスすることができます。

> Warning: Spring BootでWebMvcConfigurerAdapterを利用する際の注意点
> 
> Spring BootでWebMvcConfigurerAdapterの子クラスを作成する場合は、
> @EnableWebMvcは絶対につけないでください。@EnableWebMvcをつけてしまうと、
> Spring BootのAutoConfigureのコンフィギュレーションが一部無効になってしまいます。これはSpring Bootのリファレンスにも記述されています。

## Spring MVCの独自機能を利用した静的リソースへのアクセス

ここまではアプリケーションサーバーの機能を使用して静的リソースへアクセスする方法を紹介しましたが、
ここからは、Spring MVCが独自に提供する機能を使用して静的リソースへアクセスする方法を紹介していきます。

Spring MVCは、静的リソースにアクセスするためのHandlerとしてorg.springframework.web.servlet.resource.ResourceHttpRequestHandlerというクラスを
提供しています。
ResourceHttpRequestHandlerを利用すると、
* DispatcherServletを経由での静的リソースへのアクセス
*リソース毎(リソースのパターン毎)のキャッシュ制御
* クラスパスや任意のディレクトリに格納されているファイルへのアクセス
* バージョン付きのパス経由での静的リソースへのアクセス
* Gzip化された静的リソースへのアクセス
* WebJar内の静的リソースへのアクセス

などを実現することができます。

以下は、クラスパス上のファイルを静的リソースとして扱う場合のBean定義例です。

```java
@Configuration
@EnableWebMvc // 注意：Spring Bootの場合は、@EnableWebMvcはつけちゃダメ！！
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    // ...
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/");
    }
}
```
```
Project Root
└─src
    └─ main
        └─ resources  # クラスパス
            └─ static
                └─ css
                    └─ app.css
```

例えば、上記のようなディレクトリにCSSファイルを格納した場合は、
http://localhost:8080/(context-path/)static/css/app.cssというパスでアクセスすることができ、
以下のようなレスポンスヘッダーが設定されます。

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Last-Modified: Wed, 04 May 2016 04:42:11 GMT
Accept-Ranges: bytes
Content-Type: text/css;charset=UTF-8
```

## キャッシュの有効期間の設定

デフォルトの動作では、キャッシュの有効期間は設定されないため、キャッシュに関する動作はブラウザの仕様に依存します。
キャッシュの有効期間を設定したい場合は、明示的に有効期間を指定してください。
有効期間を指定すると、Cache-Controlヘッダーのmax-age属性に値が設定されます。

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/static/**")
            .addResourceLocations("classpath:/static/")
　          .setCachePeriod(604800); // 有効期間(秒単位)の設定 604800=1週間
}
```

該当リソースにアクセスすると、以下のようなレスポンスヘッダーが設定されます。

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Last-Modified: Wed, 04 May 2016 04:42:11 GMT
Cache-Control: max-age=604800
Accept-Ranges: bytes
Content-Type: text/css;charset=UTF-8
...
```
キャッシュの有効期間を設定すると、有効期間内であればブラウザにキャッシュされているコンテンツが利用され、
有効期間を超えている場合は、サーバー(ResourceHttpRequestHandler)への問い合わせが行われます。
ResourceHttpRequestHandlerは、コンテンツが更新されていれば更新後のコンテンツを応答し、
更新されていなければ304(Not Modified)を応答します。


> Note: 有効期間に0を指定すると・・・
> 有効期間に0を設定すると、Cache-Controlにno-store属性が設定されます。
> 
> ```
> HTTP/1.1 200 OK
> Server: Apache-Coyote/1.1
> Last-Modified: Wed, 04 May 2016 04:42:11 GMT
> Cache-Control: no-store
> Accept-Ranges: bytes
> Content-Type: text/css;charset=UTF-8
> ```

Cache-Controlの設定

有効期間(max-age)以外のキャッシュ制御の設定値を指定することもできます。

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/static/**")
            .addResourceLocations("classpath:/static/")
            .setCacheControl(
                    CacheControl.maxAge(7, TimeUnit.DAYS)
                            .sMaxAge(14, TimeUnit.DAYS)
                            .cachePrivate()
                            .cachePublic()
                            .mustRevalidate()
                            .noTransform()
                            .proxyRevalidate()
            );
}
```

該当リソースにアクセスすると、以下のようなレスポンスヘッダーが設定されます。

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Last-Modified: Wed, 04 May 2016 04:42:11 GMT
Cache-Control: max-age=604800, must-revalidate, no-transform, public, private, proxy-revalidate, s-maxage=1209600
Accept-Ranges: bytes
Content-Type: text/css;charset=UTF-8
...
```

## 静的リソースのバージョニング

Spring MVCの機能を利用すると、静的リソースにバージョンを付与することができます。
Built-inでサポートされているバージョニング方法は、以下の2種類です。


### 1.コンテンツデータのMD5ハッシュ値によるバージョニング
この方法は、静的リソースのコンテンツデータ(バイト配列)をMD5のハッシュ値にした値をバージョン番号として扱い、
バージョン番号をファイル名に含めます。つまり、ファイルの中身がかわると別のバージョンとして認識されます。
たとえば、オリジナルのリソースパスが/static/css/app.cssの場合は、
/static/css/app-f5100c1673b440e00b7839d189c43636.cssという感じのリソースパスになります。

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/static/**")
            .addResourceLocations("classpath:/static/")
            .resourceChain(false) // ローカルの開発環境ではfalse(キャッシュなし)、試験環境やプロジェクション環境ではtrue(キャッシュあり)を指定する
            .addResolver(new VersionResourceResolver()
                     .addContentVersionStrategy("/**")); // コンテンツデータのMD5ハッシュ値によるバージョニング機能の有効化
}
```

### 2. 指定した固定バージョンによるバージョニング

この方法は、指定した固定値をバージョン番号として扱い、バージョン番号はディレクトリになります。たとえば、オリジナルのリソースパスが/static/css/app.cssの場合は、/static/1.0.0/css/app.cssという感じのリソースパスになります。

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/static/**")
            .addResourceLocations("classpath:/static/")
            .resourceChain(false)
            .addResolver(new VersionResourceResolver()
                    .addFixedVersionStrategy("1.0.0","/**")); // 指定した固定バージョンによるバージョニング機能の有効化
}
```
Viewとバージョニング機能の連携

バージョニングされたリソースパスをJSPなどのView内で扱うためには、サーブレットフィルターとしてorg.springframework.web.servlet.resource.ResourceUrlEncodingFilterを登録してください。
ResourceUrlEncodingFilterを使用すると、オリジナルのリソースパスからバージョニングされたリソースパスを解決することができます。

```html
src/main/webapp/WEB-INF/web.xml
 
<filter>
    <filter-name>ResourceUrlEncodingFilter</filter-name>
    <filter-class>org.springframework.web.servlet.resource.ResourceUrlEncodingFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>ResourceUrlEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
SpringBootの場合に、下記のようにする
```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
    @Bean
    public ResourceUrlEncodingFilter resourceUrlEncodingFilter() {
        return new ResourceUrlEncodingFilter();
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        VersionResourceResolver versionResolver = new VersionResourceResolver()
                .addContentVersionStrategy("/css/**", "/js/**");
        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:static/")
                .setCachePeriod(null)
                .resourceChain(true)
                .addResolver(versionResolver);
    }
}
```
ViewとしてJSPを使用する場合は、JSTLの<c:url>やSpringの<spring:url>を使用してURLを生成すると、バージョニングされたリソースパスを出力することができます。なお、ViewとしてThymeleaf、Freemarker、Velocityなどを使用する場合でも、JSPと同様の仕組みがサポートされています。

```html
<html>
<head>
    <link href="<c:url value='/static/css/app.css'/>" type="text/css" rel="stylesheet" />
</head>
<body>
<h2>Hello World!</h2>
</body>
</html>
```

上記のJSPから、以下のようなHTMLが出力されます。

```html
<html>
<head>
    <link href="/static/css/app-f5100c1673b440e00b7839d189c43636.css" type="text/css" rel="stylesheet" />
</head>
<body>
<h2>Hello World!</h2>
</body>
<div></div>
</html>
```

また、CSSファイル内に指定しているURL(別のCSSファイルをインポートするためのURLや、画像ファイルのURLなど)も、バージョニングされたリソースパスに変換されます。

```html
@import url(/static/css/fw.css);

body {
    background-image: url("/static/images/body-background.png");
}
```

上記のCSSにアクセスすると、クライアントにはバージョニングされたリソースパスに置き換わってレスポンスされます。

```html
@import url(/static/css/fw-01e21f21ded830ac657f4afbc17e6495.css);

body {
    background-image: url("/static/images/body-background-d41d8cd98f00b204e9800998ecf8427e.png");
}
```

## WebJarの利用

OSSのCSSライブラリやJavaScriptライブラリを使用する場合は、WebJarを使うこともあると思います。
ここでは、Spring MVCの機能を使ってBootstrapのWebJarを利用する方法を紹介します。

まず、BootstrapのWebJarを依存ライブラリに追加します。

```xml
pom.xml
 
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>3.3.6</version>
</dependency>
```

次に、WebJarから静的リソースを取得するための設定を行います。

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    // ...
    registry.addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/");
}
```

最後に、ViewからWebJar内のリソースにアクセスします。

```html
<html>
<head>
    <link href="<c:url value='/webjars/bootstrap/3.3.6/css/bootstrap.min.css'/>" type="text/css" rel="stylesheet" />
</head>
<body>
<h2>Hello World!</h2>
</body>
</html>
```

### ライブラリのバージョン番号の隠蔽
WebJarから静的リソースにアクセスすることはできましたが、
View内でライブラリのバージョン番号を意識してしまっています。
このままだとライブラリのバージョンアップした時にViewも修正する必要があるため、
メンテナンス性がよくありません。ここでは、View内でライブラリのバージョン番号を意識させない方法を紹介しましょう。

まず、依存ライブラリとしてorg.webjars:webjars-locatorを追加します。

```xml
pom.xml
 
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator</artifactId>
    <version>0.31</version>
</dependency>
```

次に、WebJarのバージョン番号を隠蔽する機能(org.springframework.web.servlet.resource.WebJarsResourceResolver)を有効化します。
なお、ResourceUrlEncodingFilterの登録も必要です。

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    // ...
    registry.addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/")
            .resourceChain(false); // 自動でWebJarsResourceResolverが有効化される
}
```

最後に、ViewからWebJar内のリソースにバージョンを省いてアクセスすると、

```html
<html>
<head>
    <link href="<c:url value='/webjars/bootstrap/css/bootstrap.min.css'/>" type="text/css" rel="stylesheet" />
</head>
<body>
<h2>Hello World!</h2>
</body>
</html>
```

バージョン付きのパスに変換されます。

```html
<html>
<head>
    <link href="/webjars/bootstrap/3.3.6/css/bootstrap.min.css" type="text/css" rel="stylesheet" />
</head>
<body>
<h2>Hello World!</h2>
</body>
</html>
```


## Gzip化された静的リソースへのアクセス
OSSのCSSライブラリやJavaScriptライブラリの中には、テキストベースのファイルとは別にGzip化したファイルを同封していることがあります。
静的リソースとしてGzip化されたファイルを使用すると、データの転送量を減らすことができるため、画面の表示速度が速くなることが期待できます。
Spring MVCでも、Gzip化されたファイルへアクセスする機能(org.springframework.web.servlet.resource.GzipResourceResolver)を提供しています。

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    // ...
    registry.addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/")
            .resourceChain(false)
            .addResolver(new GzipResourceResolver());
}
```

以下に、OSSのCSSライブラリのひとつであるBootstrapのWebJarを例に、GzipResourceResolverを適用する時の実装例を紹介します。

```
WebJar(bootstrap-3.3.6.jar)
└─ META-INF
    └─ resources
        └─ webjars
            └─ bootstrap
                └─ 3.3.6
                    ├─ css
                    |   ├─ bootstrap.min.css
                    |   ├─ bootstrap.min.css.gz # Gzip化されたファイル
                    ...
```

Viewの実装では、Gzipファイルを意識する必要はありません。

```html
<html>
<head>
    <link href="<c:url value='/webjars/bootstrap/css/bootstrap.min.css'/>" type="text/css" rel="stylesheet" />
</head>
<body>
<h2>Hello World!</h2>
</body>
</html>
```

上記のView(JSP)から生成されるHTMLは、以下のようになります。

```html
<html>
<head>
    <link href="/webjars/bootstrap/3.3.6/css/bootstrap.min.css" type="text/css" rel="stylesheet" />
</head>
<body>
<h2>Hello World!</h2>
</body>
</html>
```

HTMLだけ見るとGzip化されたファイルが使われているかわかりませんが、
HTTPレスポンスのヘッダー情報を見るとGzip化されたファイルが使われていることがわかります。
Content-Encodingヘッダー値がgzip、Content-Lengthヘッダー値がGzipファイルのサイズ(約20KB)になっています。

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Last-Modified: Thu, 05 May 2016 12:14:00 GMT
Content-Encoding: gzip
Accept-Ranges: bytes
Content-Type: text/css;charset=UTF-8
Content-Length: 19747
Date: Fri, 06 May 2016 03:25:27 GMT
```

なお、GzipResourceResolverを使用しない場合のHTTPレスポンスのヘッダー情報は、以下のようになります。
Content-Encodingヘッダーは出力されず、Content-Lengthヘッダー値がテキストファイルのサイズ(約120KB)になっています。

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Last-Modified: Thu, 05 May 2016 12:14:00 GMT
Accept-Ranges: bytes
Content-Type: text/css;charset=UTF-8
Content-Length: 121392
Date: Fri, 06 May 2016 03:33:13 GMT
```


## Spring Boot上での静的リソースへのアクセス
Spring Bootは、Spring MVCの機能を使って静的リソースにアクセスします。
Spring Bootを使用すると、これまで説明してきた設定をSpring Bootの自動コンフィギュレーションの仕組みを使って行うことができます。


静的リソースの格納先
静的リソースは、クラスパス上の「/static」「/public」「/resources」「/META-INF/resources」の配下に格納することができます。

```
Project Root
└─src
    └─ main
        └─ resources  # クラスパス
            ├─ static
            |   └─ css
            |       └─ a.css
            ├─ public
            |   └─ css
            |       └─ b.css
            ├─ resources
            |   └─ css
            |       └─ b.css
            └─ META-INF
                └─ resources
                    └─ css
                        └─ d.css
```


例えば上記のような構成でファイルを格納すると、それぞれ以下のURLでアクセスすることができます。
•http://localhost:8080/css/a.css
•http://localhost:8080/css/b.css
•http://localhost:8080/css/c.css
•http://localhost:8080/css/d.css

静的リソースの格納先を変更したい場合は、以下のような設定をプロパティファイルに指定してください。

```properties
src/main/resources/application.properties
 
spring.resources.static-locations=classpath:/static/
```


## キャッシュの有効期間の設定
キャッシュの有効期間を設定する場合は、以下の設定をプロパティファイルに指定してください。

```properties
src/main/resources/application.properties
 
spring.resources.cache-period=604800
```


## 静的リソースのバージョニング
Spring Bootでも静的リソースのバージョニングが可能です。


### コンテンツデータのMD5ハッシュ値によるバージョニング
コンテンツデータのMD5ハッシュ値によるバージョニングを利用する場合は、以下の設定をプロパティファイルに指定してください。

```properties
src/main/resources/application.properties
 
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```


### 指定した固定バージョンによるバージョニング
指定した固定バージョンによるバージョニングを利用する場合は、以下の設定をプロパティファイルに指定してください。

```properties
src/main/resources/application.properties
 
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.version=1.0.0
spring.resources.chain.strategy.fixed.paths=/**
```


### Viewとバージョニング機能の連携
ViewとしてThymeleafとVelocityを使う場合は、Spring Bootの自動コンフィギュレーションでResourceUrlEncodingFilterが登録されます。
また、Spring Boot 1.4からは、Freemarkerも自動コンフィギュレーションされます（Spring Boot 1.3までは自動で登録されません）。
自動コンフィギュレーション対象になっていないViewを使う場合は、ResourceUrlEncodingFilterのBeanを明示的に定義してください。

```java
@Bean
ResourceUrlEncodingFilter resourceUrlEncodingFilter() {
    return new ResourceUrlEncodingFilter();
}
```


## WebJarの利用
Spring Bootを使用すると、WebJar用のコンフィギュレーションを自動で行ってくれるため、
使用するライブラリのWebJarを追加するだけです。
また、バージョン番号を隠蔽したい場合は、依存ライブラリとして
org.webjars:webjars-locatorを追加してください（こちらもライブラリを追加するだけで自動で有効化されます :v: ）。
Viewでの実装は、「Spring MVCの独自機能を使用してアクセスする方法」で紹介した内容と一緒です。


## Gzip化された静的リソースへのアクセス
Spring BootでGzip化された静的リソースへアクセスする場合は、以下の設定をプロパティファイルに指定してください。

```
src/main/resources/application.properties
 
spring.resources.chain.gzipped=true
```


## リソースチェーンのキャッシュの無効化
バージョニング機能やGzip機能などを使う際に、デフォルトの動作だと読み込んだリソース情報をキャッシュします。
これは試験環境やプロダクション環境では有効な設定ですが、ローカルの開発環境では開発効率を低下させる可能性があります。
ローカルの開発環境では、キャッシュを無効化する方がようでしょう。
キャッシュを無効化する場合は、以下の設定をプロパティファイルに指定してください。

```
src/main/resources/application.properties
 
spring.resources.chain.cache=false
```


## 自動コンフィギュレーションの無効化
Spring Bootが行う自動コンフィギュレーションを無効化したい場合は、以下の設定をプロパティファイルに指定してください。

```properties
src/main/resources/application.properties
 
spring.resources.add-mappings=false

```



# まとめ
今回は、Spring MVCが提供する静的リソースがらみの機能を紹介しました。
Spring Bootでアプリケーションを作っているとあまり意識しない(しなくてもいい)部分かもしれませんが、
Spring MVCはこんな仕組みで静的リソースへアクセスしています。


# 補足

Gzip機能とバージョニング機能の併用について (2016/5/14 追加)
初回投稿時に、Gzip機能とバージョニング機能の併用の仕方＋Spring Bootで併用ができない旨を記載していましたが、
どうもSpring Framework的にはサポート外な使い方みたいなので、記載を削除しました（すみません・・・ :persevere: ）。
詳しくは 「[Spring Bootのgh-5876](https://github.com/spring-projects/spring-boot/issues/5876)」をご覧ください。




