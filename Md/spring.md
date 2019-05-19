# Springメモ

## Bean定義
bean定義はいくつか方法があるが、一般的にはxml,Java Config,LDAPなどがある。

spring bootでは設定不要なわけではなく、規模大きい場合はxmlでも書いている。

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

## ロギング　
spring bootでは全ての内部ロギングで"CommonsLogging"を利用している。