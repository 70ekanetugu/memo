# Thymeleafメモ

# 目次

# 埋め込み  
## テンプレートの指定方法
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

## th:include  
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

## th-replace
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
## th:fragment
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

## tiles.xml(抜粋)
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


