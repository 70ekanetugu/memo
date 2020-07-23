# React
# 目次
- [概要](#概要)
- [基礎](#基礎)
- []()

# 概要
詳細は、[React Docs](https://ja.reactjs.org/docs/getting-started.html)。  
Reactの導入は大きく分けて以下の通り
- scriptタグへCDNリンク追加
    - 既存アプリへの部分的な導入など
- create-react-appの使用
    - 新しいSPA作成時など
- Next.jsの使用
    - 静的サイトやサーバサイドレンダリングされるアプリ
    - ルーティングのソリューションを含み、サーバサイドとしてNode.jsを想定している。
- Gatsbyの使用
    - 静的なサイトに最適

# 基礎
## 1. JSX
javascriptの拡張構文。  
Reactでは、マークアップとロジックを分けず、両方を含んだ`コンポーネント`単位で分割する。  
- `{}`を記述すると、中にjs式を埋め込める
- JSXはインジェクション攻撃を防ぐ
- JSXはBabelにより、`React.createElement()`で"React要素"(オブジェクト)にコンパイルされる。

## 2.要素のレンダー
ルートDOMノードに対して以下を実行することで、React要素をレンダリングできる。  
基本的には、ルートDOMノードは1つであるが、部分的な導入の場合複数ある事もある。
```js
//jsファイル
ReactDOM.render(
    element,//React要素を格納したjs変数
    document.getElementById('root')//React要素を埋め込むHTML要素
);
```

## 3.コンポーネント
Reactで作成する構成の最小単位。  
コンポーネント属性には、何でも指定できる(文字列・数値・関数・オブジェクトetc...)
渡された属性は、コンポーネント内で`props`から参照することが出来る。(読み取り専用)  


Reactにおけるコンポーネントの表現は以下の2通り。
- 関数コンポーネント  
関数コンポーネントで状態管理する場合はhook機能を利用する必要がある。  
`react`モジュールの`useState(初期値)`を使用する事で、stateを定義できる


```js
function Table(props) {
    //hookによるstateの利用。
    //↓は、stateとしてcountという読み取り用変数と、settterを定義する。
    const [count, setCount] = useState(0);//初期値「0」

    function handleClick() {
        //...
    }

    //return内に、レンダリングするReact要素を渡す。
    //必ず単一の親要素を定義する必要がある。
    return (
        <table>
            <th>テーブルヘッダ</th>
            <td onClick={handleClick()}>データ</td>
        </table>
    );
}

```

- クラスコンポーネント  
クラスコンポーネントで状態を管理する場合、this.stateを利用する。  
更新する場合は`setState({キー名: 値})`を利用する。

```js
class Table extends React.Component {
    constructor(props) {
        super(this);//定型文
        //コンポーネントで管理するstateを定義及び初期化する
        this.state = {
            value1 : null,
            value2 : 99,
            obj1 : {
                ...
            }
        };
        //メソッドにthis(Tableコンポーネントをバインド)
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        //...
    }

    //return内に、レンダリングするReact要素を渡す。
    //必ず単一の親要素を定義する必要がある。
    render() {
        return (
            <table>
            <th>{this.props.header}</th>
            <td onClick={handleClick()}>{this.state.value2}</td>
            </table>
        );
    }
}
```