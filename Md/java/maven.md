# Maven概要 
Javaプロジェクトの管理ツール。 

pom.xml(Project Object Model：プロジェクトの様々な情報を扱う設定ファイル)にプロジェクトに必要なライブラリや、プロジェクトビルドに関する情報を記述する事で、ダウンロードおよびビルドを自動的に行ってくれる。  
(ex.手作業でやっていたjarのDL、配置などの自動化)

ライブラリは、パブリックリポジトリに登録されている。
セントラルリポジトリ(デフォルトの参照リポジトリ)はここを指している。

# 基本フロー 
1. maven設定※
1. pom.xmlの作成
2. mavenプロジェクトの更新 
3. maven install※

# Mavenの設定(settings.xml)
1. ローカルリポジトリの指定
Mavenはリモートリポジトリから依存ライブラリをローカルリポジトリへDLしている。

このローカルリポジトリは、デフォルトでは"ユーザー/.m2"内に作られる。

"mavenインストールフォルダ/conf"内にある"settings.xml"の`<localRepository>`タグでﾊﾟｽ設定できる。

eclipseで指定する場合は、まずsettings.xmlを作成し、eclipseの"設定->Maven->ユーザー設定"でsettings.xmlの場所を指定する。

2. プロキシの設定
プロキシ環境下でMavenを利用すると、デフォルトではセントラルリポジトリ(パブリックリポジトリ)からライブラリをDLできない。これもsettings.xmlで設定可能。

`<proxies>`タグで、プロキシサーバに関する情報を記述する。
- `<proxy>`タグ
  各設定の親要素
- `<id>`
  settings.xml内での識別名(自由記述)
- `<active>`
  基本true
- `<protocol>`
  使用するプロトコル。基本はhttpでおｋ
- `<username>/<password>/<host>/<port>`
  接続先プロキシの設定。必要ない項目は無くておｋ。
- `<nonProxyHosts>`
  プロキシサーバを使わずに接続できるドメインがあれば、指定する。

# pom.xml基礎
```xml
<!--サンプル-->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
	<groupId>com.mycompany</groupId>
	<artifactId>app</artifactId>
	<name>Sample5</name>
	<packaging>war</packaging>
	<version>1.0.0-BUILD-SNAPSHOT</version>
    <properties>
		<java-version>1.8</java-version>
		<org.springframework-version>4.3.7.RELEASE</org.springframework-version>
		<org.aspectj-version>1.8.10</org.aspectj-version>
		<org.slf4j-version>1.7.24</org.slf4j-version>
	</properties>
    <dependencies>
		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${org.springframework-version}</version>
			<exclusions>
				<!-- Exclude Commons Logging in favor of SLF4j -->
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				 </exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
        </dependencies>
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-eclipse-plugin</artifactId>
                <version>2.9</version>
                <configuration>
                    <additionalProjectnatures>
                        <projectnature>org.springframework.ide.eclipse.core.springnature</projectnature>
                    </additionalProjectnatures>
                    <additionalBuildcommands>
                        <buildcommand>org.springframework.ide.eclipse.core.springbuilder</buildcommand>
                    </additionalBuildcommands>
                    <downloadSources>true</downloadSources>
                    <downloadJavadocs>true</downloadJavadocs>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.5.1</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <compilerArgument>-Xlint:all</compilerArgument>
                    <showWarnings>true</showWarnings>
                    <showDeprecation>true</showDeprecation>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.2.1</version>
                <configuration>
                    <mainClass>org.test.int1.Main</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```
1. `<?xml>`タグ 
   xmlファイル書く時の定型文。基本このまま使用。
2. `<project xmlns～～>`タグ
   pom内のﾙｰﾄになる親タグ。参照スキーマの指定を行っているが、ここも特に変更する必要はなし。
3. `<modelVersion>`タグ
   Mavenのバージョン。昔はmavenが頻繁にバージョンアップしていた為、それを合わせるためのもの。現在は変更することはなさそう。
4. `<groupId>`タグ
   プロジェクトの識別名。javaでは、逆ドメイン記述が一般的。mavenリモートリポジトリでのドメイン名競合防ぐ為?
5. `<artifactId>`タグ
   プロジェクト生成物の名前。jar,war,ear生成時に使用される。
6. `<name>`タグ
   プロジェクトの表示名。ドキュメント作成に使用される。
7. `<packageing>`タグ
    作成する成果物のパッケージングタイプを指定。jar/war/ear/zipなど。
8. `<version>`タグ
   プロジェクトのバージョンの指定。(自由記述)
9. `<properties>`タグ
   pom.xemlで利用するプロパティ値などをまとめておくタグ。エンコード指定、JDKバージョン指定が出来、ここで定義したタグに記載した値をpom内で変数として扱うこともできる。
10. `<dependencies>`タグ
   依存ライブラリ情報を記述する。(javaシステムライブラリ除く)
- `<dependency>`
    各ライブラリ毎に記述する。この配下に指定のライブラリやスコープを記述する。
- `<groupId>/<artifactId><version>`
  各ライブラリもMavenプロジェクトである為、これらのタグで対象のライブラリを指定する。
- `<scope>`
  ライブラリの適用範囲を指定できる。Junitでテストするときなどは、"test"をスコープとして指定し、test実行時のみ利用される。つまり、ビルド(アーカイブ化)時、Junitのライブラリは組み込まれない。

1. `<build>　/　<plugins>`
   Maven自体に適用するプラグインを指定するタグ。
- `<plugin>`
  書き方は、dependencyと同じ。`<configuration>`タグでプラグイン側の設定を書くこともできる。

# mavenプロジェクト更新
pom.xml作成が終わったら、eclipseの場合は"プロジェクト更新"を押して、依存ライブラリのDLをする。IDE使わないならmvnコマンド使う(詳細は調べて)

# maven install
Javaプロジェクトをビルドする。pomで指定したpackaging形式(jarなど)で生成される。成果物は、targetフォルダに作られる。
ただ、eclipseの場合は、エクスポートでwar作成できるので使うことない??

# 嵌ったエラー
1. ライブラリファイル破損
プロジェクト更新後、IDE上でエラーが内にも関わらず、アプリケーション起動まではいくが、アクセス/利用が出来なかった。pomでライブラリのバージョンを過去バージョンに戻したら上手くいったが、最新verでは依然エラー。結果として、ライブラリDL時のファイル破損が原因であった。

ローカルリポジトリ内のライブラリを削除して、再度プロジェクト更新実施。結果、良好であった。
