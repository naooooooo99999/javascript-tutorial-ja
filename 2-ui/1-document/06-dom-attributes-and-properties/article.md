# 属性とプロパティ

ブラウザがページをロードするとき、HTMLテキストを "読み" (別の言葉: "パース")、そこから DOM オブジェクトを生成します。要素ノードの場合、ほとんどの標準のHTML属性は自動的に DOM オブジェクトのプロパティになります。

例えば、もしタグが `<body id="page">` の場合、DOM オブジェクトは `body.id="page"` を持ちます。

しかし、属性-プロパティのマッピングは1対1ではありません! このチャプターでは、これら2つの概念を分け、それらが同じであるとき、異なるときにそれらがどのように機能するか、見てみましょう。

[cut]

## DOM プロパティ 

私たちはすでに組み込みの DOM プロパティを見ました。それらはたくさんありますが、技術的には誰も制限はしておらず、もしそれが十分でなければ -- 我々自身で追加することができます。

DOM ノードは通常のJavaScriptオブジェクトです。それを変更することができます。

例えば、`document.body` に新しいプロパティを作成してみましょう:

```js run
document.body.myData = {
  name: 'Caesar',
  title: 'Imperator'
};

alert(document.body.myData.title); // Imperator
```

同様にメソッドの追加もできます:

```js run
document.body.sayHi = function() {
  alert(this.tagName);
};

document.body.sayHi(); // BODY (このメソッドの "this" の値は document.body)
```

`Element.prototype` のように組込のプロトタイプを修正し、すべての要素に新しいメソッドを追加することも可能です:

```js run
Element.prototype.sayHi = function() {
  alert(`Hello, I'm ${this.tagName}`);
};

document.documentElement.sayHi(); // Hello, I'm HTML
document.body.sayHi(); // Hello, I'm BODY
```

従って、DOM プロパティとメソッドは、通常のJavaScriptオブジェクトと同じように振る舞います:

- 任意の値を持つことができる
- 大文字と小文字が区別される(`elem.NoDeTyPe` ではなく、`elem.nodeType` と書きます。)

## HTML 属性 

HTML 言語では、タグは属性を持つことがあります。ブラウザがHTMLテキストを読み込み、タグに対する DOM オブジェクトを作成するとき、それは *標準の* 属性を認識し、それらから DOM プロパティを生成します。

従って、要素が `id` または別の *標準の* 属性を持っているとき、対応するプロパティは生成されます。しかし、属性が非標準でない場合には、それは起こりません。

例:
```html run
<body id="test" something="non-standard">
  <script>
    alert(document.body.id); // test
*!*
    // 非標準の属性はプロパティを生成しません
    alert(document.body.something); // undefined
*/!*
  </script>
</body>
```

ある要素に対する標準属性は別の要素にとっては未知になる可能性があることに注意してください。例えば、`"type"` は `<input>` ([HTMLInputElement](https://html.spec.whatwg.org/#htmlinputelement)) では標準ですが、`<body>` ([HTMLBodyElement](https://html.spec.whatwg.org/#htmlbodyelement)) では標準はありません。標準属性は対応する要素クラスの仕様で記述されています。

ここではそれを見ることができます:
```html run
<body id="body" type="...">
  <input id="input" type="text">
  <script>
    alert(input.type); // text
*!*
    alert(body.type); // undefined: 非標準のため、DOM プロパティは作られません
*/!*
  </script>
</body>
```

なので、もし属性が非標準の場合、それに対する DOM プロパティはありません。このような属性にアクセスする方法はあるでしょうか？

もちろんです。すべての属性は次のメソッドを使ってアクセスすることができます。:

- `elem.hasAttribute(name)` -- 存在をチェックします。
- `elem.getAttribute(name)` -- 値を取得します。
- `elem.setAttribute(name, value)` -- 値を設定します。
- `elem.removeAttribute(name)` -- 属性を削除します。

これらのメソッドは HTML で書かれているものと正確に操作します。

`elem.attributes` を使うことで、すべての属性を読み込むこともできます: 組み込み [Attr](https://dom.spec.whatwg.org/#attr) クラスに属し、 `name` と `value` プロパティをもつ、オブジェクトのコレクションです。

これは、非標準のプロパティを読み込む例です:

```html run
<body something="non-standard">
  <script>
*!*
    alert(document.body.getAttribute('something')); // 非標準
*/!*
  </script>
