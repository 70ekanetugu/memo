# webpack使い方
# 目次
- [概要](#概要)
- [基本フロー](#基本フロー)
- [webpack.config.js](#webpack.config.js)

# 概要
webpackは、javascriptのバンドルツールである。  
cssや画像などもjsファイルにバンドルできる。  
ローダーやプラグインにより、バンドルするまでの中間操作としてbabelでトランスパイルしたり、scssのコンパイルなどができる。
その為、ビルドツールとしての認識が一般的な気がする。


## コンセプト
### 1. エントリー
webpackモジュールにおけるエントリーポイント。  
webpackは、エントリーポイントが依存しているモジュール及びライブラリ特定を行う。  
デフォルトでは、./src/index.jsがエントリーポイントとして指定される。  

```javascript
//単一エントリー構文
entry: '.src/index.js'

//オブジェクト構文
//ex.以下はマルチページアプリ想定の場合
//ページ毎のjsファイルを生成している。
entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
}
```

### 2. 出力
webpackによるバンドル後のファイル出力先。  
デフォルトでは、./dist/main.jsに出力される。  

```javascript
//単一エントリーの場合(ファイル名は静的に記述できる)
output: {
    path: '/dist',
    filename: 'main.js'
}

//複数エントリーの場合(置換を利用する)
output: {
    path: '/dist',
    filename: '[name].main.js', //エントリ名に置換
    filename: '[id].main.js' //内部チャンク(各エントリ)IDに置換
    //他にもハッシュや関数等が利用可
}
```  

### 3. Loader
webpackは本来、複数のjs(json)ファイルを1つにまとめる事が仕事の為、js以外のファイル(css, 画像など)も纏める場合は、jsで扱える形にコンパイルする必要がある。
バンドル前の中間操作的な変換処理を行うものをLoaderと呼ぶ。  
- Loaderを使用するには、module.rules配列にLoader毎のオブジェクトを記述する。  
- ローダーは下から上へ評価/実行される。。ローダーによって実行順序があるので要確認。(↓の例だと、sass->css->style->rawの順)  

```javascript
module: {
    rules: [
        {
            test: /\.txt$/,
            use: 'raw-loader'
        },
        {
            use: 'style-loader'
        },
        {
            test: /\.css$/,
            use: 'css-loader'
        },
        {
            test: /\.scss$/,
            use: 'sass-loader'
        }
    ]
}
```   


主要なローダー配下の通り。
- babel-loader　Babelを使用する為必要
- sass-loader sassをコンパイルする
- css-loader cssの@importやurl()を解決して読み込む
- file-loader コンテンツ内容のハッシュ値計算し、保存する。
- style-loader styleタグをDOMに追加する処理。css-loaderと併用。
- raw-loader テキスト・バイナリファイルを読み込む
- ts-loader Typescriptをコンパイルする。
- url-loader コンテンツをBase64エンコードする。(css内のurl()の画像などをbase64にし、httpsリクエストが不要になる)    

 

### 4. プラグイン
Loaderが特定のタイプ(拡張子)モジュールの変換に使用されるのに対し、プラグインはバンドルの最適化、asset管理、環境変数の注入など様々なタスクを実行する為のものである。
プラグインを使用する為には、使用するプラグインモジュールをrequire()し、plugins配列に追加する。  

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');
//....中間省略
plugins: [
    new HtmlWebpackPlugin({
        template: './src/index.html'
    })
]
```  

### 5. モード
webpackの最適化モードを指定する。ここでいうモードは、process.env.NODE_ENVの値である。※つまり、デバッグ様にコードの中にprocess.env.NODE_ENVでif分岐を作ることもできる。  
デフォルトはproduction。  
- development 
  - `process.env.NODE_ENV`に`development`をセットする
  - NamedChunksPluginを有効にする
  - NameModulesPluginを有効にする
- production　
  - `process.env.NODE_ENV`に`production`をセットする
  - FlagDependencyUsagePluginを有効にする
  - FlagIncludedChunksPluginを有効にする
  - ModuleConcatenationPluginを有効にする
  - NoEmitOnErrorsPluginを有効にする
  - OccureenceOrderPluginを有効にする
  - SideEffectsFlagPluginを有効にする
  - TerserPluginを有効にする
- none
  - webpackのデフォルト最適化オプションを無効にする。

### 6. ブラウザ互換  
webpackはES5準処の全てのブラウザをサポート(IE8以下はサポート外)。  
古いブラウザで、`Promise`や`import()`をサポートする為には、ポリフィルをロードする必要がある。

### 7. 環境
webpackはNode.js`バージョン8.x以降`で実行可能。

# 基本フロー
1. `$npm install --save-dev webpack babel-core bable-loader babel-preset-env`
2. 「.babelrc」を作成し、`{"presets: ["env"]"}`を記述
3. 「webpack.config.js」を作成し、以下を記述
```javascript
/* なお、以下の記述はwebpackにおけるデフォルト設定の為、本ファイルが無くても同じ内容でバンドルされる。*/
module.exports = {
        // ビルドの基点となるファイル
        entry: './src/index.js',
        // ビルド後のファイル
        output: {
            path: __dirname + '/dist',
            filename: 'main.js'
        },
        // 拡張子が.jsのファイルはbabel-loaderを通してビルド(node_modulesは除外)
        module: {
            loaders: [{
                test: /\.js$/, //testプロパティにはLoaderに通すファイルを指定。拡張子の正規表現が良い。
                exclude: /node_modules/, //Loaderに通さない(除外する)ファイルを指定。node_modulesは除外
                use: 'babel-loader' //使用するLoaderを指定
            }]
        }
};
```
4. ソースファイルを作成
5. `$ ./node_modules/.bin/webpack`を実行し、ビルド
6. outputで指定したディレクトリにビルド後のファイルが出力されている。

# webpack.config.js

## Loader
webpackは本来、複数のjs(json)ファイルを1つにまとめる事が仕事の為、js以外のファイル(css, 画像など)も纏める場合は、jsで扱える形にコンパイルする必要がある。
バンドルまでの中間操作的な変換処理を行うものをLoaderと呼ぶ。  

- ファイル系  

|ローダー名|内容|
|:--|:--|
|raw-loader|ファイルを生のコンテンツでロードする(utf8)|
|val-loader|モジュールとしてコードを実行し、jsコードとしてエクスポートする|
|url-loader|file-loaderと似たような動きだが、ロードするファイルが制限より小さい時、データURL(Base64エンコード)を返す。|
|file-loader|ファイルを出力フォルダに出力し、相対URLを返す|
|ref-loader|ファイル間の依存関係を作成|  

- JSON系  

|ローダー名|内容|
|:--|:--|
|json5-loader|JSON5ファイルをロードし、トランスパイルする|
|cson-loader|CSONファイルをロードし、トランスパイルする|  

- コンパイル  

|ローダー名|内容|
|:--|:--|
|babel-loader|Babelを使用してES6以上のコードをES5にトランスパイルする|
|buble-loader|Bubleを使用してES6以上のコードをES5にトランスパイルする。|
|traceur-loader|Traceurを使用してES6以上のコードをES5にトランスパイルする。|
|ts-loader|Typescriptをロードする|
|coffee-loader|coffeescriptをロードする|
|fengari-loader|fengariを使用してLuaコードをロード|
|elm-webpack-loader|Elmをロード|  

- テンプレート  

|ローダー名|内容|  
|:--|:--|  
|html-loader|HTMLを文字列としてエクスポートする。|    
|pug-loader|Pug及びJadeテンプレ―トをロードし、関数を返す|  
|markdown-loader|マークダウンをHTMlにコンパイル|  
|react-markdown-loader|マークダウンをReactコンポーネントにコンパイル|  
|posthtml-loader|PostHTMLを使用して、HTMLファイルをロードして変換する|
|handlebars-loader|ハンドルバーをHTMLにコンパイル|  
|markup-inline-loader|HTMLにSVG/MathMLファイルをinline化する。SVGにCSSアニメーションを適用する時に便利|  
|twig-loader|Twigテンプレートをロードし関数を返す|  

- スタイル  

|ローダー名|内容|
|:--|:--|
|style-loader|DOMにstyleとしてモジュールのエクスポートを追加|
|css-loader|@importを解決したcssコードを返す|
|less-loader|LESSファイルをロードしてコンパイルする|
|sass-loader|SASS/SCSSファイルをロードしコンパイルする|
|postcss-loader|PostCSSを使用して、CSS/SSSファイルをロードする|
|stylus-loader|Stylusファイルをロードしてコンパイルする|  

- リンティング 及び テスト  

|ローダー名|内容|
|:--|:--|
|mocha-loader|mochaを使用したテスト| 
|eslint-loader|ESLintを使用してコードをリンティングする|

- フレームワーク    

|ローダー名|内容|
|:--|:--|
|vue-loader|vueコンポーネントをロードしてコンパイルする|  
|polymer-loader|?|
|angular2-template-loader|Angularコンポネントをロードしてコンパイルする|  

#### Loaderの指定
module.rulesにLoaderの指定を行う。主要なものは以下。
- test : Loaderに通すファイルを文字列または正規表現で指定。一般的に拡張子。
- use: どのLoaderを使用するか指定。
- use.loader: Loaderの指定
- use.options: Loaderのオプション指定(※非推奨の古い書き方はuse.query)
```javascript
//babel-loaderとraw-loaderの設定例
module.exports = {
    /* 略 */

    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: { //Loaderにオプション付ける場合は、useプロパティ内にloaderとoptionを記述。
                    loader: "babel-loader",
                    options: {
                        presets: ["@babel/preset-env"]
                    }
                }
            },
            {
                test: /\.txt$/,
                exclude: /node_modules/,
                use: [ 
                    {
                        loader: "raw-loader",
                        options: { ... }
                    }
                ]
            }
        ]
    }
}
```

## 全体構成
webpackの設定ファイル。webpackがデフォルトで読み込むファイル名が「webpack.config.js」であるが、独自の名前を付けることも出来る。  
基本構成は以下の通り。  
```
module.exports = {
    //バンドルモード指定。本番なら"production"、開発なら"development"
    mode: "development", 
    devtool: "source-map", //modeがdevelpmentの時、バンドル前後のファイルを紐づけるマップを生成する。
    entry: "./src/index.js", //エントリーポイントとなるjsファイルの相対パス
    output: {
        path: __dirname + "/dist", //出力先は絶対パスにしないといけない。
        filename: "bundle.js" //出力ファイル名
    },
    //↓他設定項目の記述
    ...
    ...
}
```  
## 他設定項目  
- devServer
webpack-dev-serverモジュールのオプション指定。(省略してdev-server)
```javascript
conte path = require('path');

devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 9000
}
```  

- watch
watchモードを有効にする。`$webpack -w`するのと同じ。  
ファイル変更を監視し、自動で再コンパイルできる。
watchOptionsでwatch対象や非対象を指定する事を可能。  
webpack-dev-serverではデフォルトで有効になっている。
```
module.exports = {
    /* 略 */
    watch: true,
    watchOptions: {
        ignored: /node_modules/
    }
    /* 略 *
}
```

- resolve.extensions
  import文で拡張子を省略する場合、resolve.extensionsに登録が必要。
  初期では、`[".wasm", ".mjs", ".js", ".json"]`となっている。
  なおこの設定は上書き式なので注意。
```
//以下の場合、".wasm"、".mjs"は省略不可となる。
module.exports = {
    /* 略 */
    resolve: {
        extensions: [".js", ".json", ".jsx"]
    }
}
```

- devTool
ソースマップを生成するかのオプション。
modeと合わせて記述しておけばとりあえずおｋ。
```javascript
mode: 'development',
devTool: 'source-map' 
```

- target
特定の環境をターゲットするようにwebpackに指示する為のオプション。
デフォルトはweb

```javascript
target: 'electron-main' //Electron様にコンパイルする指定
//他にもwebworkerやnodeなどがある。

```  

- experiments
ESの実験的な機能(ESモジュールや、WebAssemblyの最新版、awaitなど)を有効にするためのオプション。  
```javascript
experiments: {
    mjs: true, //EcmaScriptモジュールを有効
    asyncWebAssembly: true, //WebAssembly有効
    topLevelAwait: true //トップレベルのAwaitを有効
}

```