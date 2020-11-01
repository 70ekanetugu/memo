# Selenium
# 目次
[・概要](#概要)
[・環境構築](#環境構築)

# 概要
Seleniumはブラウザー自動化を可能にし、それを支えるツール・ライブラリ群プロジェクトである。要約すると、Webアプリケーションのテスト自動化ツールである。  
WebブラウザでUIをクリックしたりする動作を自動し、画面の表示内容確認やシステムレスポンス測定、スナップショットができる。
[※Selenium公式サイト](https://selenium.dev/)    

Seleniumの構成は以下の通り
1. Selenium Client
    - Seleniumを使う為に最低限必要なライブラリであり、これを使用してテストコードを書く。
    - 各言語で用意されている為、自分の使いたい言語で書ける
2. Selenium WebDriver
    - Seleniumを使う為に最低限必要なソフトウェア。各ブラウザ毎にある。
    - Selenium Clientからhttp経由で各ブラウザ用WebDriverを呼び出し、WebDriverが直接実際のブラウザを起動し、動作させる。
3. Selenium Grid(Selenium Server)
    - 様々なプラットフォーム上のブラウザでテストケースを実行できる。
    - WebDriverテストの開発後、複数のブラウザー×OSの組み合わせでテストを実行する場合に必要になる。
4. Selenium IDE
    - 名前の通りIDE。
    - ブラウザのプラグインとしても提供されている為、正直不要

## サポート対象ブラウザ
|ブラウザ|バージョン|
|:--|:--|
|Chrome|全てのバージョン|
|Firefox|54以上|
|Edge|84以上|
|IE|6以上|
|Opera|10.5以上|
|Safari|10以上|

# 環境構築
最低限必要なものは以下の通り(※Chromeのみ対象とする)

|必要なもの|説明|
|:--|:--|
|WebDriver|ブラウザ操作を行う為のAPIを備えたツール。SleniumClientでは、ここのAPIへhttpリクエストを送信する事となる。|
|Selenium Clientライブラリ|selenium-java.jar等の事でありテストコードを書く為のライブラリ。(全サポートブラウザをサポートするライブラリ。固有の場合は、selenium-chrome-driver.jarなどがある)。各言語、Java,Python,C#,Ruby,javascript用がある。|

今回は、Dockerを使用してこれらの環境構築を行う。
テスト対象のWebアプリケーションが構築済みとして、進める。
### Seleniumコンテナの構築
- Dockerfileの作成
  - Selenium用のコンテナイメージを作成する。

```yml
# 既存のChrome用VNC付きSelenium環境を利用
version: '3'
services:
  chrome:
    image: selenium/standalone-chrome-debug
    container-name: chrome-selenium
    ports:
      - 4444:4444 # Selenium Server用の公開ポート
      - 5900:5900 # VNC用のアクセスポート
    volumes: 
      - ./src/chrome:/selenium/src/chrome/
```

