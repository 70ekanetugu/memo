# Springメモ
# 目次
[1. 概要](#概要)
[2. DI](#DI)
[3. MVC](#MVC)
[4. JDBC/JPA](#JDBC/JPA)
[5. Security](#Security)
[6. Cloud](#Cloud)

# 1. 概要
Springは様々な用途のフレームワークが存在するが、共通するコア機能として「DIコンテア×AOP」がある。
1. DIコンテナ
依存性の注入を行うためのコンテナ(Bean(≒インスタンス)管理している)を使用して、オブジェクトの結合度を下げる。
設定から使用までの大まかなフローは以下の通り。(設定がJavaConfigの場合)

- Configurationクラス(DIコンテナの設定)の準備
  @Configurationをクラスにつけ、@Beanをつけたインスタンスを返すメソッドを定義したもの。

- ApplicationContext(DIコンテナ)の生成　
  ApplicationContextインターフェースを実装したクラスをインスタンス化する。
  (引数にconfigrationクラスを渡す)

- Beanの取得(ApplicationContext.getBean()の使用)
 前述のApplicationContext実装クラスのgetBean(クラス名)を使用することで、DIコンテナに登録されBeanを取得する。

#### 用語
- Spring Bean(単にBeanと呼ばれるが、POJOのBeanと区別する為、ここでは頭にSpringとつけた)
  DIコンテナに登録するコンポーネントの事。(@Component, @Controller, @Service, @Repositoryなど)
- Bean定義
  Configurationの事。 代表的なのは以下の３つ
  - JavaConfigベース(@Configuration)
  - XMLベース
  - アノテーションベース(@Component、@ComponentScanの使用)

- ルックアップ
  DIコンテナから@Autowiredなどを用いてBeanを取得する事。ルックアップにもいくつか種類がある。

1. AOP




### Bean定義
spring bootでは設定ファイルなしでも動くが、規模大きい場合はxmlで書いている。xmlの読み込みは、mainメソッドがあるクラスで以下のｱﾉﾃｰｼｮﾝをつける。これで、起動時に設定が読み込まれる。
- @Configuration
- @ImportResource("xmlファイル名")

bean定義はいくつか方法があるが、一般的にはxml,Java Config,LDAPなどがある。

1. アノテーションでDIする時のxml記述(共通)
   アノテーションでDIをする場合も、最低限のxml記述が必要。("レガシーspring"の場合)
   xmlファイルの名称は自由だが、一般的には"applicationContext.xml"や"servlet-context.xml"である。
   
```xml
<?xml varsion="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/scheam/context/spring-context.xsd">

    <context:annotation-config/>
    <context:component-scan base-package="sample"/>
    <context:property-placeholder location="jdbc.properties"/>
    <!--@Autowiredによるインジェクション対象クラスが2つ以上ある場合の対策(テスト用コードの置き換え時)
    <context:component-scan base-package="sample.di.business"/>
    //<context:component-scan base-package="sample.di.dataaccess"/>
    <context:component-scan base-package="test.*"/>
    -->
</beans>
```
- 主なスキーマ
  - beansスキーマ(spring-beans.xsd)
  Bean(コンポーネント)の設定
  - contextスキーマ(spring-context.xsd)
  Bean(コンポーネント)の検索やアノテーションの設定
  - utilスキーマ(spring-util.xsd)
  定数定義やプロパティファイル読込みなどのユーティリティ機能
  - aopスキーマ


- タグ解説
  - `<context:annotation-config/>`
    @Autowiredを利用する宣言。後述のcontext:component-scanやmvc:annotation-drivenがBean定義ファイル上に記述されていれば省略可能。  
  - `<context:component-scan base-package="パッケージ名. ...."/>`
    @Component,@Serviceなどのｱﾉﾃｰｼｮﾝ付きクラスを読込み、DIコンテナに登録する。base-package以下を検索対象とする。
    - `<context:include-filter type="タイプ名" expression="指定ﾊﾟｽ"/>
    component-scanの詳細設定の１つ。検索対象のフィルタリングかけられる。


2. Bean定義ファイル 
@Autowired、@Componentで実現したことをbean定義ファイルで実現すると以下の様になる。(スキーマ省略)
```xml
<beans xmlns=~~~~省略~~~~~~>
    <bean id="productService"
        class="com.example.service.ProductServiceImple"
        autowire="byType"/>
    <bean id="productDao" class="com.example.dao.ProductDaoImpl"/>
<!--beanタグの使い方
    クラスYにクラスXをAutowiredでインジェクションする場合
    <bean id="オブジェクト名X" class="パッケージ名.クラス名X"/>
    <bean id="オブジェクト名Y" class="パッケージ名.クラス名Y" autowire="byType"/>

    クラスBにクラスAを明示的にインジェクションする場合
    <bean id="オブジェクト名A" class="パッケージ名.クラス名A"/>
    <bean id="オブジェクト名B" class="パッケージ名.クラス名B">
        <propery name="インスタンス変ス名" ref="オブジェクト名A"/> 
    </bean>   

    プロパティに文字列とスカラ値をセットする
    <bean id="オブジェクト名C" class="パッケージ名.クラス名C">
        <property name="インスタンス変数名" value="Hello"/>
        <property name="インスタンス変数名" value="109"/>
    </bean>
    -->
</beans>
```
- beanタグ(主な)
    - id
    オブジェクトを一意にするID
    - class
    idの実体。パッケージ名+クラス名
    - scope
    オブジェクトのスコープを指定する。singleton/prototype/request/session/applicationなどがある。省略するとsingletonになる。
    - name 
    オブジェクト名を定義。
    - parent 
    設定を引き継ぐBeanのオブジェクト名を指定
    - autowire
    値には、no/byName/byType/constructorなどがある。

- bean定義ファイルの分割、インポート
bean定義ファイルは分割して定義し、インポートで纏めることもできる。
```xml
<beans>
    <import resource="config/services.xml"/>
    <import resource="dataaccess.xml"/>

    <bean id="servie" class="....."/>
    <bean id="dao" class="....."/>
</beans>
```

- プロパティファイルの読み込み
  以下の様にすることで、serviceクラスのmessage変数に、"Hello Spring"が代入される。
```xml
<beans>タグ内で....
    <util:properties id="msgProperties" location="classpath:sample/config/message.properties"/>

    <bean id="message" class="sample.MessageServiceImpl">
        <property name="message" value="${msgProperties.message}"/>
    </bean>
</beans>
```
```java
public class MessageServiceImpl implements MessageService{
    private String message;
    public void setMessage(String message){
        this.message=message;
    }
    public String getMessage(){
        return message;
    }
}
```
```
message.propertiesファイル内...

message="Hello Spring"

```

3. Java Config
Bean定義ファイルでやったことと同じことがjavaコードで書ける。
```java
@Configuration
@ComponentScan("sample.web.controller")
@Import({InfrastructureConfig.class,WebConfig.class})
public class AppConfig{
    @Bean(autowire=Autowire.BY_TYPE)
    public ProductServiceImpl productService(){
        return new ProductServiceImple();
    }
    @Bean(scope=Scope.SESSION)
    public ProductDaoImple productDao(){
        return new ProductDaoImpl();
    }

}

```

### ロギング　
spring bootでは全ての内部ロギングで"CommonsLogging"を利用している。

# 2. DI


# 3. MVC


# 4. JDBC/JPA
### 4.1. データベース接続設定
```yml
# SpringBootの場合、application.ymlに以下の設定をする。
# configでプロパティの読み込みを設定していれば、名称は好きに変えられる。
spring:
  datasource: 
    url: jdbc:postgresql://localhost:5432/DB名
    driver-class-name: org.postgresql.Driver
    username: ユーザー名
    password: パスワード
```
### 4.2. Java Config
```java
//Web


```

# 5. Security
 認証・認可
1. LDAP(Lightweight Directory Access Protocol)
ユーザやコンピュータの情報を集中管理するディレクトリサービスのアクセスに用いるプロトコル。(winのActiveDirectoryみたいな?)
LDAPクライアントはLDAPサーバ上のデータを「検索・参照」したり、追加・削除・変更などの操作ができる。ただ頻繁なデータ変更や複雑なデータ管理には向かないため、あくまで情報検索・参照がメイン。
認証情報の保管・検索照合によく使われる。

springで使うときは、以下の様な感じ。
``` yml
#[yml]:LDAPサーバの設定
spring:
  ldap:
    embedded:
      base-dn: dc=springframework,dc=org
      ldif: classpath:test-server.ldif
      port: 8389
```
``` java
//@EnabeWebSecurityの使用
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    //JavaConfigで記述
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().fullyAuthenticated()
                .and()
            .formLogin();
    }
    //AuthenticationManager(認証情報を持つオブジェクト)の設定
    @Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .ldapAuthentication()
                .userDnPatterns("uid={0},ou=people")
                .groupSearchBase("ou=groups")
                .contextSource(contextSource())
                .passwordCompare()
                    .passwordEncoder(new LdapShaPasswordEncoder())
                    .passwordAttribute("userPassword");
    }
    //データソース
    @Bean
    public DefaultSpringSecurityContextSource contextSource() {
        return new DefaultSpringSecurityContextSource(Arrays.asList("ldap://localhost:8389/"), "dc=springframework,dc=org");
    }
}
```




