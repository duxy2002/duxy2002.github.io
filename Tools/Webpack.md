# Webpack

## Webpack是什么？

 把CSS，JavaScript，图像等等的构成Web的文件以Module为单位组织起来，而后变换成bundle.js一个文件的工具。
 
 例えば、１つのJavaScriptファイルに長い処理を書くと、可読性が悪くなり、バグが生まれやすくなったり、結果的に開発スピードが遅くなったりする原因になります。
 これを解決するために、「モジュール」という単位でJavaScriptを複数ファイルに分割して開発することがあります。
 こうすれば、エラーが起こったり仕様変更をした場合に、どのモジュールを修正すればいいのかが分かり、開発スピードが早くなります。
 
 しかし、JavaScriptの現行の仕様では、モジュールを扱うための仕組みが整っていないため、これまでにモジュールを取り扱うためにさまざまな仕様が検討されてきました。
 Webpackを使えば、JavaScriptモジュールをブラウザで扱える形に簡単に変換することができます。
 
 さらにユニークなのは、Webpackでは、JavaScriptだけでなくCSS、画像、Webフォント、音声、動画など、あらゆるアセットをモジュールとして扱い、１つの「バンドル」として出力することができる点です。

Webpackコマンドを使ってビルドすることで、ビルドされた「バンドル」の中にさまざまなファイルが「コード」として埋め込まれます。
これにより、ファイルのリクエスト数が減少し、ページ読み込み速度の改善が見込めるのです。

## Webpackを使うための準備
Webpackを使うには、npm経由でダウンロード・インストールを行います。
Macではターミナル、Windowsならコマンドプロンプトに下記のコマンドを入力するだけでOＫです。
```
npm install -g webpack 
```

もしMacでインストール中にエラーが出る場合は、コマンド冒頭に「sudo」と指定して管理者権限で実行してください。

インストールが完了したら、コンテンツのファイル一式を保存するためのフォルダーを任意の場所に作成し、コマンドでその場所に移動してください。
 移動後、下記コマンドを実行してnpmを初期化します。

```
npm init 
```

すると、プロジェクトの設定情報が記述された「package.json」というファイルが生成されます。
Webpackを実行するために、Webpack本体をプロジェクトフォルダにもインストールします。
```
npm install --save-dev webpack 
```
これでWebpackを使用するための準備が整いました。


#webpack.config.js
Webpack的配置文件。
主要的配置参数说明如下：
* entry: 页面入口文件配置，可以使一个或者多个入口文件；
* output：对应输出项配置，即输出入口文件编译后的文件；
* resolve：定义了解析模块路径时的配置，常用的是extensions，可以用来指定模块的后缀，这样在引入模块时就不需要些后缀，它会自动补全。
* module.loaders: 它是最关键的配置项，它告知Webpack每一类文件都需要使用什么加载器来处理。