# Dockerメモ
# 目次
- 概要
- 基本操作フロー
- コマンド一覧

# 概要
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

# 基本操作フロー
```
基本的なフローは以下の2通り
【 Docker Hubのimageファイルを利用するフロー 】
   imageファイルのpull -> imageファイルを元にコンテナ作成・起動 -> コンテナ利用

【 Dockerfileを元に利用するフロー 】
   Dockerfileの作成(or DL) -> Dockerfileを元にコンテナを作成・起動 -> コンテナ利用

```

### DockerHubのimageファイルを利用するフロー
##### 1. imageファイルのDL
```sh
$ docker pull "イメージ名"
```  
DockerHubからimageファイルをDLする。  

[Docker Hub]
・docker imageのリポジトリ。「OFFICIA」にOKが付いているものが公式のイメージ。
・基本的にDockerfileが付いている(中身が明示されている)imageを利用するべきである。  

##### 2. コンテナの作成・起動  
```sh
$ docker run [実行オプション] --name "コンテナ名" [生成オプション] "imageファイル名"[:タグ名] [引数]

----------------------------------------------------------------------------

# 一度作成したことがあるコンテナの場合は、以下コマンドで起動。
$ docker start "コンテナ名"

```  
imageファイルを元にコンテナを作成+起動する。  

[実行オプション]
-d   => バックグラウンド実行。(デフォルトはこっち。大抵-itを使う)      
-it  => 対話実行＋(ホストOSの)標準出力へ表示する為のオプション
--add-host=ホスト名:IP => コンテナの/etc/hostsを指定できる。
--dns=IPアドレス       => コンテナ用のDNSサーバ指定
--expose              => 指定したポート番号を割り当てる
--mac-address=MAC     => MACアドレスを指定
--net                 => Dockerネットワークを指定。これによりホストポートを介さずにコンテナ同士の通信が可能になる。

[生成オプション]
-p 8888:80 => ホストOSとコンテナのポートフォワード設定。
-v C:\volume:/home/   => ホストとコンテナ間でファイル共有を行う。(マウント)
[引数]
・コンテナ起動後、実行するプロセスコマンドを書く。  
-itとの組み合わせで"/bin/bash"のようなシェルの起動コマンドを書く事が多い。


##### 3. コンテナの利用(ex 起動したコンテナへのログイン)
```sh
$ docker exec -it コンテナ名 "/bin/bash"
```
起動したコンテナでシェルを起動(exec)し、ホストOSのコンソール上で対話実行する。
(実質コンテナへのログイン操作)  

[実行オプション]：基本的にrun時と同様
-it : ホストOSのコンソール上で対話実行する為のオプション。

### Dockerfileを利用するフロー
##### 1. Dockerfile作成  
Dockerfileをテキストエディタで作成する。記述項目は以下の通り。
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
※注意点
- Dockerfileは特に指定しない限り、キャッシュが適用される。その為、RUNコマンドはできるだけ先に記述し、ADD,COPYなどのコマンドは後に記述する様にして2回目以降の構築時間短縮を図る。  
- Dockerfileは各命令実行毎にコンテナを作成し、それを重ねることでイメージを作成している。その為、できるだけ１つの命令(1行)でまとめて実行するようにする。(特にyum利用時などは&&等を使う)
- 前述の通り、各命令毎にコンテナを起動している為、カレントディレクトリなどは保持されない。(毎回"/"からとして扱われる)その為、ディレクトリを維持したい場合は"WORKDIR"を使用する。実行ユーザーを指定する場合は"USER"。

```sh
# PostgreSQLを構築するDockerfile例

FROM ubuntu
MAINTAINER naoe@yahoo.co.jp

# PGPキーの取得。
RUN apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8
# postgresqlのリポジトリ追加
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main" > /etc/apt/sources.list.d/pgdg.list

# インストール
RUN apt-get update && apt-get install -y python-software-properties software-properties-common postgresql-9.3 postgresql-client-9.3 postgresql-contrib-9.3
#postgresユーザーの作成
USER postgres

# postgresのロールとパスワード設定
RUN /etc/init.d/postgresql start && psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';" && createdb -O docker docker

# リモートからの接続を有効化する。
RUN echo "host all all 0.0.0.0/0 md5" >> /etc/postgresql/9.3/main/pg_hba.conf

# Listenアドレスを設定
RUN echo "listen_addresses='*'" >> /etc/postgresql/9.3/main/postgresql.conf

# ポートの解放
EXPOSE 5432

# 設定、ログ、dbクラスタをホスト側に永続化する為VOLUMEに設定。
VOLUME ["/etc/postgresql","/var/log/postgresql","/var/lib/postgresql"]

# デフォルトコマンドの設定(run時に何も指定が無ければこれが実行される。)
CMD ["/usr/lib/postgresql/9.3/bin/postgres","-D","/var/lib/postgresql/9.3/main","-c","config_file=/etc/postgresql/9.3/main/postgresql.conf"]

```
##### 2. Dockerfileのビルド(imageの生成)
```sh
$ docker build -t "作成するimage名" "Dockerfileがあるディレクトリパス"
```
Dockerfileがあるディレクトリパスは指定してもいいし、同じディレクトリへ移動してから"."指定でも良い。

##### 3. コンテナの生成・実行
```sh
$ docker run --rm -p 5555:5432 --name "コンテナ名" "image名"

# --rmはコンテナが終了した時に、コンテナを削除する為のオプション。残すならなくて良い。
```



# コマンド一覧
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