</body>
```

HTML属性は次の特徴を持っています:

- それらの名前は大文字小文字を区別しません(それはHTMLです: `id` は `ID` と同じです)
- それらは常に文字列です。

ここでは、属性を使った動作の拡張デモを紹介します。:

```html run
<body>
  <div id="elem" about="Elephant"></div>

  <script>
    alert( elem.getAttribute('About') ); // (1) 'Elephant', 読み込み

    elem.setAttribute('Test', 123); // (2), 書き込み

    alert( elem.outerHTML ); // (3), 参照

    for (let attr of elem.attributes) { // (4) すべてをリスト
      alert( attr.name + " = " + attr.value );
    }
  </script>
</body>
```

注意事項:

1. `getAttribute('About')` -- 最初の文字はここでは大文字で、HTMLではすべて小文字です。 しかし、それは重要ではありません。: 属性名は大文字と小文字を区別しません。
2. 属性には何でも割り当てることができますが、それは文字列になります。 だからここでは値として `"123"` があります。
3. 私たちが設定したものを含むすべての属性は `outerHTML` で見えます。
4. `attributes` のコレクションは反復可能で、属性は `name` と `value` を持っています。

## プロパティ-属性 の同期 

標準の属性を変更するとき、対応するプロパティは自動更新され、(いくつかの例外を除いては)逆もまた然りです。

下の例では、`id` は属性として修正され、我々はそのプロパティの変更も見ることができます。そして後ろも同じです。:

```html run
<input>

<script>
  let input = document.querySelector('input');

  // 属性 => プロパティ
  input.setAttribute('id', 'id');
  alert(input.id); // id (更新された)

  // プロパティ => 属性
  input.id = 'newId';
  alert(input.getAttribute('id')); // newId (更新された)
</script>
```

しかし、例外もあります。例えば、 `input.value` の同期は 属性から -> プロパティの場合のみで、逆は同期されません: 

```html run
<input>

<script>
  let input = document.querySelector('input');

  // 属性 => プロパティ
  input.setAttribute('value', 'text');
  alert(input.value); // text

*!*
  // ☓ プロパティ => 属性
  input.value = 'newValue';
  alert(input.getAttribute('value')); // text (更新されていません!)
*/!*
</script>
```

上の例では:
- 属性 `value` の変更は、プロパティも更新します。
- しかし、プロパティの変更は属性に影響を与えません。

この "特徴" は実際には便利な場合があります。なぜなら、ユーザは `value` を変更する可能性があるからです。その後、HTMLから元の値を復元する場合、それは属性の中にあります。

## DOM プロパティは型付けられています 

DOM プロパティは常に文字列とは限りません。例えば、`input.checked` プロパティ(チェックボックス用)は真偽値です:

```html run
<input id="input" type="checkbox" checked> checkbox

<script>
  alert(input.getAttribute('checked')); // 属性値: 空文字
  alert(input.checked); // プロパティ値: true
</script>
```

他の例があります。`style` 属性は文字列ですが、`style` プロパティはオブジェクトです:

```html run
<div id="div" style="color:red;font-size:120%">Hello</div>

<script>
  // string
  alert(div.getAttribute('style')); // color:red;font-size:120%

  // object
  alert(div.style); // [object CSSStyleDeclaration]
  alert(div.style.color); // red
</script>
```

これは重要な違いです。しかし、たとえ DOM プロパティの型が文字列でも、属性とは異なる可能性があります!

例えば `href` DOM プロパティは、たとえ属性に相対URLや単なる `#hash` が含まれていても常に *完全な* URLです。

以下、例です:

```html height=30 run
<a id="a" href="#hello">link</a>
<script>
  // 属性
  alert(a.getAttribute('href')); // #hello

  // プロパティ
  alert(a.href ); // 完全なURLの形式 http://site.com/page#hello
</script>
```

もし HTMLの中で書かれている通りの正確な `href` や他の属性値が必要な場合、`getAttrubute` を使います。


## 非標準の属性、データ・セット 

HTMLを記述する際には、多くの標準属性を使います。しかし、非標準、カスタムのものはどうでしょう？まず、それらが役立つのかどうか、何のために必要なのかを見てみましょう。

HTML から JavaScript へカスタムデータを渡す、もしくは JavaScript のためにHTML要素を "マークする" ために、非標準の属性が使われることがあります。

このようになります:

