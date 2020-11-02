# Java
# 目次
- Java構成
- javaバージョン切り替え※Unix系
- javaコマンド 


# Java構成
JDK 12の場合を記載(主要のみ)
```
jdk-12
|--bin
|   |--java    (jre実行コマンド)
|   |--javac   (コンパイル)
|   |--javadoc (ソースファイルからhtml形式のdocment作る)
|   |--javap   (逆コンパイルコマンド)
|   |--jar     (アーカイブ化(jarファイル)または展開など)
|   |--jlink   (jre生成コマンド。今後jreはアプリ側がモジュラー化して配布)
|   |--jmod    (モジュール化コマンド。jarの上位単位となる。jdk9以降)
|  
|--lib  
|   |--classlist
|   |--
|   |
|   |
|   |
|   |
|
|

```

###  マニフェスト
mainクラス(エントリーポイント)の情報などが書かれたファイル。
コンパイル時に自動生成されるので基本は弄る必要ないが、場合によっては...。

### モジュール分割
javaはpackage分割するが、さらにモジュール分割できるようになった。

# javaバージョン切り替え  
alternativesを使えば、バージョンを簡単に切り替えられる様になる。  
勿論、Java以外の言語でも同じように利用可能。  

- インストールしたjavaコマンドをalternativesに登録する。
```shell
# 利用するコマンドをalternativesに登録する
$ alternatives --install /usr/bin/java java /usr/lib/java/jdk-11/bin/java 1
$ alternatives --install /usr/bin/javac javac /usr/lib/java/jdk-11/bin/javac 1
```  

- バージョン切り替え  
```shell
$ alternatives --config java

  3 プログラムがあり 'java' を提供します。

  選択       コマンド
-----------------------------------------------
   1           java-1.7.0-openjdk.x86_64 (/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.171-2.6.13.2.el7.x86_64/jre/bin/java)
*+ 2           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64/jre/bin/java)
   3           /usr/lib/java/jdk-11/bin/java

Enter を押して現在の選択 [+] を保持するか、選択番号を入力します: 3
```  


# コマンド
```java
//javaファイルのコンパイル
$javac [オプション] ソースファイル

[オプション]
--classpath パス　※パスを複数指定する場合は、";"繋ぎにし、クラスパスからの相対パスで指定
```
```java
//classファイル実行
$java [オプション] classファイル

[オプション]
--classpath ﾊﾟｽ


```　　