# VSCodeメモ
# 目次
- Node.jsデバッグ
- Docker+Node.jsデバッグ

# Node.jsデバッグ
VSCodeでデバッグを行うためには、対象のディレクトリを開いた状態で行う。  
また、launch.jsonを作成する必要がある。
launch.jsonはデバッグモード(虫アイコン)の歯車アイコンを押すと作成される。

#### 1. launch.json
以下の様なjsonファイルが作成される。  
programの値を、起点となるjsファイルに変える。(エントリーポイントを指定)

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${file}"
        }
    ]
}

```
#### 2. デバッグ起動
デバッグモードで、緑の矢印アイコンを押す。
あとは好きにBPの設置をする。

# Docker+Node.jsデバッグ  
#### 1. Dockerコンテナ作成
```sh
$ docker run -p 8888:8080 -p 29830:2983 image名
```
上記コマンドの様に、ポートを２つフォワーディングする。  後の2項、3項で使用する。
1つめは、通常通りのホスト-コンテナ間のapp用フォワーディング。  
2つめは、デバッグ用のポートである。


#### 2. launch.json
launch.jsonのに以下の記述を行う。
```json
"configurations": [
        {
            //Dockerを利用する際の定数
            "name": "Docker: Attach to Node",
            //デバッガの種類
            "type": "node",
            //既に起動されているものをデバッグするときはattach。STの様な起動時にデバッグも始める場合はlaunch
            "request": "attach",
            //デバッグに使用する任意のポート。(ホスト側)
            "port":29830,
            //ホスト側のアドレス
            "address":"127.0.0.1",
            //実際にBPを置くファイルのappルートパス(ホスト側)
            "localRoot":"C:\\Users\\u20190501\\Documents\\その他_社内関係\\99_勉強資料\\00.sample_code\\docker\\node_app",
            //対象のリモートappルートパス(コンテナ側)
            "remoteRoot": "/usr/src/app",
            //使用プロトコル。特に指定無ければauto。
            "protocol": "auto"
        }, 
        ....
```



#### 3. コンテナ側、Node.js実行
```node
node --inspect=0.0.0.0:2983 ./server.js
```
コンテナ自身のアドレス及びポートと、デバッグ対象のエントリーポイントを指定する。  
既にバックグラウンドでnode.js起動している場合は、`ps -a`後、`kill -9 PID`でキルする。

#### 4. デバッグ起動
- VSCodeでデバッグモードに切り替え、緑ボタン横のプルダウンを押す。  
- launch.jsonで指定したname=`Docker Attach to Node`を選択する。
- 緑ボタンを押下するとデバッグ起動する。  

※デバッグコンソールがオレンジ色にならないなら、どこかエラー出てるので要確認。