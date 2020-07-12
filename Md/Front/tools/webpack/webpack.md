# webpack使い方

# 概要
javascriptのバンドルツール。  
cssや画像などもjsファイルにバンドルできる。  
設定ファイルやプラグインにより、バンドルするまでの中間操作としてbabelでトランスパイルしたり、scssのコンパイルなどができる。
その為、ビルドツールとしての利用が一般的。

# 基本フロー
1. `$npm install --save-dev webpack babel-core bable-loader babel-preset-env`
2. 「.babelrc」を作成し、`{"presets: ["env"]"}`を記述
3. 「webpack.config.js」を作成し、以下を記述
```
module.exports = {
        // ビルドの基点となるファイル
        entry: './src/entry.js',
        // ビルド後のファイル
        output: {
            path: __dirname + '/dist',
            filename: 'bundle.js'
        },
        // 拡張子が.jsのファイルはbabel-loaderを通してビルド(node_modulesは除外)
        module: {
            loaders: [{
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader'
            }]
        }
};
```
4. ソースファイルを作成
5. `./node_modules/.bin/webpack`を実行し、ビルド
6. 設定で指定したディレクトリにビルド後のファイルが出力されている。

# Loader
webpackは本来、複数のjs(json)ファイルを1つにまとめる事が仕事の為、js以外のファイル(css, 画像など)も纏める場合は、jsで扱える形にコンパイルする必要がある。
つまりバンドルまでの中間操作的な変換処理を行うものをLoaderと呼ぶ。
- babel-loader　Babelを使用する為必要
- sass-loader sassをコンパイルする
- css-loader cssの@importやurl()を解決して読み込む
- file-loader コンテンツ内容のハッシュ値計算し、保存する。
- style-loader styleタグをDOMに追加する処理。css-loaderと併用。
- raw-loader テキスト・バイナリファイルを読み込む
- ts-loader Typescriptをコンパイルする。
- url-loader コンテンツをBase64エンコードする。(css内のurl()の画像など
- をbase64にし、httpsリクエストが不要になる)


# webpack.config.js
webpackの設定ファイル。webpackがデフォルトで読み込むファイル名が「webpack.config.js」であるが、独自の名前を付けることも出来る。  
基本構成は以下の通り。
```
module.exports = {
    mode: "development", //バンドルモード指定。本番なら"production"、開発なら"development"
    devtool: "source-map", //開発時、バンドル前後のファイルを紐づけるマップ。
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
#### Loaderの指定
module.rulesにLoaderの指定を行う。主要なものは以下。
- test : Loaderに通すファイルを文字列または正規表現で指定。一般的に拡張子。
- use: どのLoaderを使用するか指定。
```
//babel-loaderとraw-loaderの設定例
module.exports = {
    /* 略 *

    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
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


#### 他設定項目
- watch
watchモードを有効にする。`$webpack -w`するのと同じ。
watchOptionsでwatch対象や非対象を指定する事を可能。
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
