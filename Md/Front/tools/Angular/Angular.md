# Angular
# 目次
- [概要](#概要)
- [基礎](#基礎)

# 概要
AngularはHTMLとTypescriptでSPAを開発するプラットフォーム兼フレームワーク。  
Angularを利用する上で理解すべき概念は以下の通り。

- モジュール
    - `NgModule`がAngularの基本構成要素
- コンポーネント
    - 全てのアプリにおいて、必ず1つのルートコンポーネントが存在し、DOMに接続される
    - `@Component()`デコレータの下のクラスをコンポーネントとして認識し、そのクラスにデータとロジックを定義する。  
    - ビューを定義するHTMLテンプレートに関連付けられる
    - サービスをDIできる
    - 上記によりコードをモジュール化し、再利用可能にする
- ビュー
    - コンポーネントを元にAngularが選択し変更できる画面要素の集まり
    - つまりコンポーネントのクラスとテンプレートが一緒になってビューとなる
- サービス
    - ビューに直接関係しない特定の機能(コンポーネント間で共有したい機能など)を提供するもの
    - `@Injectable()`デコレータを付けるとサービスクラスとして扱われる
    - サービスプロバイダーはコンポーネントにDIすることが出来る
    - サーバーアクセスやロギング等に使われるのが一般的
- ディレクティブ
    - Angularが提供するプログラムロジック
    - `*ngIf`や`*ngFor`などがある
- データバインディング
    - ビューとコンポーネント間でデータを紐付けることで単方向と双方向がある
    - イベントバインディングと、プロパティバインディングがある  


# 基礎
### 1.コンポーネント&テンプレート
- テンプレートシンタックス
    - `{{コンポーネントプロパティ}}`　：補間。クラスのプロパティをバインディング
    - `[DOMプロパティ]="コンポーネントプロパティ"`　：クラスのプロパティをDOMプロパティに紐づける
    - `[class.クラス名]="論理式"`　：論理式がtrueならclassを付ける
    - `[style.スタイルプロパティ]="論理式"`　：class同様
    - `(イベント名)="js式"`　：DOMのイベントに対するトリガー指定
    - `[(title)]="name"`　：双方向バインディング。`[title]="name" (titleChange)="name=$event"`と等価
    - `#変数名` ：テンプレート参照変数。#変数を定義した要素を参照できるようになる


- ディレクティブ　※ビルトイン
    - `*ngIf`　：論理式を""で囲む
    - `*ngFor`　：反復可能要素をテンプレート入力変数に展開する
    - `[ngSwitch]`　：条件式を記述し、子要素のCaseで判定を行う
    - `[ngSwitchCase]`　：Switchディレクティブの一致条件
    - `[ngSwitchDefault]`　：Switchディレクティブのデフォルト
    - `[ngClass]`　：オブジェクトまたは関数でCSSを指定
    - `[ngStyle]`　：同上のstyleバージョン
    - `[(ngModel)]`　：フォームコントロールの双方向バインディング、パース、バリデーション

- クラスデコレータ
    - 以下いずれもクラスの上に記述する
    - `@Component({})`　：コンポーネントであることを宣言し、メタデータを提供する
    - `@Directive({})`　：ディレクティブである事を宣言し、メタデータを提供する
    - `@Pipe({})`　：パイプである事を宣言し、メタデータを提供する
    - `@Injectable()`　：他のクラスから提供・注入する為の宣言。(DIコンテキストへの登録)

- クラスフィールドデコレータ
    - `@Input()`　：
    - `@Output()`　：
    - `@HostBinding`　：
    - `@HostListener`　：


```javascript
import {Component} from '@angular/core'

@Component({
    selector: 'root',
    template: `
        <h1 [id]="id">{{name}}</h1>
        <button (click)="onClick($event)">ボタン</button>
        <ul *ngIf="isClick">
            <li *ngFor="let list of lists">{{list}}</li>
        </ul>
    `
})
export class AppComponent {
    this.lists = ['Chrome', 'Edge', 'Safari'];
    this.isClick = false;

    constructor() {
        public id: number,
        public name: string,
    }

    onClick(e) {
        this.isClick = true;
        console.log('event: ' + e);
        alert('Click Button!');
    }
}
```