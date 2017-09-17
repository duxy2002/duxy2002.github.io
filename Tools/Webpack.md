# Webpack
官网上中文翻译：https://doc.webpack-china.org/plugins/html-webpack-plugin/
重要参考URL:https://github.com/webpack-contrib/awesome-webpack

## Webpack是什么？

 把CSS，JavaScript，图像等等的构成Web的文件以Module为单位组织起来，而后变换成bundle.js一个文件的工具。
 **注意，不推荐全局安装 webpack。这会锁定 webpack 到指定版本，并且在使用不同的 webpack 版本的项目中可能会导致构建失败。**
 
 Webpackの要点をまとめてみると、
 * 主にJavaScriptをビルドするために使う
 * entryでコンパイル元を指定して、outputで出力先を指定する
 * Loaderという仕組みがあり、それでES6やReactをコンパイルできる
 * Pluginという仕組みがあり、それでUglifyなど圧縮処理などができる
 * WebpackDevServerという便利な開発ツールがある（次回以降のブログで扱いたい）
 * 大規模プロジェクトだとちょっと動作が重たい場合がある（→キャッシュオプションを活用する）

Webpackの処理の流れとしては、
1. entryに指定されたファイルを読み込む
2. 読み込む際にLoaderを通して変換する（例：BabelLoaderでES6をES5に変換する）
3. 処理の前後でプラグインが実行される（実行タイミングなどはプラグインごとに違う）
4. output先にビルド結果を出力する
つまり、entry〜outputまでの間に色々と処理を行うための仕組みとして、LoaderとPluginが用意されています。

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

##监视模式
webpack 的 watch mode 会监视文件的更改。如果检测到任何的更改，它都会再次执行编译。

我们也希望在编译时有一个好看的进度条。运行以下命令：
```
webpack --progress --watch
```
在你的文件中做一点更改并且保存。你应该会看到 webpack 正在重新编译。

## webpack.config.js
Webpack的配置文件。
主要的配置参数说明如下：
* entry: 页面入口文件配置，可以使一个或者多个入口文件；
A filename or a set of named filenames which act as the entry point to build your project. 
You can pass multiple entries (every entry is loaded on startup). 
If you pass a pair in the form <name>=<request> you can create an additional entry point. 
It will be mapped to the configuration option entry.

