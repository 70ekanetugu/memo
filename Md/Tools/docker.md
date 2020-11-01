# Docker・DockerCompose
# 目次
- [Docker](#Docker)
- [DockerCompose](#DockerCompose)

# Docker
## 概要
コンテナ技術を用いて、プロセスを隔離して管理するツール(プラットフォーム)の事。(つまりコンテナは少し特殊ではあるが、実態は単なるプロセス)  

```json
PIDで考えると分かり易い。
・hPID : ホストOS上のPID
・cPID : コンテナ内のPID(ホストOSや他のコンテナからは不可視)
下図の通り、コンテナが異なれば(名前空間が異なれば)PIDが同じでも関係ない

hPID 1
|--hPID 2
|--hPID 3
|--.....
|--Dockerデーモン(dockerd) hPID 50
   |--コンテナデーモン(containerd) hPID 51
      |-- httpdコンテナ hPID 52
      |  `-- "cPID 1"   
      |-- mysql hPID 53 
      |  `-- "cPID 1"
      |.....
※dockerd : Dockerエンジントップレベルのデーモン
※containerd : コンテナやイメージｍネットワーク、ストレージを管理するdockerd配下の管理デーモン

ファイルシステムについても、通常のマウントと同じような考え方。
あくまでコンテナは、ホストOS上のディレクトリをマウントして利用しているに過ぎない。
```  

実際の操作に従って解説すると、DockerはDockerイメージを元にコンテナプロセスを起動・停止したりする。Dockerは「Dockerエンジン」と「Docker CLI」のサーバ・クライアント構成である。  
ちなみに、Dockerの鯨は「Moby」というらしい

## Dockerイメージ  
Dockerイメージは、複数のイメージレイヤーからなる。
1つ1つのイメージレイヤーは、ファイルシステム(/binや/etcなどのディレクトリ、ファイル)をtarアーカイブ化したものや、メタ情報が記録されたもの。  
`DockerFile`の記述は、命令ごとにイメージレイヤーを作成し、最後にこれをまとめている。完成したDockerイメージは読み込み専用であり書込みは出来ない。  
Dockerコンテナは、このDockerイメージの上に書込み可能なレイヤーを重ねたものである。

## Dockerネットワーク
Dockerには「bridge」「host」「none」といった3つのネットワークモデルがある。当然このネットワークも隔離される。  
各ネットワークはコンテナの停止など無しで、動的に付け外し出来る。
1. bridge
    - デフォルトは「ブリッジ」ドライバを使うネットワーク
    - 複数のブリッジを定義できる
    - コンテナ同士であれば、IPアドレスをコンテナ名で名前解決できる。
2. host
    - ホストOSのIFを直接利用する。
    - パフォーマンスは上がるが、セキュリティに難あり。
3. none
    - ネットワークIFを持たせない場合は、noneを指定する。

## Dockerボリューム
コンテナのデータを永続化する領域の事。  
Dockerが直接管理しているボリュームか、ホストOS上の任意のディレクトリをマウントして使う。  
ボリュームはコンテナ間で共有でき(１つのコンテナが占有しない)、各コンテナ内には好きな場所にマウント出来る。

## Docker利用フロー  
基本的なフロー配下の通り

1. Dockerfileの作成
    - ベースイメージ(FROM命令)
    - イメージレイヤーの作成(CMD RUN命令等)
    - Dockerボリュームの指定
2. Dockerネットワーク作成(デフォルトでよければ不要)
3. docker runコマンドによるコンテナの生成・実行

## Dockerfile
1. インフラ構成要素の記述(通常であれば、インフラ設計書やパラメータシートに書く内容)
 ・ベースになるDockerイメージ
 ・Dockerコンテナ内で行った操作(コマンド)
 ・環境変数などの設定
 ・Dockerコンテナ内で動作させておくデーモン実行
2. Dockerfileの基本構文　： "命令 引数" ->この書式を羅列する。
  ①命令一覧
　・FROM	：ベースイメージの指定。FROM [イメージ名][:タグ名]
　・RUN　	：ビルド(イメージ生成)時に任意のコマンドを実行。要は、dockerエンジン上のLinuxコマンド？
		　ex.RUN [コマンド] 	
　・CMD　　　	：コンテナ実行時のコマンド。(docker run時に指定するような)
　・LABEL　　	：ラベルを設定。
　・EXPOSE　　	：ポートのエクスポート(解放)。
　・ENV　　　 　：環境変数。
　・ADD　　　	：ファイル/ディレクトリの追加。
　・COPY　　　	：ファイルのコピー。	COPY [host側ファイル] [イメージ側指定ディレクトリ]
　・ENTRYPOINT　：docker run時に実行されるコマンド。実行時にCLIから引数を受け取れる。ENTRYPOINT ["パス"]
　・VOLUME　　	：ボリュームのマウント。
　・USER　　　	：ユーザーの指定。
　・WORKDIR　	：作業ディレクトリを指定。ない場合は新しく生成する。 WORKDIR [パス]
　・ARG		:Dockerfile内で使用できる変数を定義できる。
　・SHELL 	:デフォルトシェルの設定。
 ②コメント　
   #でその行はコメントとなる。
3. Dockerfileの作成フロー
　・FROMでベースイメージ指定
　・必要なミドルウェアをインストール、ユーザーやディレクトリを作成するコマンドの実行。
　・コンテナのデーモン起動
4. Dockerfile内コマンドの実行
　RUN [実行したいコマンド]
　=>2通りの記述がある。
  ①Shell形式での記述
　　 `RUN apt-get install -y nginx` ※nginxのインストール
　　　=>シェルで実行する。使用するシェルは、Dockerfile内SHELL命令で指定。
　②Exec形式での記述
　　 `RUN ["/bin/bash","-c","apt-get install -y nginx"]`
      =>
5. $docker buildコマンドでDockerfileを元にイメージを生成する。


## Dockerよく使うコマンド
`$ docker images ls`：イメージ一覧
`$ docker image inspect イメージ名`：イメージの詳細情報
`$ docker build -t 生成するイメージ名：タグ名　Dockerfileﾊﾟｽ `：Dockerfileを元にイメージを作成
`$ docker run -it --name コンテナ名`：イメージを元にコンテナ作成と実行
`$ docker ps -a`：コンテナ一覧
`$ docker start コンテナ名`：コンテナを起動
`$ docker exec -it コンテナ名 実行コマンド`：指定コンテナで指定コマンドを実行
`$ docker cp コンテナ名：コンテナパス ホストﾊﾟｽ`：コンテナ内のファイルをホストにコピー。逆にすれば、ホスト->コンテナとなる。


# DockerCompose
## 概要
一言でいうと、Dockerコンテナを纏めて実行管理するためのもの。  

一般的にWebアプリは複数のプロセスが組み合わさって動いている。(ex. httpd+mysql+redis)  
これらを１つのDockerコンテナ内で動かす事も可能だが、これをするとスケーラビリティが著しく低下する。(クラウドでは、コンテナを増やすことでスケールアウトしている)  
その為、各プロセス毎にコンテナを立ち上げる事になるが、このプロセスの数が仮に100個あると、100回Dockerコマンドを実行しなければならずその順序も意識しなければならない。これをまとめて行えるようにしたものがdocker-composeである。   

composeファイルは、yaml形式で定義する。

## docker-compose利用フロー
基本的なフローは以下の通り。
1. Dockerfileの作成
    - ベースイメージ(FROM命令)
    - イメージレイヤーの作成(CMD RUN命令等)
    - Dockerボリュームの指定　など
2. Compose.ymlファイルを作成
    - Dockerイメージ(or Dockerfile)に関する記述
    - ボリュームに関する記述
    - ネットワークに関する記述
    - 起動順序の記述　など
3. docker-compose upコマンドの実行
    
## docker-compose.yml
docker-compose.ymlの定義について記載する。
```yml
# docker-composeのバージョン '3'にしとけば良い？
version: '3'

# docker-composeにおいてDockerコンテナはサービスと呼ばれる。
# services配下に各service名(コンテナ名)として名前空間を定義する。
services: 
  db: # サービス名。ホスト名として扱える。
    image: mysql:5.7
    restart: always # 再起動条件
    environment: # 起動時の環境変数を設定
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
  web: 
    image: nginx # 使用するDockerイメージ
    deploy:
      replicas: 3 # コンテナの複製を3つ起動
      resources:
        limits: 
          cpus: "0.1" # CPUリソース制限
      restart_policy:
        condition: on-failure
    ports: # ポートフォワード設定「ホストOS:コンテナ」
      - "80:80"
    networks:
      internal: # internalという名前のブリッジネットワークに接続し、
      aliases: # ネットワークのエイリアスを、"frontend"に指定
        - frontend
    volumes: # ボリュームのマウント。ここでは「ホストOS:コンテナ内」
      - /etc/localtime:/etc/localtime:ro
    depends_on: # 先に起動させるサービスを指定。dbが起動してかｒwebが起動する。
      - db
networks:
  internal: 
```  


ボリュームの定義は以下の3通りがある。
```yml
#トップレベルの定義の場合
volumes:
  - /var/lib/mysql # コンテナ内のﾊﾟｽのみの場合
  # => ランダムidが割り振られたボリューム。使い捨て用途

  - db_data:/usr/local/data # 名前:コンテナパスの場合
  # => 名前付きのボリューム。ホスト上でdocker volumesしたとき参照しやすい。

  - ./configs:/etc/configs:ro # ホストOSﾊﾟｽ:コンテナﾊﾟｽの場合
  # => ホストOSの指定ディレクトリをコンテナにマウント。:roを付けると読込み専用
#========================================================================
# service配下にvolumesを定義する場合
# => そのサービスだけが参照する。 
services:
  db:
    image: postgres:9.4
    volumes:
      - db_data:/var/lib/postgresql/data 
```

