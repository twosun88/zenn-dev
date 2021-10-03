---
title: "Reactでdocumentをクリックしたら何かする処理を作ってカスタムフックにする"
emoji: "🐕"
type: "tech"
topics: [React, JavaScript]
published: true
---
こんにちは。最近Reactでいろいろやるのに夢中の [twosun](https://twitter.com/twosun88) です。

さっそく本題に入ります。

Reactで開発してる時にモーダルやポップアップ、アコーディオンなどを開いた後に **「指定の要素以外をクリックしたら閉じるようにしたい！」** そんな経験ありませんか？

ちょうど今作っている個人サービスで上記のような処理が必要になってきましたので今回記事にしたいと思います。

-----

### 動きを確認したい方はこちら（Codesandbox） ###
下記から動きを確認できます。
https://codesandbox.io/s/react-is-click-on-document-f6lym?file=/src/App.js

-----

### 仕様 ###
1. ボタンをクリックしたら対象のコンポーネントがアクティブになる（モーダルやポップアップ、アコーディオンなどが開く）
2. 指定の要素以外をクリックしたら閉じる
3. 複数の箇所で使用する可能性もあるのでコンポーネントを分割しておく

### 見た目作成 ###
まずはとりあえず状態なし（stateなし）で見た目を作成します。
今回はフェードイン・アウトして表示するポップアップを作っていきます。
※CSSは本記事では割愛しています（Codesandboxの方でご確認ください）

```js:App.js
import "./styles.css";

export default function App() {

  return (
    <div id="app">
      <div className="container">
        <button className="button">Toggle Button</button>
        <div className="modal">
          <div className="modal__inner">
            <p>Contents</p>
          </div>
        </div>
      </div>
    </div>
  );
}
```
![](https://storage.googleapis.com/zenn-user-upload/fc9c7113e1b16f7fc5f41a42.png)
灰色で囲まれた部分が表示したり消えたりする対象のポップアップのコンポーネントです。
**「赤い部分（.modal__inner）以外の箇所をクリックすると非表示にする」** 動きを想定しています。

### State追加 ###
次に **"表示・非表の切り替え用に必要なstateを追加しクリックでstateの値を変更し表示する"** 処理を追加していきます。

今回は**stateの変動に合わせてdata-show属性を変更し、CSSでフェードイン・アウト**させます。

::: message
追加・変更があった行のみハイライトしています
:::

```diff js:App.js
import './styles.css';
+ import { useState } from 'react';

export default function App() {

+  const [ isClickOnDocument, setIsClickOnDocument ] = useState( false )

+  const handleClickToggle = () => setIsClickOnDocument( true )

  return (
    <div id="app">
      <div className="container">
+        <button className="button" onClick={ handleClickToggle }>Toggle Button</button>
+        <div className="modal" data-show={ isClickOnDocument ? 'show' : 'hidden' }>
          <div className="modal__inner">
            <p>Contents</p>
          </div>
        </div>
      </div>
    </div>
  );
}
```
これで`.button`をクリックしたら`isClickOnDocument`が`true`になり、`.modal`の`data-show="show"`に変更され表示されます。

ですが、このままではクリック後に`isClickOnDocument`が`true`のままなので、再度非表示にすることができません。

### 対象の要素以外をクリックしたら非表示にする ###
表示しっぱなしはいけませんので、対象の要素以外（赤い箇所以外）をクリックした時に非表示になる処理を追加していきます。
`document`をクリックした時に、クリックされた箇所が **対象の要素の内側か外側か？** を判断して処理を分けていきます。

::: message
documentとは？
[https://developer.mozilla.org/ja/docs/Web/API/Document](https://developer.mozilla.org/ja/docs/Web/API/Document)
:::
```diff js:App.js
import './styles.css';
+ import { useEffect, useState, useRef } from 'react';

export default function App() {

  const [ isClickOnDocument, setIsClickOnDocument ] = useState( false )

  const handleClickToggle = () => setIsClickOnDocument( true )

+  const refEle = useRef( null )

+  const handleClickDocument = useRef( null )

+  useEffect( () => {
+    handleClickDocument.current = ( e ) => {
+      if ( !refEle.current.contains( e.target ) ) {
+        setIsClickOnDocument( false )
+        document.removeEventListener( 'click', handleClickDocument.current )
+      }
+    }
+  }, [] )

+  useEffect( () => {
+    isClickOnDocument && document.addEventListener( 'click', handleClickDocument.current )
+  }, [ isClickOnDocument ] )

  return (
    <div id="app">
      <div className="container">
        <button className="button" onClick={ handleClickToggle }>Toggle Button</button>
        <div className="modal" data-show={ isClickOnDocument ? 'show' : 'hidden' }>
+          <div className="modal__inner" ref={ refEle }>
            <p>Contents</p>
          </div>
        </div>
      </div>
    </div>
  );
}
```
主な処理の流れは下記の通り。
1. `useRef`で要素`.modal__inner`を予め参照できる状態にしておく。
2. ボタンクリック→`isClickOnDocument`が`true`に変更され、表示（フェードイン）
3. document（対象外）クリック→`isClickOnDocument`が`true`の場合に`document.addEventListener( 'click', handleClickDocument.current )`発火する
4. クリックされた箇所が`.modal__inner`の内部なのかを`if ( !refEle.current.contains( e.target ) ) `で判断し処理を分ける。
5. クリックされた箇所が`.modal__inner`の外部なら`isClickOnDocument`を`false`にし非表示（フェードアウト）にする。
6. 最後に`document.removeEventListener`にてイベントを削除

::: message
documentにイベントを設定することによりどの要素がクリックされたのか取得が可能です。
:::

一度仕組みがわかるとそれほど難しいことではありませんね。
もしも、**「ここをクリックした場合も非表示にしたくない」** など対象の要素が増えた場合は、その要素に`ref={ refEle }`を追加しましょう。
そうすることでその要素をクリックしても何も処理をしないようになります。

表題の **"Reactでdocumentをクリックしたら何かする処理"** の部分はこれで完了ですが、これだと使い回しができません。

モーダルやアコーディオンなどを使用しているコンポーネントが1つなら特に問題ありませんが、複数のコンポーネントで上記と同じ処理をしなければいけないとなった時に、いちいち同じ処理を個別に書くのは面倒です。

どのコンポーネントからでも使えるようにカスタムフックにしてみましょう。
::: message
カスタムフックとは？
[https://ja.reactjs.org/docs/hooks-custom.html](https://ja.reactjs.org/docs/hooks-custom.html)
:::

### カスタムフックにする ###
カスタムフックにするため別ファイルを作成します（ここでは、useIsClickOnDocument.js）

::: message alert
カスタムフック名はuseから始まる必要があります。
:::

``` js: useIsClickOnDocument.js
import { useEffect, useRef, useState } from "react";

export const useIsClickOnDocument = ( useref ) => {

  const [ isClickOnDocument, setIsClickOnDocument ] = useState( false )

  const handleClickDocument = useRef( null )

  useEffect(() => {

    handleClickDocument.current = ( e ) => {
      if ( !useref.current.contains( e.target ) ) {
        setIsClickOnDocument( false )
        document.removeEventListener( 'click', handleClickDocument.current )
      }
    }

  }, [ useref ] )

  useEffect( () => {
    isClickOnDocument && document.addEventListener( 'click', handleClickDocument.current )
  }, [ isClickOnDocument ] )

  return [ isClickOnDocument, setIsClickOnDocument ]
}
```

`App.js`から必要な部分だけをコピペして`useIsClickOnDocument.js`を作成しました。
処理の仕組みは変わりありません。

このカスタムフックが返すのは、
- `isClickOnDocument`： true or falseの状態
- `setIsClickOnDocument`： isClickOnDocumentを変更するための関数

この2つをリターンします。
また、他のコンポーネントでも使いまわせるように**クリック対象外の要素（refが指定されている要素）は親コンポーネントから受け取る**ことにしています。

このカスタムフックをApp.jsに組み込んでいきます。
```diff js:App.js
import './styles.css';
+ import { useRef } from 'react';

+ // Hooks
+ import { useIsClickOnDocument } from "./useIsClickOnDocument";

export default function App() {

  const handleClickToggle = () => setIsClickOnDocument( true )

  const refEle = useRef( null )

+  const [ isClickOnDocument, setIsClickOnDocument ] = useIsClickOnDocument( refEle )

  return (
    <div id="app">
      <div className="container">
        <button className="button" onClick={ handleClickToggle }>Toggle Button</button>
        <div className="modal" data-show={ isClickOnDocument ? 'show' : 'hidden' }>
          <div className="modal__inner" ref={ refEle }>
            <p>Contents</p>
          </div>
        </div>
      </div>
    </div>
  )
}
```
かなりスッキリしました。
`App.js`では、ボタンをクリックした時のイベントハンドラーの定義と対象内要素をuseRefで指定するのみで、他の処理はカスタムフックで行っています。

カスタムフックを利用することで、コードの見通しがよりくなり他のコンポーネントでも同じ処理を使いまわせるのは開発するにあたって大きな利点ですね。

### コンポーネントを分割する ###
最後に、今のままだと`App.js`にHTMLを直がきしているため若干見にくいので、必要な部分を別コンポーネントとして独立させておきます（ここではPopUp.js）

そして、コンポーネントを複数設置して、動作に問題ないか確認してみましょう。

※今回は色情報のみ親コンポーネントから受け取っています。

```js: components/PopUp.js

import { useRef } from 'react'

// Hooks
import { useIsClickOnDocument } from '../useIsClickOnDocument.js'

const PopUp = ({ color }) => {

  const handleClickToggle = () => setIsClickOnDocument( true )

  const refEle = useRef(null);

  const [ isClickOnDocument, setIsClickOnDocument ] = useIsClickOnDocument( refEle )

  return (
    <>
      <button className="button" onClick={ handleClickToggle }>
        { color } Toggle Button
      </button>
      <div className="modal" data-show={ isClickOnDocument ? 'show' : 'hidden' }>
        <div className={ `modal__inner ${ color }` } ref={ refEle }>
          <p>{ color }</p>
        </div>
      </div>
    </>
  )
}

export default PopUp
```
```diff js: App.js
// Style
import "./styles.css";

// Components
+ import PopUp from "./components/PopUp.js"

export default function App() {
  return (
    <div id="app">
      <div className="container">
+        <PopUp color="Red" />
+        <PopUp color="Blue" />
      </div>
    </div>
  )
}
```
カスタムフックの読み込みをコンポーネント`PopUp`の方に移行しました。
`App.js`側はコンポーネントを読み込み`color`だけpropsで渡しています。


赤と青のポップアップを閉じたり開いたりして動作を確認してみてください。
それぞれが独立した動きをしていればOKです。

今回はポップアップを例にあげましたが、他にも **"モーダルを開いた時に外側をクリックしたらモーダル閉じる"** などに応用できる思います。

本記事は以上になります。
最後までお読みいただきありがとうございました。
