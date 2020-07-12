# VSCode
# 目次

# 共通

## ショートカット一覧  
  
- VSCode全体  

|ショートカット|内容|
|:--|:--|
|Ctrl+Shift+P|VSCodeコマンドパレット表示|
|Ctrl+@|ターミナル表示|
|Ctrl+Shift+@|ターミナルを新規起動して表示|
|Ctrl+Shift+E|エクスプローラを開く|
|Ctrl+Tab|アクティブエディタの切り替え(パレットが出る)|
|Ctrl+Alt+→|エディタグループを分割(2窓)|

- エディタ―関連  

|ショートカット|内容|
|:--|:--|
|Ctrl+Enter|現在カーソルの下に空行を追加|
|Ctrl+W|選択中のタブを閉じる|
|Alt+↑ or Alt+↓|現在の行を移動|
|Alt+Shift+↑ or Alt+Shift+↓|現在の行をコピー|
|Ctrl+Shift+\\|対応するカッコにジャンプ|

# 拡張機能一覧
## Remote Development
リモート環境(Dockerコンテナ含む)のフォルダーをVSCodeで参照する為の拡張セット。  
- Remote Containersを使う場合は、ローカルにDockerは必要無いがDocker-CLIは必要。  
- SSHクライアントにPuTTYは利用できない。デフォルトで入っている`ssh`でOK。

リモートサーバ側で外部へポートを公開していなくても、VSCodeでポートフォワード設定することで接続できるようになる。手順は以下の通り
1. VSCodeのSSHエクスプローラで「転送されたポート」の「+」を押下
2. リモートサーバの転送したいポートを入力し、Enterを押下（ここでは"3000"にしたとする)
3. ローカルブラウザで、localhost:3000にアクセスすると、リモートサーバの3000ポートに転送される。

## Java環境
「Java IDE Pack」を入れればOK。Springも含めて全て入っている。  
また、別途setting.json(ユーザー用 or ワークスペース用)に"java_home"を登録する。
```json
{
//...
"java_home": "C:\\Program Files\\Java\\jdk-11.0.2",
//...
}
```