* output：对应输出项配置，即输出入口文件编译后的文件；
A path and filename for the bundled file to be saved in. It will be mapped to the configuration options output.path and output.filename.
```
//webpack.config.js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

* resolve：定义了解析模块路径时的配置，常用的是extensions，可以用来指定模块的后缀，这样在引入模块时就不需要些后缀，它会自动补全。

* module.loaders: 它是最关键的配置项，它告知Webpack每一类文件都需要使用什么加载器来处理。

    * Loaders  
    The goal is to have all of the assets in your project to be webpack's concern and not the browser's. 
    (This doesn't mean that they all have to be bundled together). webpack treats every file (.css, .html, .scss, .jpg, etc.) as a module. 
    However, webpack **only understands JavaScript**.  
    **Loaders in webpack transform these files into modules as they are added to your dependency graph.**  
    At a high level, they have two purposes in your webpack config.  
        1.  Identify what files should be transformed by a certain loader. (test property)
        2. Transform that file so that it can be added to your dependency graph (and eventually your bundle). (use property)
            ```
            //webpack.config.js
            const path = require('path');
            
            const config = {
              entry: './path/to/my/entry/file.js',
              output: {
                path: path.resolve(__dirname, 'dist'),
                filename: 'my-first-webpack.bundle.js'
              },
              module: {
                rules: [
                  {test: /\.(js|jsx)$/, use: 'babel-loader'}
                ]
              }
            };
            
            module.exports = config;
            ```
            
            The configuration above has defined a rules property for a single module with two required properties: test and use. This tells webpack's compiler the following:  
            >"Hey webpack compiler, when you come across a path that resolves to a '.js' or '.jsx' file inside of a require()/import statement, use the babel-loader to transform it before you add it to the bundle".

*Plugins
插件是 wepback 的支柱功能。在你使用 webpack 配置时，webpack 自身也构建于同样的插件系统上！

插件目的在于解决 loader 无法实现的其他事。
Since Loaders only execute transforms on a per-file basis, plugins are most commonly used (but not limited to) performing actions and custom functionality on "compilations" or "chunks" of your bundled modules (and so much more). The webpack Plugin system is extremely powerful and customizable.

In order to use a plugin, you just need to require() it and add it to the plugins array. Most plugins are customizable via options. Since you can use a plugin multiple times in a config for different purposes, you need to create an instance of it by calling it with new.
```
//webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      {test: /\.(js|jsx)$/, use: 'babel-loader'}
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```
plugin的List在[Plugins](https://webpack.js.org/plugins/)
## Common Options

1. Build source using a config file  
Specifies a different configuration file to pick up. Use this if you want to specify something different than webpack.config.js, which is the default.
webpack --config example.config.js


2. Specify build environment  
Send environment variable to be used in webpack config file.
```
webpack --env.production    # sets production to true
webpack --env.platform=web  # sets platform to "web"
```

## ソースマップを表示する

我们推荐你在生产环境中使用 source map，因为 Source Maps 对于 debug 和运行基准测试(benchmark tests)非常有用。webpack 可以在 bundle 中生成内联的 source map 或生成到独立文件。

在你的配置中，使用 devtool 对象来设置 Source Maps 的类型。我们现在支持七种类型的 source map。你可以在我们的 配置 文档页面找到更多相关的信息（cheap-module-source-map 是其中一种基本选项，对每行使用单独映射）

Webpackを用いることで簡単にソースマップも作成することができます。SourceMapを用いることで、React→ES5へ変換されたコードについて不具合があった場合に、ブラウザのDevツールで変換前のReactコードで不具合箇所を確認することができます。Webpackでソースマップを出力するにはdevtoolを設定します。
```typescript
// webpack.config.js
module.exports = {
    entry : './react-app.js',
    output : {
        filename : 'react-app.bundle.js'
    },
    module : {
        loaders : [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: /node_modules/,
                query: {
                  presets: ['react', 'es2015']
                }
            }
        ]        
    },
    //***************
    // devtoolで'source-map'を指定する
    //***************
    devtool : 'source-map'
};
```

## Plugins列表
* [write-file-webpack-plugin](https://github.com/gajus/write-file-webpack-plugin)
Forces webpack-dev-server to write bundle files to the file system.
    * Usage  
    Configure webpack.config.js to use the write-file-webpack-plugin plugin.
    ```
    import path from 'path';
    import WriteFilePlugin from 'write-file-webpack-plugin';
    
    export default {
        output: {
            path: path.join(__dirname, './dist')
        },
        plugins: [
            new WriteFilePlugin()
        ],
        // ...
    }
    ```
* webpack-dev-server

webpack-dev-server 为你提供了一个服务器和实时重载（live reloading） 功能。这很容易设置。

在开始前，确定你有一个 index.html 文件指向你的 bundle。假设 output.filename 是 bunlde.js。
```
<script src="/bundle.js"></script>
```
首先从 npm 安装 webpack-dev-server：
```
npm install --save-dev webpack-dev-server
```
安装完成之后，你应该可以使用 webpack-dev-server 了，方式如下：
```
webpack-dev-server --open
```

通常の webpack コマンドも --watch （または -w）オプションつきで実行することにより
ファイルの変更を検知して自動でリビルドを行うことが可能ですが、
webpack-dev-server は上記に加えて

    * ローカルサーバーも起動してくれる（中身は Node.js の Express サーバーらしい）
    * ファイルの変更を検知して自動ビルドした後、ブラウザも自動的にリロードしてくれる（Automatic Refresh）
    * ブラウザ全体のリロードではなく、編集したモジュールのみを更新する Hot Module Replacement という仕組みが使える（後述）
といった機能を備えているため、ローカルで開発する分にはこちらを使う方が便利です。



webpack-dev-server も webpack を実行した時と同じく webpack.config.js の内容を読み込み、ビルドを行います。
同時に、ローカルサーバーを起動し（デフォルトで）http://localhost:8080 または http://localhost:8080/webpack-dev-server/ からアクセス可能になります。

webpack-dev-server --hotによりmodule.hotがtrueにされますが、これは開発モード（development）でのみ読み込まれます。
本番モード（production）ではmodule.hotの値はfalseになるため、最終的なバンドルファイルからは取り除かれます。
运行webpack -p (也可以运行 webpack --optimize-minimize --define process.env.NODE_ENV="'production'"，他们是等效的)。它会执行如下步骤：
使用 UglifyJsPlugin 进行 JS 文件压缩
运行LoaderOptionsPlugin，查看其文档
设置 NodeJS 环境变量，触发某些 package 包，以不同的方式进行编译。

* ExtractTextWebpackPlugin
webpack 能够用 ExtractTextWebpackPlugin 帮助你将 CSS 单独打包。

# Tree Shaking
Tree shaking 是一个术语，通常用来描述移除 JavaScript 上下文中无用代码这个过程，或者更准确的说是按需引用代码，它依赖于 ES2015 模块系统中 import/export 的静态结构特性。这个术语和概念实际上是兴起于 ES2015 模块打包工具 rollup。

## QA
1. chunk代表什么含义
2. webpack.config.ts文件里面module.rules下面可以有loaders？
3. 是不是把Idea的safe write去掉以后，我们的修改就能实时反映到chrome中