```html run
<!-- "name" を表示する div をマークします -->
<div *!*show-info="name"*/!*></div>
<!-- ここは age です -->
<div *!*show-info="age"*/!*></div>

<script>
  // コードは、マークのある要素を探し、要求されたものを表示します
  let user = {
    name: "Pete",
    age: 25
  };

  for(let div of document.querySelectorAll('[show-info]')) {
    // フィールドに、対応する情報を挿入します
    let field = div.getAttribute('show-info');
    div.innerHTML = user[field]; // Pete, then age
  }
</script>
```

また、要素のスタイルのために使われることもあります。

例えば、ここでは注文状態に対して、属性 `order-state` が使われます:

```html run
<style>
  /* スタイルはカスタム属性 "order-state" に依存します */
  .order[order-state="new"] {
    color: green;
  }

  .order[order-state="pending"] {
    color: blue;
  }

  .order[order-state="canceled"] {
    color: red;
  }
</style>

<div class="order" order-state="new">
  A new order.
</div>

<div class="order" order-state="pending">
  A pending order.
</div>

<div class="order" order-state="canceled">
  A canceled order.
</div>
```

属性が `.order-state-new`、`.order-state-pending`、`order-state-canceled` のようなクラスよりも好ましい理由は何でしょう？

それは、属性はより管理がしやすいためです。状態は次のように簡単に変更できます。:

```js
// 古いクラスの削除/新しいクラスの追加よりも少しシンプルです
div.setAttribute('order-state', 'canceled');
```

しかし、カスタム属性には問題がある可能性があります。仮に、我々の目的のために非標準の属性を使い、後に標準がそれを発表し何かをさせるとしたらどうでしょう？HTML言語は生きており、成長し、開発者のニーズに合うようなより多くの属性が登場しています。このような場合に予期しない影響が生じることがあります。

衝突を避けるために、[data-*](https://html.spec.whatwg.org/#embedding-custom-non-visible-data-with-the-data-*-attributes) 属性があります。

**"data-" で始まるすべての属性はプログラマのために予約されています。それらは `dataset` プロパティで利用可能です。**

例えば、もし `elem` が `"data-about"` という名前の属性を持っている場合、それは `elem.dataset.about` として利用できます。

このように:

```html run
<body data-about="Elephants">
<script>
  alert(document.body.dataset.about); // Elephants
</script>
```

`data-order-state` のような複数語はキャメルケースになります: `dataset.orderState`.

ここでは "order state" の例を書き直しています:

```html run
<style>
  .order[data-order-state="new"] {
    color: green;
  }

  .order[data-order-state="pending"] {
    color: blue;
  }

  .order[data-order-state="canceled"] {
    color: red;
  }
</style>

<div id="order" class="order" data-order-state="new">
  A new order.
</div>

<script>
  // read
  alert(order.dataset.orderState); // new

  // modify
  order.dataset.orderState = "pending"; // (*)
</script>
```

`data-*` 属性の利用は、カスタムデータを渡す有効かつ安全な方法です。

data-属性の読み込みだけでなく、変更もできることに注意してください。そして、CSS はそれに応じて ビューを更新します。: 上の例では、最後の行 `(*)` で色を青に変更しています。

## サマリ 

- 属性 -- は HTML で書かれているものです。
- プロパティ -- は DOM オブジェクトの中にあるものです。

小さな比較:

|            | プロパティ | 属性 |
|------------|------------|------------|
|型|任意の値、標準プロパティーには、仕様に記述されているタイプがあります|文字列|
|名前|名前は大文字と小文字を区別します|名前は大文字と小文字を区別しません|

属性を操作するメソッドは次のとおりです:

- `elem.hasAttribute(name)` -- 存在をチェックします
- `elem.getAttribute(name)` -- 値を取得します
- `elem.setAttribute(name, value)` -- 値をセットします
- `elem.removeAttribute(name)` -- 属性を削除します
- `elem.attributes` はすべての属性の集合です

ほとんどのニーズに対して、DOMプロパティはうまく機能します。 次の例ように、正確な属性が必要なとき、DOMプロパティが適切でない場合にのみ属性を参照すべきです。:

- 非標準の属性が必要な場合。ただし、`data-` で始まる場合は `dataset`を使うべきです。
- HTMLで "書かれている" 値を読みたい場合。 DOMプロパティの値は異なる可能性があります。例えば、 `href` プロパティは常に完全なURLであり、私たちは "オリジナル" の値を取得したい場合があります。
