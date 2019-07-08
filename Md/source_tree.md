# Source Treeメモ
## 画面構成
![](./pic/sourcetree.jpg)

①.Gitコマンド群  

②.選択中のブランチの表示切替
- ファイルステータス　:　変更ファイルをリスト化して表示する。
- Histroy　:　現在選択中のブランチヒストリーを表示する。
- Search　：　ログをキーワード検索する。

③.ブランチ一覧、スタッシュ
- ブランチの切替えやチェックアウトなど、ほとんどのブランチ操作ができる。作成は①の「ブランチ」で行う。  
- ブランチ直下は、ローカルブランチの一覧である。  
- リモートは名前の通り、リモートブランチが一覧表示される。

④.ブランチヒストリーが表示される。

⑤.変更ファイル一覧表示  
- 変更した作業ツリー上のファイル、又はインデックス上のファイルが表示される。  
右側には選択した変更ファイルの内容が表示される。


## 通常操作
### クローンの作成
1. 新規タブを開く
2. Cloneアイコンをクリックし、以下を順に入力する。
    - リポジトリパス　：　リモートリポジトリのURL（「http～～/XXX.git」で終わるはず)
    - 保存先のパス　　：　自身のPC内のどこにローカルリポジトリ・作業ツリーを作る場所を指定する。
    - 名前　　　　　　：　自動でつけられる。変えたければ変える。
    - Local Folder　 ： ルートのままでおｋ。 
    - 詳細オプション　：　作成するクローンのブランチを選択できる。
  
  ![](./pic/st2.jpg)


### ブランチの作成
1. 上部のブランチアイコンをクリック
2. ブランチ名を入力
3. コミットを選ぶ。
   - 作業コピーの親　：　現在のブランチから分岐させる。
   - 指定のコミット　：　指定したコミットから分岐させる。
  
### ブランチの切替え
   - 切り替えたいブランチをダブルクリックすれば勝手に切り替わる。
   
### ブランチの削除
   - 削除したブランチで右クリックして、削除選ぶ。
   - 又は、上部ブランチアイコンをクリックして「ブランチを削除」タブを選ぶ

### コミットのチェックアウト
- チェックアウトしたコミットの上で右クリックして、チェックアウト。

### マージ
   1. ベースとなるブランチに切り替えた状態(ブランチ名左に〇が付いた状態)で、マージする分岐ブランチ上で右クリックする。
   2.  「現在のブランチに～～～をマージする」を選択する。
   3.  ファストフォワードの場合は、チェック入れる。(プロジェクトによってはチェック禁止の場合も)
   4.  「はい」押せばマージされる。
>もしくは以下でもおｋ
   1. ベースとなるブランチに切り替えた状態で、上部マージアイコンを押す。
   2. ヒストリーからマージするブランチを選択し、OKを押す。

![](./pic/st3.jpg)
### プル
1. 上部プルアイコンをクリック
2. 基本的にチェックを入れるようなことはない(はず)
3. ok押す
4. conflictエラー出るようなら、競合を起こしてるファイルを確認・修正してから、再度プルする。 

### push間違えたとき
- 間違ったコミットそのものを消す
   1. ローカルでコミットをリセットする。(上部破棄アイコン)
   2. 上部プッシュアイコンをクリックする。
   3. リセット後の状態で、そのままpushすると競合エラーが出るため、確認窓下部の「強制プッシュ」にチェックを入れpushする。(コマンドでいうところの`$git push -f`)
   4. 証拠隠滅完了...
- 間違えたコミットは残し、打ち消したものをコミットする(revert)
   1. 間違えたコミットで右クリックして、「このコミットを打ち消す」をクリック
   2. 確認窓で「はい」押せば、コミット前の状態に戻る。
   3. コミットヒストリーにリバートのコミットが追加される。(間違えてコミットし、修正したことがわかる)
   4. その後、pushすればリモートにもrevertのコミットが表示される。
   5. 





## カスタム操作の追加
標準のSource Treeは、gitコマンドの全てをGUI利用できるわけではない。必要に応じて追加する必要がある。  
追加方法は以下の通り。(※Windowsの場合)
1. ユーザー/"ユーザー名"/AppData/Local/Atlassian/sourcetree/customactions.xmlを作成　(既にある場合は追記)  
2. 上記xmlに以下を記述
```xml
<?xml version="1.0"?>
<ArrayOfCustomAction xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <CustomAction>
  　<!--リモートで追跡中のファイルをローカルでのみ追加しないようにする。-->
    <Caption>ローカルで管理対象外に</Caption>
    <OpenInSeparateWindow>false</OpenInSeparateWindow>
    <ShowFullOutput>false</ShowFullOutput>
    <Target>git</Target>
    <Parameters>update-index --skip-worktree $FILE</Parameters>
  </CustomAction>
  <CustomAction>
    <!--管理対象に戻す。-->
    <Caption>管理対象に戻す</Caption>
    <OpenInSeparateWindow>false</OpenInSeparateWindow>
    <ShowFullOutput>false</ShowFullOutput>
    <Target>git</Target>
    <Parameters>update-index --no-skip-worktree $FILE</Parameters>
  </CustomAction>
  <CustomAction>
    <Caption>ローカル変更を無視</Caption>
    <OpenInSeparateWindow>false</OpenInSeparateWindow>
    <ShowFullOutput>false</ShowFullOutput>
    <Target>git</Target>
    <Parameters>update-index --assume-unchanged $FILE</Parameters>
  </CustomAction>
  <CustomAction>
    <Caption>ローカル変更の無視を解除</Caption>
    <OpenInSeparateWindow>false</OpenInSeparateWindow>
    <ShowFullOutput>false</ShowFullOutput>
    <Target>git</Target>
    <Parameters>update-index --no-assume-unchanged $FILE</Parameters>
  </CustomAction>
</ArrayOfCustomAction>
```
3. Source Tree再起動
4. ファイルステータスの作業ツリーのファイル上で「右クリック=>カスタム操作=>xmlに追加した操作」を確認する。

   