# Docker・DockerCompose
# 目次
- [Docker](#Docker)
- [DockerCompose](#DockerCompose)
- [Dockerコマンド](#Dockerコマンド)

# Docker
## 概要
Linuxカーネルが持つLXCというコンテナ機能を使って、他のプロセスやコンテナから独立したPC上でappを動かす技術。  
VMWareやVirtualBoxなどの仮想環境と比較して以下のメリット、デメリットがある。
- メリット
  - 構築・破棄が用意
  - 仮想コンテナをプロセスと同様に扱える。(高速)
  - リソースを必要最小限に絞れる。(各仮想コンテナで共通化できる箇所は共用する)
  - オーバーヘッドが少ない
  - app開発
    
- デメリット
  - ホストOSと異なるシステム(OS)をコンテナとして稼働できない。
  - 複数のディストリビュージョンは混在不可。
  - カーネルは各仮想コンテナで共用される為、カーネル機能に制限・制約がある。(開けるFS上限など)
  - Dockerで再現されるOS環境は、VirtualBox等の仮想環境とは完全にイコールではない(インフラ検証には向かない)


### ※補足
上記でいう「ホストOSと異なるシステムをコンテナとして稼働できない」や「カーネルの共用」に関する詳細は、LXCを学ぶ必要があるが概要としては以下の通りである。
- DockerはあくまでLinuxのコンテナ技術を利用したものである。(Linuxで動くものである)
- MacやWindowsでは本来Dockerを使用できない。
- Docker for MacやWindowsをインストールする事で、Hyper-Vなどの準仮想化技術で軽量かつ高速なLinux(Alpine Linux)を仮想マシンとして動かし、この上でDockerを利用することが出来る。
- ただ、上記とは別にWindowsコンテナというものもあり、これは名前の通りホストのWindowsOS上でDockerを動かすものである。つまり、WindowsServerなどがコンテナで利用できるようになる。

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
  - 命令一覧  

|命令|引数|備考|
|:--|:--|:--|
|FROM|<イメージ名>[:タグ名]|コンテナの元となるimageファイル|
|MAINTAINER|<名前>|作者を指定できる。|
|RUN|<コマンド>|RUN命令は既存レイヤーに新レイヤーを作成するコマンドであり、RUN命令の結果をコミットし、次のDockerfileステップへ渡す。一度実行したことがある(キャッシュがあればそれが適用される)|
|EXPOSE|<ポート番号>|run時に指定のポートをListenする。ただ、ホストからアクセスする為には別途-pによるフォワーディングが必要|
|ENV|変数名 値[or 変数名=値]|環境変数を定義できる|
|USER|ユーザー名|コンテナ内に指定ユーザーを作成する|
|ADD|送信元 送信先|ソースをホストやリモートURLからコンテナへ転送する。COPYとの違いはホスト上に無くてもできる点。また、RUNと違いキャッシュが使用されず、これ以降のコマンドも同様にキャッシュが利用されなくなる。|
|COPY|送信元 送信先|略。ADDとほぼ同様|
|CMD|<コマンド>|Dockerfile内で一度だけ実行できる命令。複数ある場合は最後尾のみ。docker run(start)時に実行される。|
|ENTRYPOINT|["実行ファイル","arg0","arg1"]|コンテナが実行するファイルの設定。exec形式とシェル形式があるが、exec形式推奨。|
|VOLUME|[マウントポイント名]|指定したmountポイントを作成する。|
|WORKDIR|パス名|作業ディレクトリを指定する(cdと同じ)。Dockerfile内で複数利用できる。|
|ARG||Dockerfile内で使用できる変数を定義できる。|
|SHELL||デフォルトシェルの設定。|

- ※注意点
  - Dockerfileは特に指定しない限り、キャッシュが適用される。その為、RUNコマンドはできるだけ先に記述し、ADD,COPYなどのコマンドは後に記述する様にして2回目以降の構築時間短縮を図る。  
  - Dockerfileは各命令実行毎にコンテナを作成し、それを重ねることでイメージを作成している。その為、できるだけ１つの命令(1行)でまとめて実行するようにする。(特にyum利用時などは&&等を使う)
  - 前述の通り、各命令毎にコンテナを起動している為、カレントディレクトリなどは保持されない。(毎回"/"からとして扱われる)その為、ディレクトリを維持したい場合は"WORKDIR"を使用する。実行ユーザーを指定する場合は"USER"。

  - コメント　
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
3. docker-compose upコマンドの実行 ※`$docker-compose up -d`
    
## docker-compose.yml
docker-compose.ymlの定義について記載する。
```yml
# docker-composeのバージョン '3'にしとけば良い？
version: '3'

# docker-composeにおいてDockerコンテナはサービスと呼ばれる。
# services配下に各service名(コンテナ名)として名前空間を定義する。
services: 
  db: # サービス名。ホスト名として扱える。
    image: mysql:5.7 # または「build: .」とすれば指定ディレクトリのDockerfileを利用する意味となる。
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


なお、ボリュームの定義は以下の3通りがある。
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

## 例：Go開発環境の構築例
GoコンテナとPostgresコンテナを使った開発環境をセットする。
```Dockerfile
# 最新のGo環境
FROM golang:latest
# コードを置くディレクトリを作成
RUN mkdir /go/src/
# コンテナログイン時の作業ディレクトリを指定
WORKDIR /go/src/
# ホストのファイルをコンテナに追加(マウントではない)
ADD . /go/src/
```

```yml
version: '3'
services:
  go-app:
    build: .
    container_name: go
    tty: true
    volumes:
      - .:/go/src
    networks:
      - gonet

  postgresql: 
    image: postgres:latest
    container_name: go-postgresql
    tty: true
    ports:
      - 5432:5432
    volumes: 
      - ./postgres/init:/docker-entrypoint-initdb.d/ # ここにホストに用意したsqlファイルをマウントするとコンテナ起動時に実行してくれる。複数用意する場合は、ファイル名先頭に「01_ファイル名.sql」「02_ファイル名.sql」とすることで順番に実行してくれる。
      # - ./pgdata:/var/lib/postgresql/data ←dbデータを永続化する場合はホストのディレクトリにマウント要。
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: root
      POSTGRES_INITDB_ARGS: "--encoding=UTF-8"
    hostname: postgres
    restart: always
    user: root
    networks:
      - gonet

# Goコンテナとpostgresコンテナ用のネットワークを定義
networks:
  gonet:
    driver: bridge

```

## 例：Jenkinsの構築例
ホスト側に以下のディレクトリを作っておく
- 「./jenkins_mnt_data」　：Jenkinsのデータをマウントするディレクトリ
  - `$ chown -R 1000:1000 ./jenkins_mnt_data`：Jenkins内ユーザがUID1000を使うため設定しておく

```yml
# Jenkinsイメージを使用。「jenkins」にすると古いイメージなので、必ず「jenkins/jenkins」とする事
version: '3'
services: 
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    tty: true
    volumes:
      - ./jenkins_mnt_data:/var/jenkins_home
    ports:
      - 8080:8080
      - 50000:50000
```  

1. `$docker-compose up -d`でコンテナ起動する。
2. 「http://localhost:8080」へアクセス
3. Jenkins初期設定ウィザードで初期パスワードを聞かれる為、「/var/jenkins_home/secrets/initialAdminPassword」 の内容を使う。
4. プラグインを選択する。(とりあえずはデフォルトでおｋ)
5. インストール失敗しても何度かリトライすればいけるはず
6. 管理ユーザ作成を行う
7. Jenkinsサーバが利用するアドレス指定を行う(Not nowでもおｋ)
8. 初期設定完了
9. あとはダッシュボードで色々設定する  


# Dockerコマンド
### 1. 起動・停止
```sh
# 既にあるコンテナの起動/停止コマンド
$docker start [オプション] コンテナID
$docker stop [オプション] コンテナID
```
[オプション]
-a コンテナを起動してログインする。(atatchする)  


### 2. リネーム、削除
```sh
# リネーム
$ docker rename 古いコンテナ名 新コンテナ名

# コンテナ削除
$ docker rm "コンテナ名" [...]

# イメージファイル削除
$ docker rmi "イメージファイル名"
```
### 3. ファイル操作
```sh
# ホスト⇔コンテナ間のファイル転送。
$ docker cp 転送元 転送先　※1
※1.コンテナ側は、"コンテナ名:パス"の形式で記述

```

### 2. 表示関係
```sh
# dockerイメージ一覧
$ docker images 
# 起動中のdockerコンテナ一覧(-a をつけると停止中のものも含む)
$ docker ps [オプション]
# コンテナ内で実行中のプロセス一覧表示
$ docker top "コンテナ名"
#　稼働コンテナで実行されているプロセスの転送先ポート(ポートマッピング)を表示
$ docker port "コンテナ名"

```
### 3. ネットワーク関連
```sh
# ブリッジネットワークの作成
$ docker network create "ネットワーク名"
```
--linkオプションもあるが、レガシーなので上記のようにネットワークを作成し、docker runの--netオプションで参加させる。  
また、デフォルトで、「bridge」というネットワークもあるがこれはDNSが設定されていない為、コンテナ名による名前解決ができない。その為、通常は新規にネットワークを作る。
  
```sh
# dockerネットワークの一覧表示
$ docker network ls
# 指定ネットワークの詳細表示
$ docker network inspect "ネットワーク名"
```

