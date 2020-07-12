# npm使い方
# 目次

# 基本フロー
1. `$npm init -y` 
2. `$npm install パッケージ名@バージョン --save-dev`
3. `$npm install パッケージ名@バージョン --save`
4. package.jsonで、`scripts`の登録(エイリアス)
5. package.jsonに`private: true`を追記
6. ソース記述後、`$npm run 登録したscripts`でビルドや、コード実行

# コマンド一覧
- npm init
  - package.jsonを生成するコマンド。package.jsonの中身については後述する。

- npm install パッケージ名
  - npmパッケージをローカルにインスト―ルするコマンド。
  - パッケージ名が無い場合、カレントのpackage.jsonにしたがって依存関係がインストールされる。
  - -gをつければグローバルインストールされ、パスも勝手に追加してくれるが、流行として、-gを付けてないローカルインストールが主流。(各プロジェクトを汚さないようにする為)
  - パッケージ名@バージョン番号とすることでバージョン指定できる。
  - --saveオプションで、package.jsonの依存関係に自動で記述してくれる。
  - --save-devオプションはpackage.jsonの開発に関する依存関係に自動記述。

- npm uninstall パッケージ名
  - ローカルのnpmパッケージをアンインストールする。
  - -gをつければグローバル、付けなければカレントのプロジェクトからパッケージを除外
  - --saveや--save-devはinstallの逆で、package.jsonから削除する。

- npm list
  - インストールされているパッケージの一覧表示
  - -gつければグローバルも

- npm info パッケージ名
  - パッケージ情報を確認する。
  
- npm outdated パッケージ名
    - パッケージの最新版確認
    - パッケージ名指定しなかればインストールされているパッケージ全てを確認

- npm update パッケージ名
  - 更新する

- npm search パッケージ名
  - パッケージの検索
  - 部分一致でも検索してくれる。

# package.json
javascriptプロジェクト(npmモジュール)の環境設定ファイル。

### name
モジュールの名前。
ソース内でimportやrequire()でモジュール読み込みで利用される。npmモジュールはnameとversionで一意となることが想定されているので、他のライブラリと名前が重複してはいけない。
```
{ "name": "react-native-card-media" }
```

### version
モジュールのバージョン。必須項目。
npmモジュールはnameとversionで一意となることが想定されている。バージョンアップのnpm公開するときは、バージョン番号の更新を忘れないように。
```
{ "version": "0.0.5" }
```

### private
このプロパティがtrueになっていると、モジュールの公開ができない。公開しないプロジェクトは誤って公開しないようにtrueにしておく。
```
{ "private" : true }
```

### main
モジュールの中で最初に呼ばれるスクリプトファイルを指定。
例えばモジュールにfooと名前をつけ、それをユーザーがインストールし、require("foo")を実行した時にmainで指定したモジュールのexportsオブジェクトが返される。
パッケージルートからの相対パスを指定しなければいけない。
```
{ "main": "index.js" }
```

### scripts
任意のshell scriptを実行するエイリアスコマンドを定義できる。
```
{
  "scripts": {
    "test": "eslint *.js ./components/*.js ./example/*/*.js",
    "start":      "node app.js",
    "production": "NODE_ENV=production node app.js"
  }
}
```
キーはnpm test、npm startのようにエイリアスとして利用できる名前となり、値にはshell scriptをワンラインで指定する。

ただし、testやstartなどの予約語以外をエイリアスコマンドとして、登録した場合はnpm run productionと実行する。

ワンラインのshell scriptを記述する際はdependenciesやdevDependenciesに入っているモジュールの bin は、自動的に PATH に入る。
```
{
  "devDependencies": {
    "eslint": "^3.18.0"
  }
}
```
と、していれば
```
{
  "scripts": {
    "test": "node_modules/.bin/eslint *.js"
  }
}
```
ではなく
```
{
  "scripts": {
    "test": "eslint *.js"
  }
}
```
のように書ける。
基本的にはbuildやwatchを登録したりするのに使う。以下sassとTypescriptのwatcher例。
一度、`$npm run watch`を実行すれば、sassとtsが変更監視状態になる。
```
{
  "scripts": {
    "watch": npm run watch:sass & npm run watch:ts",
    "watch:sass": "sass --watch input.sass output.css",
    "watch:ts": "tsc -w main.ts"
  }
}
```
より複雑なタスク登録をしたければ、gulp等を使う。


### repository
ソースコードが管理されている場所を指定する。
```
{
  "repository": {
    "type": "git",
    "url": "git+ssh://git@github.com/dondoko-susumu/react-native-card-media.git"
  }
}
```

