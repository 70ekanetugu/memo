# Babel使い方
# 目次

# Babel概要
javascriptコードを指定の仕様へトランスパイルする為のツール。
babel単体では動作せず、トランスパイル先のpreset(pluginのコレクション)が必要。

# 基本フロー
1. `$npm install --save-dev babel babel-preset-env babel-cli`
2. プロジェクトルートに「.babelrc」を作成し、Babelの設定を記述。  

```json
//変換指定envのみの場合、preset-2015, 2016などの最新が適用される?
{
    "presets": ["env"]
}

//変換対象を特定のブラウザ指定する場合
{
  "presets": [
    ["env", {
      "targets": {
        "browsers": ["last 2 versions", "safari >= 7"]
      }
    }]
  ]
}
```  

3. ソースコードを作成
4. 変換操作(トランスパイル対象が単数か複数により以下いずれか)
   - `./node_modules/.bin/babel 作成コード -o 出力ファイル名`
   - `./node_modules/.bin/babel ./ソースがあるディレクトリ --out-dir 出力先ディレクトリ`


# Babel-Presets
Babelのpluginコレクション。トランスパイルに必要なプラグインセットをBabel本体に渡す役割(トランスパイル対象により異なるコレクションがある)
指定しなければデフォルトの、es5(`babel-preset-2015`)が適用される。
- babel-preset-env　ES用
- babel-prest-react React(JSX)用
- babel-prest-typescript Typescript用
- ...など

### Polyfill
Polyfillとは、ある言語バージョンにおける機能を過去の古いバージョンで再現する為の代替コードの事。
jsの場合、Promiseやgenerateなどの構文以外の部分。分かり易く言えば新規追加された標準関数とかオブジェクトを実現するコードを指す。
Babelのpresetにおいて、デフォルト設定では構文変換のみでPolyfillはoffになっている。その為、以下のオプション指定が必要。
```
{
  "presets": [
    ["env", {
        "targets": [...],
        "useBuiltIns": "usage"
      }
    ]
  ]
}
```
もしくは、アプリ内の1箇所に下記インポート文を記述。(複数箇所あるとエラーになる)
```
import "@babel/polyfill";
```
