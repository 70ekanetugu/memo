# Thymeleafメモ

# 目次
[基本属性](#基本属性)
[テンプレート指定](#テンプレート指定)

# 基本属性
|属性|内容|例|
|:--|:--|:--|
|th:|thymeleafの名前空間。||
|th:text|要素の値をセットする。|`<p th:text="${data}">ここが${data}の内容で置き換わる</p>`|
|th:value|要素のvalue属性に値をセットする。|`<input type="hidden" th:value="${data}"/>`|
|th:if|条件分岐。trueならこの属性を付けたタグを表示する。子要素にも影響あり。|`<div th:if="${flag}"></div>`|
|th:unless|ifの逆でfalseなら、表示|`<div th:unless="${flag}"></div>`|
|th:each|繰り返し処理。後述のオブジェクトと一緒によく使われる。||
|th:object|オブジェクトを定義できる。|`th:object="${hoge}"`以降hogeへのアクセスは`*{}で省略できる。`|
|th:block|疑似ブロック。レンダリング後は、表示されない疑似的なブロックとして利用可能。||
|${}? : |三項演算子|`<p th:text="${flag} ? ${data} : null"></p>`|
|and or not !|論理演算子。th:の右辺であれば、${}で囲わなくても使用できる。|`<span th:text="true and false or true and not true or !false"></span>`|
|> < >= <= != ==|比較演算子||
|${...}|SpEL式と呼ばれるオブジェクトにアクセスする為の記述。※1||
|@{...}|リンク式。指定の仕方にもよるが、パスを補完してくれる※2||

※1：SpEL式
- publicなフィールドやメソッドなら直接参照が可能
- getXXX()メソッドの場合、プロパティとしてobj.XXXでアクセス可能
- フィールドなら、obj['フィールド名']でもアクセス可能

※2：リンク式
- @{foo/bar}：「foo/bar」が生成される。あまり使わない？
- @{/foo/bar}：「/コンテキストパス/foo/bar」となる。コンテキストパスが保管される。
- @{/foo/bar(param='value1', param2=${obj.value})}：クエリパラメータを指定できる
- @{/foo/{pathVal}/bar}：パスに変数を埋め込める

# 暗黙参照オブジェクト
- ${#httpServletRequest}：HttpServletRequestオブジェクト
- ${#httpSession}：HttpSessionオブジェクト
- ${param}：リクエストパラメータ
- ${session}：sessionの属性にアクセスできる。
- ${application}：ServletContextの属性を指定できる。${application['attribute_name']} 



# テンプレートの指定方法
以下の方法がある。  
また、埋め込みのパスをtiles.xmlに記述しまとめて管理することもできる。
- templateName  
  パーツとなるファイル名の事。テンプレートディレクトリ群からのパス。  
  (templatesフォルダ配下以降のパス)
  ```
  ex. layout/header
  ```
- templateName::domSelector
  
- templateName::fragmentName
  fragment属性の値(名前)を指定。  

### th:include  
  includeの場合は、子要素として追加される。
```html
<!--parts.html -->
<div id="children">
　埋め込むファイル
</div>
```
```html
<!--include.html-->
<div id="parent" th:include="part">
    parts.htmlをここに埋め込む
</div>
```
```html
<!--埋め込み結果(レンダリング後)-->
<div id="parent">
    <div id="children">
        埋め込むファイル    
    </div>
</div>
```

### th-replace
 replaceの場合は、要素が入れ替わる。
```html
<!--parts.html -->
<div id="children">
　埋め込むファイル
</div>
```
```html
<!--include.html-->
<div id="parent" th:replace="part">
    
</div>
```
```html
<!--埋め込み結果(レンダリング後)-->
<div id="children">
    埋め込むファイル    
</div>
```
### th:fragment
htmlに複数のパーツを定義する場合に使う。fragument属性を付与した要素を他ファイルに埋め込めるようになる。
```html
<!--parts.html(パーツ定義)-->
<div id="fragment1" th:fragment="template1">
    <span>teplate1</span>
</div>
<div id="fragment2" th:fragment="template2(text)">
    引数も渡せます
    <span th:text="${text}"></span>
</div>
```
- includeでfragment埋め込み
  includeの要素は残り、fragmentの中身が渡される。th:ifで埋め込むか条件判断ができる。
```html
<div id="include" th:include="parts::template1">
    <span>置き換わる</span>
</div>
```
```html
<div id="include">
    <span>template1</span>
</div>
```
- replaceでfragment埋め込み
　replace要素ごと、fragument要素に置き換えられる。th:replace側のidやifは評価されない。
```html
<div id="replace" th:replace="fragment::template1"></div>
```
```html
<div id="fragment">
    <span>template1</span>
</div>
```

### tiles.xml(抜粋)
紛らわしいけど、慣れれば管理が一括でできて楽そう。
```xml
<tiles-difinitions>
    <!--コントローラが"home/*"をviewリゾルバに渡した時、layout/header.htmlが呼ばれることを表す。-->
    <definition name="home/*" template="layout/header">
		<!--header.htmlの"common-header-1"はhome-header-1/*のdifinitionを埋め込むことを表す。-->
        <put-attribute name="common-header-1" value="home-header-1/{1}" />
	</definition>
    
    <!--home-header-1/*を呼んだところに、home.htmlのheaderフラグメントを埋め込むことを示す。-->
    <difinition name="home-header-1/*" template="home::header"/>
</tiles-difinitions>

```


