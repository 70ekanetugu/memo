# SQLアンチパターン
DB、クエリ―設計におけるアンチパターンと、その改善策をまとめる。

# 目次
 [概要](#概要)
 [論理設計](#論理設計)
 [物理設計](#物理設計)
 [クエリ―](#クエリ)
 [アプリケーション開発](#アプリケーション開発)

# 概要  
DB取り扱いにおける基本的なアンチパターンを纏める。
各セクションで目的別にアンチパターンと解決策を記載する。
扱うセクションは以下の通り。

1. 論理設計
    - エンティティの関連・正規化に関する内容
2. 物理設計
    - 論理設計を元にした実際のテーブル定義に関わる内容
3. クエリー
    - SQLに関する内容
4. アプリケーション開発

### 基礎知識
- 正規化
  1. 第一正規化
  2. 第二正規化
  3. 第三正規化
  4. ボイスコッド正規化
  5. 第四正規化

- レプリケーション
- シャーディング
- 主キー
  - 主キーはそれ自身がデータにもなるし、メタ的な疑似データ(疑似キー)にもなり得る。
  - 慣習だからと安易に主キーをid(疑似キー)とするのはNG。  
- 外部キー
  - 外部キーによる参照制約の役割は想像以上に大きい。
  - パフォーマンス影響を考えて外部キーを付けないのは、その分参照制約をコードでカバーする必要がある為、推奨しない。※そもそもパフォーマンス影響は無いに等しいし、コードで参照制約をカバーするほうがよっぽどパフォーマンス悪い。
  - 参照制約による更新・削除はカスケードで対応できる。`ON UPDATE CASCADE`や`ON DELETE RESTRICT`,`ON DELETE SET DEFAULT`など


# 論理設計
1. 属性に複数の値がある
    - アンチパターン　：カンマ区切りのフォーマットリスト
        データ型をVARCHAR(100)やTEXTにし、`data1, data2, data3...`のようなカンマ区切りでデータを格納するパターン。つまりは非正規化状態。(データがjson等でも同じ)  
        これには以下の問題がある。
        - 検索時にパターンマッチが必要
        - 結合(JOIN)条件指定する場合に非効率
        - 集約クエリ作成が非効率※集約クエリは複数行に対する使用で設計されている
        - 更新処理が非効率
        - 無効入力のチェックはアプリに依存せざるを得ない
        - 区切り文字の選択問題
        - リストの長さ制限  
   - 解決策
      - そもそもリストの各要素にアクセスが不要な場合は、採用できる。(ログなど?)
      - リストが入るような属性を新規テーブルで扱い、交差テーブルを作成する。  

```sql
/*
 *アンチパターン
 * ・official_positionの値を、検索条件(パターンマッチ)や結合条件に使用する可能性が無いなら有効な方法。
 */
CREATE TABLE Employee (
    employee_id SERIAL,
    name VARCHAR(100),
    official_position TEXT -- ex：'Director,Quality manager,union member'
    PRIMARY KEY (employee_id)
)
```  
   
```sql
/*==========================================
 *解決策：交差テーブル使用
 */
CREATE TABLE OfficialPosition (
    official_position_id SERIAL,
    position_name VARCHAR(100)
    PRIMARY KEY (official_position_id)
);
CREATE TABLE Intersection( -- 交差テーブル
    employee_id BIGINT NOT NULL,
    official_position_id BIGINT NOT NULL
    FOREIGN KEY (employee_id) REFERENCES Employee(employee_id),
    FOREIGN KEY (official_position_id) REFERENCES OfficialPosition(official_position_id)
);
CREATE TABLE Employee (
    employee_id SERIAL,
    name VARCHAR(100),
    official_position_id BIGINT,
    PRIMARY KEY (employee_id),
    FOREIGN KEY (official_position_id) REFERENCES Intersection(official_position_id)
);
```  

2. 階層構造を持つデータ設計
    - アンチパターン　：親ID属性を付加する
      テーブルにparent_idを持たせた `隣接リスト設計` は、簡単に階層構造を表現できるがいくつか問題がある。
        - 親と子の2階層に対するクエリ―は簡単だが、階層が深くなるほどJOIN句が増加する。
        - 階層の深さに制限が無い場合、動的なクエリー作成が困難
        - 削除処理が困難。※サブツリー以下を全て削除する場合、全ノード検索後に子要素から順次削除する必要がある。
    - 解決策
      - 階層の深さが浅く挿入のみの場合は採用できる。親子の取得、新規挿入は簡単に行えるので有効な設計といえる。
      - 代替ツリーモデルの使用
        1. `経路列挙モデル`
           - パスのような属性を追加し、階層構造を表現する
        2. `入れ子モデル`
           - 各ノードに子ノードに関する情報を持たせる。値は深さ優先探索で割り当て。
        3. `閉包テーブルモデル`
            - 階層構造を表現するテーブルを新規に作成し、これに親と子の情報を格納する。  

```sql
/*
* アンチパターン
* ex.コメントに階層構造がある場合(コメントへの返信で階層)
*/
CREATE TABLE Comment(
    comment_id SERIAL,
    parent_comment_id BIGINT,
    comment VARCHAR(1000)
    PRIMARY KEY (comment_id)
)
```  
アンチパターンのデータ例
|comment_id|parent_comment_id|備考|
|:--|:--|:--|
|1|-|-|
|2|1|-|
|3|1|-|
|4|2|-|
|5|3|-|  

```sql
/*
*経路列挙モデル
*[メリット]
*・パターンマッチで、先祖、子孫が簡単に取得できる。
* ex. 「WHERE '1/2/3/4' LIKE path || '%';」　←先祖が1/2/3/%,1/2/%,1/%と一致
*
*[デメリット]
*・以下の様な前述1項の設計と同じ問題がある。
*・パターンマッチングが必要になる為、パフォーマンスが落ちる。
*・パス値の正当性はアプリ側で担保しなければならない。
*・VARCHAR等にした場合、ﾊﾟｽの制限が生まれる
*/
CREATE TABLE Comment (
    comment_id SERIAL,
    path VARCHAR(100),
    comment TEXT
    PRIMARY KEY (comment_id)
)
```  
|comment_id|path|備考|
|:--|:--|:--|
|1|1/|-|
|2|1/2/|-|
|3|1/2/3/|-|
|4|1/4/|-|
|5|1/4/5/|-|
|6|1/4/6/|-|
|7|1/4/6/7/|-|  

```sql
/*
*入れ子モデル
*[メリット]
*・非葉ノードを削除しても、子ノードは削除ノードの親の直接の子と見なされる。
*　階層構造の再編が不要。
*
*[デメリット]
*・クエリが複雑になる。
*/
CREATE TABLE Comment (
    comment_id SERIAL,
    nsleft integer,
    nsright integer,
    comment TEXT
    PRIMARY KEY (comment_id)
)
``` 
|comment_id|nsleft|nsright|備考|
|:--|:--|:--|:--|
|1|1|14||
|2|2|5||
|3|3|4||
|4|6|13||
|5|7|8||
|6|9|12||
|7|10|11||   

```sql
/*
*閉包テーブルモデル
*[メリット]
*・代替ツリーモデルの中では最も汎用的。Webではこのモデルが安牌？
*  データと、階層構造が別テーブルなので扱いやすい
*
*[デメリット]
*・単純にテーブルが増え、階層構造が深くなると行数も多くなるため、ストレージを圧迫する。組込みなどハードリソースの制限が厳しい場合には向かない。
*/
CREATE TABLE Comment (
    comment_id SERIAL,
    comment TEXT
    PRIMARY KEY (comment_id)
);
CREATE TABLE CommentPath (
    ancestor BIGINT NOT NULL,
    descendant BIGINT NOT NULL
    PRIMARY KEY(ancestor, descendant),
    FOREIGN KEY(ancestor) REFERENCES Comment(ancestor),
    FOREIGN KEY(descendant) REFERENCES Comment(descendant)
);
``` 
|先祖|子孫|備考|
|:--|:--|:--| 
|1|1||
|1|2||
|1|3||
|1|4||
|1|5||
|1|6||
|2|2||
|2|3||
|3|3||
|4|4||
|4|5||
|4|6||
|5|5||
|6|6||

3. 可変属性をサポートする場合(属性のタイプに応じて内容が変わるが、共通属性がある場合)
    - アンチパターン　： 汎用的な属性テーブル使用(EAV)
      - `attr_name`,`attr_value`の様な汎用的な属性を持つテーブルとするkey-value形式の方法。(EAV、オープンスキーマ、スキーマレス、名前/値ペアなどと呼ばれる。)
      - データ型が文字列になる為、無効データのチェックはアプリコードに依存。
      - 属性ごとの必須制約が設定できない。
      - 参照整合性を強制できない。
      - 行の再構築が必要。
      - とはいえ、以下の様なメリットもある。
        - nullだらけの行が無くなる。
        - テーブルの列数を減らせる。
        - 属性追加が簡単(DDL不要)
    - 解決策
      - RDBのメリットが失われる為、基本的に使わない。使いたい場合はNoSQLを使用すべき。
      - サブタイプモデリングを行う。
        1. シングルテーブル継承
           - 全てのタイプの属性を1つのテーブルにまとめ、タイプ属性を持たせる。
           - nullだらけのレコードが存在しうる。 
        2. 具象テーブル継承
           - タイプ毎のテーブルを作成し、それぞれにすべての属性を持たせる。
           - タイプごとの同じカラム定義が必要なため、属性追加するときは大変。 
        3. クラステーブル継承
           - オブジェクト指向の継承を模倣したもの。 
           - 具象テーブルにおける、共通属性を抜き出して共通テーブルとする方法。 
           - 共通テーブルと、具象テーブルには1対1の関連が強制される。
        4. 半構造化データ(シリアライズLOB)
           - サブタイプの数が多い場合や、頻繁に属性追加がある場合は、LOB列(TEXT,BLOBなどのLarge Object)を追加し、XMLやJSON、blob等の形式で値を格納できる。 
           - 拡張性が高い。
           - LOB列のデータは、アプリコード側でパースする必要がある。  

```sql
/*
*アンチパターン：　汎用テーブル(IssueAttributes)を使用する場合
*/
CREATE TABLE Issues (
    issue_id SERIAL PRIMARY KEY
);

CREATE TABLE IssueAttributes (
    issue_id BIGINT NOT NULL,
    attr_name VARCHAR(100) NOT NULL,
    attr_value VARCHAR(100) NOT NULL,
    PRIMARY KEY (issue_id),
    FOREIGN KEY(issue_id) REFERENCES Issues(issue_id)
);

--以下の様に汎用テーブル内にテーブル定義があるような感じ。
INSERT INTO IssueAttributes (issue_id, attr_name, attr_value) 
VALUES
    (1234, 'status', 'NEW'),
    (1234, 'description', '説明文'),
    (1234, 'version', '1.6'),
    (1234 'update_date', '2020-01-01 00:00:00');
```

```sql
/*
*シングルテーブル継承：　全ての属性を持たせるパターン。歯抜けのレコードが増える可能性。
*/
CREATE TABLE Issues (
    issue_id SERIAL PRIMARY KEY,
    status VARCHAR(100),
    description VARCHAR(100),
    version VARCHAR(10),
    update_date DATE
);
```

```sql
/*
*具象テーブル継承
*　・サブタイプ毎にテーブルを用意する。同じ属性が各テーブルに存在する。
*/　

CREATE TABLE Bugs (
    issue_id SERIAL PRIMARY KEY,
    reported_by BIGINT,
    product_id BIGINT,
    status VARCHAR(100),
    severity VARCHAR(100),--当該テーブル独自
    version_affected VARCHAR(20)--当該テーブル独自
)

CREATE TABLE FeatureRequests (
    issue_id SERIAL PRIMARY KEY,
    reported_by BIGINT,
    product_id BIGINT,
    status VARCHAR(100),
    sponsor VARCHAR(100)--当該テーブル独自の属性
)

```

```sql
/*
*クラステーブル継承
*　・共通テーブルを作成し、サブタイプは独自の属性だけ保持したテーブルにする。
*/

CREATE TABLE Issues (
    issue_id SERIAL PRIMARY KEY,
    reported_by BIGINT,
    product_id BIGINT,
    status VARCHAR(100)
)

CREATE TABLE Bugs (
    issue_id SERIAL PRIMARY KEY,
    severity VARCHAR(100),--当該テーブル独自
    version_affected VARCHAR(20)--当該テーブル独自
    FOREIGN KEY(issue_id) REFERENCES Issues(issue_id)
)

CREATE TABLE FeatureRequests (
    issue_id SERIAL PRIMARY KEY,
    sponsor VARCHAR(100)--当該テーブル独自の属性
    FOREIGN KEY(issue_id) REFERENCES Issues(issue_id)
)
```  

```sql
/*
*半構造化データ： 汎用属性としてLOB列を定義したテーブル
*　・サブタイプが多い場合や、頻繁に属性追加がある場合に適用。
*　・LOB列にはJSON,XML等の形式で格納も可能。
*　・シリアライズLOBとも呼ばれる。
*/
CREATE TABLE Issues (
    issue_id SERIAL PRIMARY KEY,
    reported_by BIGINT,
    product_id BIGINT,
    status VARCHAR(100),
    issue_type VARCHAR(100),--サブタイプの値
    attributes TEXT NOT NULL--サブタイプに応じた属性が格納される。
)
```

# 物理設計
1. 小数値を取り扱う場合(丸め誤差)
    - アンチパターン　：FLOAT型の使用
        - 数値はIEEE754形式で扱われる。(仮数部、指数部で扱う形式)
        - 上記から丸め誤差は避ける事ができない。(ex. 1/3=0.33333...≒0.333)
        - 丸目誤差の影響は積演算で顕著になる。
    - 解決策
      - 科学計算とかで無い限り、FLOAT型やDOUBLE型は使用しない。
      - Numeric、Decimal型を使用する。有理数も丸められる事がない型。
      - 精度(小数部含めた10進桁の総数)と、スケール(小数点以下の桁数)を指定できる。
      - Decimal(5,2)の場合、精度「5」、スケール「2」を示す。※Numericも同様


2. 画像ファイルなどを扱う場合
    - アンチパターン　：画像などのLOBファイルをファイルシステム上で管理する
        - DBにはファイルのパスを格納し、実ファイルはファイルシステム上で管理している場合。
        - ファイル削除時、実ファイルを削除する為のアプリコードを書く必要がある
        - 更新等のトランザクション中に、他ユーザが実ファイルを参照してしまう可能性がある。
        - 削除、更新のロールバック時、実ファイルは戻せない。
        - バックアップは、DBとファイルシステム両方が必要であり、尚且つバックアップを同期させなければならない。
    - 解決策
        - 一般的には、アンチパターンの手法が取られることが多く、間違いではない。
        - 問題としているのは、盲目的に採用する事。
        - 代替案として、BLOB型を使用してDBに管理させる。
        - トランザクション、ロールバックが問題にならない。
        - バックアップもDBのみで済む。 

3. インデックスの作成
    - アンチパターン　：インデックスを闇雲に使用
        - インデックスを使用しない
        - インデックスを多用しすぎる
        - 基本的に主キーは自動的にインデックスが作成される為、手動で作成するのは冗長。
        - VARCHAR型などで文字列長が長い場合、あまり有効ではない。
        - 条件などで指定する事が無いカラムにインデックスを作成するのは無意味
        - 複合インデックスを検索条件や結合条件、ソートで使用する場合、インデックス列の定義順に指定する必要がある。
        - LIKE句ではインデックスは機能しない。
    - 解決策
      - MENTORの原則に基づいて、効果的なインデックス管理を行う。
        1. Measure
            - プロファイリングツールやアプリへのログ仕込みなどで、アプリケーション全体のパフォーマンスを測定し、遅いSQLを特定する。
            - スロークエリーなどDBによってはクエリ―実行時間のトレース機能がある為、これらで実行速度が遅いSQLを特定することも可能。
        2. Explain
            - EXPLAINコマンドで行う。
            - クエリ実行計画(QEP)解析結果のレポートを取得する。
        3. Nominate
            - QEPを読んで、インデックスが使用されていないテーブルアクセス箇所を探し、インデックスを作成する。
            - 場合のよっては、カバーリングインデックスも採用する。※本来インデックスに不要な列を含めることで、複合インデックスに含まれる値のみの検索であれば、インデックスのみの参照で済むため、高速化できる。
        4. Test
            - 修正後、再度パフォーマンス測定を行う。
            - 結果から、修正が有効か判断し、改善の余地が残っているなら再度Nominateを行う。
        5. Optimize
            - インデックスは使用頻度が高いデータの為、キャッシュメモリに格納されやすい。
            - その為、キャッシュバッファサイズを検討する。※むやみに大きくするのはNG。DBサイズやシステムメモリなどから検討する。 
        6. Rebuild
            - VACUMコマンドで、インデックスの再構築を行う。  

# クエリ―  
1. NULLの取り扱い
   - アンチパターン ：NULLを一般値(0やfalse)として扱う or NULLを使用しない 
     - SQLの3値論理の誤解
     - 他プログラム言語の様にNULLを一般値として扱うことはできない。(NULLはNULLでしかない)
     - NULLの使用せず、unknownなど代替の値を使用すると色々不都合が生じる。
   - 解決策
     - SQLの3値論理を理解する。
     - `IS NULL`,`IS NOT NULL`,`IS DISTINCT FROM :条件`を使う。
     - NOT NULL制約の利用
     - COALESCE()関数の使用(SQL標準の関数)

```sql
--以下のSQLは等価
SELECT * FROM Sample WHERE column1 IS NULL OR column1 != 1;
SELECT * FROM Sample WHERE column1 IS DISTINCT FROM 1;

/*
*動的なデフォルト値
*　・テーブル定義でデフォルトを設定する必要が無い
*　・その為、特定のケースのみデフォルト値をセットしたい場合などに有効
*
*ex. ↓の場合、column1->column2->デフォルト値の優先度で、NULL出ない値が適用される。
*/
SELECT COALESCE(column1 , column2, 'デフォルト値') FROM Sample;
```  

2. グループ

# アプリケーション開発


