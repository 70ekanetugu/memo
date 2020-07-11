# SSH
# 目次
- [SSH鍵認証設定](#SSH鍵認証設定)

# SSH鍵認証設定
### 1. 非対称鍵を生成  
- 以下のコマンドをSSHクライアント側で実行。※リモートで生成してもいいけど秘密鍵転送するリスクがある。  
- パスフレーズなど聞かれるが、セキュリティ要件等なければそのままEnterでよい。  
※Github用のSSH鍵は、パスフレーズの利用を推奨している。
```shell
$ ssh-keygen -t rsa -b 4096 [-C "メールアドレス"] [-f 生成鍵の保存先パス]
```  
上記コマンド実行後、-fで指定した場所に以下の鍵ができる。(※指定無しの場合`~/.ssh`に作成される。)
- id_rsa
- id_rsa.pub  

### 2. 非対称鍵のパーミッション変更
- SSHの鍵認証はパーミッションが限定されている必要がある為、変更する。

```shell
#Mac等の場合
$ chmod 700 ~/.ssh/id_rsa
$ chmod 600 ~/.ssh/id_rsa.pub

#Windowsの場合
# エクスプローラで「id_rsa」と「id_rsa.pub」の権限を以下の通り変更する。
#「id_rsa」：ログインユーザーにのみ全権限を与える。
#「id_rsa.pub」：ログインユーザーに「読み取り」「書込み」のみ与える。
```

### 3. リモートサーバへ公開鍵を転送_登録
- SSHクライアント側で以下のコマンドを実行し、公開鍵を登録する。
- リモートサーバの`~/.ssh`に「authorized_keys」が作成される。
```shell
# Mac等の場合
ssh-copy-id ~/.ssh/id_rsa.pub

# Windowsの場合(上記ssh-copy-idの代替)
cat ~/.ssh/id_rsa.pub | ssh remoteUser@host "cat >> ~/.ssh/authorized_keys"
```  

### 4. リモートサーバのSSH設定  
- 「/etc/ssh/sshd_config」ファイルを開き、以下の設定を変更する。
```shell
# 鍵認証を有効にする。初期はコメントアウトされているので"#"を消す。
PubAuthentication yes

# パスワード認証を無効にする。※鍵認証が確認出来てからでよい。後でやる。
PasswordAuthentication no

# rootユーザでのssh接続を無効にする。※セキュリティ気にしないのであれば不要
PermitRootLogin no
```  
- 変更後、sshdを再起動する。  
```shell
$ systemctl restart sshd
```  

### 5. リモートサーバの公開鍵権限変更  
- 「~/.ssh」配下の公開鍵権限を変更する。  
```shell
$ chmod 600 ~/.ssh/id_rsa.pub
$ chmod 600 ~/.ssh/authorized_keys
```  

### 6. ログイン確認  
- SSHクライアント側でパスワード要求無しでログイン出来る事を確認する。
```shell
$ ssh -i id_rsa ユーザ@ホスト
```