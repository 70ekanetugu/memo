# レイアウト
# 目次
  - [1.デザイン原則](#1.デザイン原則)
  - [2.レイアウト](#2.レイアウト)
  - [3.配色](#3.配色)

# 1. デザイン原則
## Webデザイン法則
- 視覚階層
- 黄金律　：巻貝みたいなやつ
- ヒックの法則：選択肢が多いほど意思決定までの時間が長くなる
- フィットの法則　：大きく近いほど、使いやすい
- 三分割法　：縦横に均等に2本の線を引いたときの交点に被写体を置く手法
- ゲシュタルトの法則
  1. 近接
     - 関連する項目は近づける。
     - ex.画像とテキストは近づけるなど
  2. 整列
     - ページ上の配置を統一する
     - ex.左端をそろえるなど 
  3. 反復
     - 同じ種類のコンテンツは同じパターンで繰り返す
     - ex.youtubeの動画一覧みたいな 
  4. コントラスト
     - コンテンツにメリハリを持たせる
     - ex.「大きい文字-小さい文字」、「大きな画像-小さな画像」、「寒色-暖色」など 
- 余白　：余白は印象を大きく変える。微調整でも怠らない
- オッカムの剃刀　：シンプルisベスト



# 2. レイアウト
## 1カラム
- 特徴
  サイドバーのないレイアウト。  
  企業のランディングページなどに向いている。  
- メリット
  - メインコンテンツに関心を向けられる
  - サイドバーが無いため、レスポンシブでの構成が簡単(PC,スマホ対応が楽)
- デメリット
  - サイドバーが無いため、ページ誘導が不便

<details>
    <summary>1カラム例</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"></meta>
    <title>サンプル</title>
    
<!--ここから↓差し替え-->

    <style>
        body, h1, p {
            margin: 0;
            padding: 0;
            overflow-x: hidden;
        }
        .one-column-layout{
            position:relative;
            background: rgb(255, 255, 255)  url(./pic/harley-davidson-2S4FDh3AtGw-unsplash.jpg);
            background-size: cover;
            background-repeat: no-repeat;
            background-position: center;
            background-attachment: fixed;
            width: 100vw;
            height: 100%;
        }
        
        .header{
            position: fixed;
            background-color: rgba(0,0,0,0.8);
            box-sizing: border-box;
            border-radius: 10px;
            top: 0;
            left: 0;
            width: 100vw;
            height: 150px;
            padding: 25px 0;
        }
        .flex-container {
            display:flex;
            flex-wrap: nowrap;
            justify-content: space-around;

        }
        ion-icon {
            color: dimgrey;
            font-size: 100px;
        }
        ion-icon:hover {
            color: white;
            filter: drop-shadow(0 3px 10px lightgoldenrodyellow)
        }
        section {
            font-family:serif;
            padding: 350px 5% 0;
            overflow-x: hidden;
        }
        article.phrase{
            color: mintcream;
            font-style:oblique;
            letter-spacing: 1%;
            font-size: 300px;
        }

        .grid-container{
            display:grid;
            margin-top: 10vh;
            grid-template-columns: repeat(4, 25%);
            grid-template-rows: repeat(2, 25vh);
            row-gap: 40px;;
            grid-template-areas: 
                "a b b b"
                "c b b b";
        }
        .grid-item1 {
            grid-area: a;
            background-color: rgba(0,0,0,0.7);
            border-radius: 10px;
        }
        .grid-item2 {
            grid-area: c;
            background-color: rgba(0,0,0,0.7);
            border-radius: 10px;
        }
        article h1{
            color: lightgoldenrodyellow;
            font-size: 75px;
            border-left: solid 30px lightgoldenrodyellow;
            border-bottom: solid 1px lightgoldenrodyellow;
            margin: 20px;
            padding-left: 0.5em;
        }
        .main {
            box-sizing: border-box;
            width: 90%;
            height: 75vh;
            margin: 5% 5% 0 5%;
            background-color: rgba(0,0,0,0.7);
            border-radius: 10px;
        }
        .main article h1{
            font-size: 130px;
        }
        </style>

<!--▲差し替え-->
</head>
<body>
    <div class="one-column-layout">
        <div class="header">
            <div class="flex-container">
                <ion-icon name="menu-outline"></ion-icon>
                <ion-icon name="home-outline"></ion-icon>
                <ion-icon name="cart-outline"></ion-icon>
                <ion-icon name="person-outline"></ion-icon>
            </div>
        </div>
        <section>
            <article class="phrase">
                <p>自分探しの旅へ. . .</p>
            </article>
        </section>
        <section class="grid-container">
            <article class="grid-item1">
                <h1>Information</h1>
                <p></p>
            </article>
            <article class="grid-item2">
                <h1>New Release</h1>
                <p></p>
            </article>
        </section>
        <section class="main">
            <article>
                <h1>見出し</h1>
                <p></p>
            </article>
        </section>
    
    </div>
    
    <script src="https://unpkg.com/ionicons@5.0.0/dist/ionicons.js"></script>
</body>
</html>
```
</details>

## 2カラム
- 特徴
  今も昔も定番のレイアウト。  
  ECサイトなどのサイト回遊率を高めるサイトは左にサイドバー。  
  ニュースやブログなどのサイトは右側にサイドバーを置くのが主流。  
- メリット
  - サイドバーを設置できる為、他のコンテンツへの誘導がしやすい
- デメリット
  - メインコンテンツへの関心が分散してしまう

## 3カラム
- 特徴
    両サイドにサイドバーがある為、情報量が増える。  
    サイドバーの誘導が2カラムに比べよりしやすくなる。
- メリット
  - 2カラムにくらべ情報量が増える
  - 誘導がしやすい
- デメリット
  - メインコンテンツが狭くなる
  - サイドバーで情報が増えるが場合によっては扱いづらくなる
  
## カード型
- 特徴
    カードを複数並べて構成したレイアウト。  
    各カードの中に、画像及びテキストを配置し、クリックによってモーダル表示やページ遷移を行う。
- メリット
  - 1ページで多くのコンテンツを見せる事が出来る。
  - 自由な配置が可能
- デメリット
  - 配置によっては視線の動きが一方向出なくなるため、見づらいくなる可能性
  - カードの大小により関心に偏りが生じる

## サイドバーまたはヘッダー固定
- 特徴
    基本は、1カラムや2カラムのヘッダーやサイドバーが固定かされたもの。
- メリット
  - ヘッダー、サイドバーが固定される為、ユーザビリティは高い
- デメリット
  - 表示領域を占有する為、情報量が少なくなる
  - スマホの場合、ストレスとなる  

# 3. 色
## 色相
赤、緑、青などの色味の違い。
この色の中からピックアップし、円にして並べたものを色相環という。  
色相環では隣り合う色を類似色(近似色)、反対側の色を補色、補色の両隣の色を反対色という。
- 近似色
- 補色
- 反対色

## 彩度
色の鮮やかさ(=濁り具合)の事。  
彩度の中で最も鮮やかな色(濁りが無い色)を純色、反対に彩度が色を無彩色(グレー)という。

## 明度
色の明るさの事。  
明度が高くなると白に近づき、低くなると黒に近づく。

## 配色
配色は、一般的に色相(近似、反対、補色)とその面積比率で感覚が変わる。
例えば、補色である赤と緑を組み合わせる時、面積比率が1:1の場合と1:5の場合で印象は変わる。  
コントラストが強い為、面積比率が同じ場合は両者の比較に用いることが出来、1:5の様なアンバランスな場合は比率が大きい側をメインとしてはっきり分ける事が出来る。
逆に近似色で構成した場合は、

以下の3つの色を決めると良く、以下の比率がおすすめらしい
- ベースカラー：70%
- メインカラー：25%
- アクセントカラー：5%

> ベースカラー  
  
背景や余白など、サイトで最も大きな面積を占める色

> メインカラー  

サイトの印象を決める色。最も重要なイメージカラー

> アクセントカラー  

ベース、メインカラーに対して、目が引くような色の事。  
一般的にボタンなどに使われる色


