---
id: interactivity-and-dynamic-uis
title: 相互作用と動的なUI
permalink: docs/interactivity-and-dynamic-uis-ja-JP.html
prev: jsx-gotchas-ja-JP.html
next: multiple-components-ja-JP.html
---

Reactで[どうやってデータを表示するか](/react/docs/displaying-data-ja-JP.html)については既に学んでいます。これからは、どうやって相互に作用するUIを作成するかを見ていきましょう。


## 単純な例

```javascript
var LikeButton = React.createClass({
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'liked' : 'haven\'t liked';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});

ReactDOM.render(
  <LikeButton />,
  document.getElementById('example')
);
```


## イベントハンドリングとイベントの組み合わせ

Reactでは、普通のHTMLで行っているように単純にキャメルケースのpropとしてイベントハンドラを渡します。ReactはIE8と同じように全てのイベントが動作することと、組み合わされたイベントシステムが実行されることを保証します。これは、Reactが、イベントがどのように発生して、仕様にそってそれをキャプチャする方法を知っていることと、イベントはイベントハンドラに渡されて、どのブラウザを使っていても、[W3Cの仕様](http://www.w3.org/TR/DOM-Level-3-Events/)と一致することを保証することを意味します。

## 中身を見る: オートバインディングとイベントの委譲

中身を見ていくと、Reactはコードの性能と、理解しやすさを保つためにいくつかのことを行っています。

**オートバインディング:** JavaScriptでコールバックを作成するときには、普通は `this` の値が正しくなるように、インスタンスにメソッドをはっきりとバインドする必要があります。
Reactでは、全てのメソッドが自動でコンポーネントのインスタンスにバインドされます。ReactはCPUとメモリを効率的に使えるように、バインドするメソッドを持っています。これによって、タイピングの量を少なくすることもできます！

**イベントの委譲:** Reactは実際はノード自体にイベントハンドラをアタッチはしません。Reactを立ち上げる時、1つのイベントリスナを使って最上位で全てのイベントをリスニングし始めます。コンポーネントがマウントされたり、アンマウントされた時は、イベントハンドラは内部のマッピングを単純に加えたり減らしたりします。イベントが起こった時には、Reactはこのマッピングを使ってどのようにディスパッチするかを知っています。マッピングの中にイベントハンドラが無い場合は、Reactのイベントハンドラは何も行いません。なぜこれの速度が速いかについて詳しく知るためには、[David Walshの素晴らしいブログの投稿](http://davidwalsh.name/event-delegate)をご覧ください。

## コンポーネントは静的なマシーンに過ぎない

ReactはUIをただの静的なマシーンと考えます。UIがたくさんのstateの中にあり、それらのstateをレンダリングするものだと考えることで、UIの一貫性を保ちやすくなります。

Reactでは、コンポーネントのstateをアップデートするだけで、新しいstateに基づいた新しいUIをレンダリングします。Reactは最も効率的な方法でDOMをアップデートする面倒を見てくれます。


## どのようにStateが働くか

Reactがデータの変更を通知する共通な方法は `setState(data, callback)` を呼ぶことです。このメソッドは `data` を `this.state` にマージし、コンポーネントを再度レンダリングします。コンポーネントが再度レンダリングし終わったら、オプションの `callback` が呼ばれます。ほとんどの場合、 `callback` を提供する必要はありません。ReactがUIを最新に保ってくれるからです。


## どのコンポーネントがStateを持っているべきでしょうか？

コンポーネントは単純に `props` からデータを取得し、レンダリングすべきです。しかし、時々、ユーザのインプットやサーバのリクエストや、時間の経過に反応する必要が有ります。このような場合にstateを使うのです。

**コンポーネントの多くを、可能な限りステートレスに保つよう試みてください。** このようにすることで、stateを最もロジックに近い場所から離し、冗長性を減らすことができ、アプリケーションが理解しやすくなります。

共通なパターンはただデータをレンダリングするいくつかのステートレスなコンポーネントを構築することで、 `props` を通してその子要素にstateを渡す階級でステートフルなコンポーネントを持つことです。ステートフルなコンポーネントは、ステートレスなコンポーネントが叙述的な方法でデータをレンダリングする助けを行う一方で、全ての相互作用のロジックを持ちます。

## stateで何が行われる *べき* でしょうか？

**stateはUIのアップデートのトリガーとなるコンポーネントのイベントハンドラとしてのデータを保持するべきです。** 現実のアプリでは、このデータはとても小さく、JSONでシリアライズできるものになります。ステートフルなコンポーネントを構築する時は、このstateの表現をできるだけ小さくなるように、また、 `this.state` の中だけにそれらのプロパティを貯めこむように考えてください。 `render()` の中では、このstateに基づく、他の必要な情報をただ計算してください。このように考えたりアプリケーションを記述したりすることが最も正しいアプリケーションへの道であると発見するでしょう。stateに余剰な値や、計算後の値を加えることはReactがそれらを計算してくれるのに頼るのではなく、同期的にそれらを明白に保つ必要があることを意味します。


## stateで何が行われる *べきではない* でしょうか？

`this.state` UIのstateを表す必要がある最小限の量のデータだけを保持するべきです。このような点で以下のようなものは保持するべきではありません。

* **計算されたデータ:**
stateに基づく事前に計算された値について心配は要りません。 `render()` を用いて、全ての計算を行って、UIの一貫性を保証するよりも簡単です。例えば、stateにリスト化されたアイテムの配列を持っていて、文字列としてその数をレンダリングしたいとして、それらをstateに保持するよりも `render()` メソッドに `this.state.listItems.length + ' list items'` を単純にレンダリングすれば良いのです。
* **Reactのコンポーネント:** 隠れたpropsとstateに基づいて `render()` でそれらをビルドしてください。
* **propsからコピーしたデータ:** 可能であれば、propsを真のソースとして使ってみてください。propsは時とともに変化し得るので、正しくstateにpropsを保持する使い方は、以前の値を知っていることになり得ます。
