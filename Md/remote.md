# Powershell リモート操作
# 目次
1. [設定](#設定)
2. [基本操作](#基本操作)
    - [SSH:リモートログイン](#SSH)
    - [SCP:リモートファイル転送](#SCP)
    - [サーバー操作(ホストOS-VirtualBox-仮想OS起動)](#サーバー操作(ホストOS-VirtualBox-仮想OS起動))
3. [VirtualBoxコマンド操作](#VirtualBoxコマンド操作)
4. [GUI操作](#GUI操作)
    - [X-Windows:リモートGUI](#X-Windowsの利用)
    - [WOL:リモート起動](#WOL)


# 1. 設定
## 文字化けする時
Powershellからリモートにログインした際、文字化けする場合は以下をそれぞれ試してみる。  

> Powershellのフォント変更
1. Powershellを起動し、下図1の赤丸内で右クリックし、プロパティを選択。  
2. 「フォント」タブのフォントでMSゴシックを選択。

||
|:--:|
|![](./pic/powershell.jpg)|
|図1.powershell|

> 文字コードの変更
1. Powershellで$OutputEncodingコマンドを打ち、文字コードを確認する。
2. リモート側のコンソール(or SSHログイン状態)で、```echo $LANG```打つ。
3. 1&2項で確認した文字コードに相違あれば、どちらかに統一する。    


# 2. 基本操作
## 2.1. SSH
- リモートサーバへログインするコマンド。telnetのセキュア版。(ウェルノウンポート：22)  
- サーバ側で、パスワードor秘密鍵の認証方式を設定できる。(デフォはパスワード？)　
- selinux有効だと、接続できなくなることがある。(/etc/selinux/config参照)
- sshの設定は/etc/ssh/sshd_config参照。
```shell
$ ssh -l リモートユーザー名 リモートホスト名(or IPアドレス)
```

  
## 2.2. SCP
- リモート⇔ローカル間でファイル転送を行うときに使うコマンド。  
ローカル/リモートいずれに向けて行う場合でも、ローカル(自分)側でコマンドを実行する。※1
- ディレクトリをコピーする場合は、「-r」オプションを付ける。
- リモート側の転送先は/home/ユーザー名が無難
- パスに漢字(日本語)含まれているとエラーになる。


※1.scpは「sshを使ってファイル転送」する為、sshでログインする必要なし。SSH+FTPと思っておk？？
```shell
//リモート ⇒ ローカルへコピー
$ scp リモートユーザー名@リモートホスト名:コピーしたいリモートのファイル名　ローカルのコピー先

//ローカル ⇒ リモートへコピー
$ scp ローカルのコピーしたいファイルパス　リモートユーザー名@リモートホスト名:コピー先パス
```  

## 2.3. サーバー操作(ホストOS-VirtualBox-仮想OS起動) 
#### 1). 起動フロー
- ホストOSでVirtualBoxを起動 
- VirtualBoxをコマンド操作し、ゲストOS(仮想マシン)を起動
- ゲストOS上で好きな操作する
```shell
//ホストOS側の操作
$virtualbox
$vboxmanage startvm VM1 

//ゲストOS側の操作(例)
$service postgresql-9.5 start
```

#### 2). 停止フロー
-  ゲストOSのシャットダウン操作
-  ホストOSシャットダウン操作

```shell
//ゲストOS
$shutdown -h now

//ホストOS
$shutdown -h now
```
## 2.4. サービス(デーモン)関係
#### 1). 概要
各ソフトウェアやプログラムをサービス(デーモン)として、`systemctl`コマンドで扱うことができる。(旧serviceコマンド)  
このコマンドでは、サービスの起動・停止・自動起動などの操作ができる。
よく使うコマンドは以下の通り。  
```shell
//サービス起動
$systemctl start "サービス名"

//サービス停止
$systemctl stop "サービス名"

//サービス再起動
$systemctl restart "サービス名"

//登録サービス一覧確認(オプション付けるとserviceに絞れる)
$systemctl list-unit-files [-t service]

//自動起動有効
$systemctl enabled "サービス名"

//自動起動無効
$systemctl disabled "サービス名"

//自動起動ステータス確認
$sytemctl is-enabled "サービス名"
```

各ソフトウェアは、基本的にはインストール時に自動でサービスとして登録される。
しかし、自作のシェルスクリプトなどをsystemctlで扱う為には、Unitファイルを作成する必要がある。
```shell
/etc/systemd/system


```

## 2.5. cron設定
自動実行のためのデーモン。スクリプトをcronに登録することで、指定周期で自動実行してくれる。
```shell
* * * * *  ユーザー名 実行スクリプトのパス
| | | | |  
| | | | `--- 曜
| | | `----- 月
| | `------- 日
| `--------- 時
`----------- 分

ex).3/1～3/22、(月)、12:00以降5分刻みにrootでtest.shを実行したい場合。

0/5 12 1-22 3 1 root /usr/bin/test.sh
```


# 3. VirtualBoxコマンド操作
CUIでVirtualBox関係の操作をする時は`VBoxMange`コマンドを使う。  
主要なコマンドは以下の通り。


# 4. GUI操作
## X-Windowsの利用
WindowsマシンからリモートのLinuxサーバのGUIを利用する方法。
#### 1. 準備
- Linux側(リモート)の準備
   - ssh-Server(リモートログインできる状態)
   - GUIがインストールされている。(ex.gnome)
- Windows側で用意するもの
   - Xming(下記URLのPublicDomainからDL)
   - Xming-fonts(同上)
    http://www.straightrunning.com/XmingNotes/
   - OpenSSH(OpenSSH-Win64.zip)
   https://github.com/PowerShell/Win32-OpenSSH/releases
#### 2. リモート側(Linux)
- sshd_conf編集
  /etc/ssh/sshd.confのX11Forwardingをyesにする。  
![](pic/sshd_conf.jpg)

- $DISPLAYの設定
  ```shell
  $ echo $DISPLAY
  //表示が何もない(何も設定されていない)なら以下を実行する。

  $ export DISPLAY=localhost:0.0
  ```

#### 3. ローカル側(Windows)
- Xming,fontsインストール
　基本デフォルトで問題なし。(fontsも同様)
   ![](pic/xming.jpg)
   ![](pic/xming1.jpg)
   ![](pic/xming2.jpg)
   ![](pic/xming3.jpg)  

- xmingの設定(xLanch)
  XLanchを起動し、以下の手順で設定を行う。
  ![](pic/xlanch1.jpg)
  ![](pic/xlanch2.jpg)
  ![](pic/xlanch3.jpg)
  ![](pic/xlanch4.jpg)

- xming(x0.hosts)の編集
  xmingフォルダ内のx0.hostsを開く。
  localhostの下にリモート接続するマシンのIPを追加する。

- ssh.exeのコピー
  DLしたOpenSSH-Win64.zipを解凍し、中にある"ssh.exe"をxmingフォルダ直下にコピーする。
  ※PuTTY使う場合は、plink.exeをコピーする。
　
　![](pic/xming.4.jpg)

- teratermの設定
  teratermのメニューから「"設定"->"ssh転送"->"リモートの(X)アプリケーションをローカルのXサーバに表示するにチェックを入れる"->"OK"」
  その後、teratermを再起動する。
  ![](pic/teraterm.jpg)

#### 4. 実行
以下の順で実行する。
1. xmingを起動する
2. teratermを起動しリモートログインする
3. GUIアプリケーションをコマンド起動する
   ```shell
   //最後に"&"をつける事
   $ firefox &
   ```
4. xmingでGUIが表示される

#### その他
WOL:リモート起動
