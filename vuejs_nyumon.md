## vuejs

### 概要

* 能書き
* jsfiddle.netでプレビュー付いたエディタで自由に試すことができる
* jsfiddle.netでvueを指定すると、すでにtodoアプリができている

### サンプルを動かす

helloworldするやつ
```
<!DOCTYPE html>

<title>はじめてのvue.js</title>
<script src="https://unpkg.com/vue@2.5.17"></script>

<div id="app" ></div>

<script>
var vm = new Vue({
        template: '<p>{{msg}}</p>',
        data: {msg: 'hello,world'}
    }).$mount('#app')
</script>
```

#### 構成説明
* deta, view, action に基づいて動的ページを作成していく
* eventはstateへ変更を行い、stateの変化の結果DOM(element)が変更される？

#### オブジェクトと種類
* `var vm = new Vue({しょり})`という決まり文句がある  
vmというグローバル変数にVueというオブジェクトを作成する  
処理のなかで、オプションとなるオブジェクトを設定する  

|オプション名|内容|
|---|---|
|data|UIの状態、データ|
|el|Vueインスタンスをマウントする要素|
|filters|データを文字列と整形する|
|methods|イベントが発生した時などの振る舞い|
|computed|データから派生して算出される値|

### オプションオブジェクトは何に使うの

#### el
* Vueの処理はVueインスタンスを生成しDOM要素にマウントするところから始まる
```
var vm = new Vue({
  el: '#app'
  //
})
```
html中のIDがappな要素にマウントし、変更可能な状態にするという意味になる？
```
var vm = new Vue ({ 処理 })
別の記述や処理
別の記述や処理
vm.$mount(el)
```
前の例とできることは同じだが、任意のタイミングでマウントができる  
elは対象のエレメントの指定に使うオプションオブジェクト  

#### data

あるアイテムの値や個数などの状態を表す。値に応じた表示変更などに利用される  
具体的にはdataプロパティの値が変化するたびVue.jsが自動で検知して表示などが切り替わる
Vueインスタンス生成時にdataを与えておきそれをテンプレートで表示する  
dataにはオブジェクトもしくは関数を与える

```
var vm = new Vue({
  data: {
    キー：値
  }})
```

data.html
```
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
</head>
<script src="https://unpkg.com/vue@2.5.17"></script>
<div id="app">
<p>
  {{ items[0].name }}
</p>

</div>
<script src=data.js></script>
```

data.js
```
var items = [
  {
    name: '鉛筆',
    price: 300,
    quantity: 0
  },
  {
    name: 'note',
    price: 400,
    quantity: 0
  },
  {
    name: '消しゴム',
    price: 500,
    quantity: 0
  }
]

var vm = new Vue({
  el: '#app',
  data: {
    items: items
  }  
})
```
chromeのDevtoolのconsoleで`console.log(vm.items)`とすると内容が確認できる  
Vueの中身は変数vmに属しているので、このように呼び出せる  
また値の変更もできる。vm.items[0].name = 'Pen' と入力すると一番目の要素のnameがPenに
書き換わり、ブラウザの表示も変化する。変更はreloadすると戻る  
  
こうしたデータの書き換えがすぐに反映されるのはVue.jsのリアクティブシステムという  
監視と更新の機能によるもの  
  
能動的な監視を設定してみる
```
vm.$watch(function () {
    return this.items[0].quantity
}, function (quantity) {
    console.log(quantity)
})
```
$watchは監視のメソッドで、第一引数が監視対象の値で  
第二引数が監視対象の値が変わったときに呼ばれる関数となる  
return this.items[0].quantityが監視対象で、  
変わったときにconsole.log(quantity)が呼ばれる  
  
### テンプレート構文

テンプレートの中身はデータの値と表示内容の定義、となる  
データの変化に応じてビューを更新する仕組みをデータバインディングという  
  
テンプレート構文では次の２つの概念が重要らしい
* Mustache記法によるデータの展開
{{}}で囲むやつのこと
* ディレクティブによるHTML要素の拡張
"v-" で始まる、HTMLの属性を用いて独自の拡張をするディレクティブのことらしい  

### テキストへの展開

サンプルで書いたこの部分のこと
```
<div id="app">
  <p>
    {{ items[0].name }} : {{ items[0].price }} x {{ items[0].quantity }}
    {{ items[0].price * items[0].quantity }}
  </p>
</div>
```

中で掛け算とかもできる。  
forとかもかけるかな  
  

### 属性値の展開

htmlのタグなどにVue.jsの展開をしたい場合、v-bindを使用する  
"v-"から始まる拡張ディレクティブ  
  
v-bindの使用例１
```
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
</head>
<script src="https://unpkg.com/vue@2.5.17"></script>
<button id="b-button" v-bind:title="loggedInButton">購入</button>
<script>
  var vm = new Vue({
      el: '#b-button',
      data: {
          loggedInButton: 'ログイン済みなので購入できます'
      }
  })    
</script>
```

buttonにマウスオーバーした時にdata:内で定義されているテキストを表示している  
んだがよくわからない  
buttonにtitleはないし、マウスオーバーするならcss使うのではないのか？  
また、v-bind:の効果範囲はタグの終了までなのか？  
後半に説明があれば確認したい  

v-bindの使用例２
```
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
</head>
<script src="https://unpkg.com/vue@2.5.17"></script>
<button id="b-button" v-bind:disabled="!canBy">購入</button>
<script>
  var vm = new Vue({
      el: '#b-button',
      data: {
          canBy: false
      }
  })    
</script>
```

グレーアウトして押せないボタンが作れる  
title="テキスト"やdisabled="true"はVue.js内で定義があるのだろう  
知らないのでよくわからない  

### フィルタ

表示の加工をする  
0.5を50%と表示したり、YYYYMMDDに/をはさんだりなど  

<details>

<summary>フィルタ使用例</summary>

filter.html
```
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
</head>
<script src="https://unpkg.com/vue@2.5.17"></script>
<div id="app">
  <p>
    {{ items[0].name }} : {{ items[0].price }} x {{ items[0].quantity }}
  </p>
  <p>
    フィルター処理 {{100000000 | numberWithDelimiter}}
  </p>
</div>
<script src="filter.js" charset="UTF-8">
</script>
```

filter.js
```
var items = [{
    name: '鉛筆',
    price: 300,
    quantity: 0
  },
  {
    name: 'note',
    price: 400,
    quantity: 0
  },
  {
    name: '消しゴム',
    price: 500,
    quantity: 0
  }
]

var vm = new Vue({
  el: '#app',
  data: {
    items: items
  },
  filters: {
    numberWithDelimiter: function (value) {
      if (!value) {
        return '0'
      }
      return value.toString().replace(/(\d)(?=(\d{3})+$)/g, '$1,')
    }
  }
})
```

</details>

例では値に３個区切りで,を挿入している  
また、結果にさらに|でフィルタをつけることもできる  

### 算出プロパティ(computed)

埋め込みには単一式しか書けないきまりがあるので、それ以上書く必要がある場合  
computed:で書いて、埋め込み側ではcomputedの要素を呼び出す仕組み  


### thisによる参照

そのインスタンス自身のこと  
他言語でもよくつかわれる

### サンプルアプリケーション
上記要素を踏まえたサンプルを見てみる

<details>

<summary>計算サンプル</summary>

daisample1.html
```
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
</head>
<script src="https://unpkg.com/vue@2.5.17"></script>
<div id="app">
  <p>
    {{ items[0].name }} : {{ items[0].price }} x {{ items[0].quantity }}
  </p>
  <p>
    小計: {{ totalPrice | numberWithDelimiter}} 円
  </p>
  <p>
    合計(税込): {{totalPriceWithTax | numberWithDelimiter}}円
  </p>
</div>
<script src="daisample.js" charset="UTF-8">
</script>
```
daisample1.js
```
var items = [{
    name: '鉛筆',
    price: 300,
    quantity: 0
  },
  {
    name: 'note',
    price: 400,
    quantity: 0
  },
  {
    name: '消しゴム',
    price: 500,
    quantity: 0
  }
]

var vm = new Vue({
  el: '#app',
  data: {
    items: items
  },
  filters: {
    numberWithDelimiter: function (value) {
      if (!value) {
        return '0'
      }
      return value.toString().replace(/(\d)(?=(\d{3})+$)/g, '$1,')
    }
  },
  computed: {
    totalPrice: function () {
      return this.items.reduce(function (sum, item) {
        return sum + (item.price * item.quantity)
      }, 0)
    },
    totalPriceWithTax: function () {
      return Math.floor(this.totalPrice * 1.08)
    }
  }
})
```

</details>

itemsの個数が変更できないので、clomeのconsoleで追加してやると確認できる
`items[0].quantity = 1`など  


### v- から始まるディレクティブの話

v-bind: title="data内の変数"  
上記のような使い方以外によく使う要素の表示/非表示  
v-if
```
<p v-if="引数">
真偽で表示、非表示
</p>
```
v-show  
```
<p v-show="引数">
真偽で表示、非表示
</p>
```
* v-ifは対象のエレメントを削除する
* v-showはstyleのdisplayプロパティの操作(つまりソースには載る)

```
html
  <p v-show="!canBuy">
    {{ 1000 | numberWithDelimiter}}円以上から購入できます
  </p>
js

computed: {
  canBuy: function () {
      return this.totalPrice >= 1000 //1000円以上であればtrueを返す
  }
}
```

### ディレクティブでスタイルを変えたい

売り切れたら赤文字にしたいとかタイムセールのときはフォント変えたいとか  
動的な変化でスタイルを変えたいことが起こる  
v-bindディレクティブでそれを実現できるが、v-bindはディレクティブのなかでも  
書き方が特殊なので、以下に記載する  

```
v-bind: 属性名="データを展開した属性値"
```
属性名にはclass or styleが入る。左辺にはdata:の指定値が入る。  
  
classとstyleの属性値はCSSの記法通りに当て込むのが困難。   
classやstyleは複数の要素を持っているし、複数の場合はclassが空白区切り、  
色の指定がコロンなので、機械的に書き換えるのは困難  
  
"データを展開した属性値"にオブジェクトや配列が指定された場合、CSSのフォーマットに  
宛て込めてくれる仕組みがある  
  
#### classの場合  
```
<p v-bind:class="{shark: true, mecha: false}"></p>
```
オブジェクトを指定した際に値がtrueのプロパティ名を指定の属性値と判断する  
上記の場合はsharkが指定値となる  
  
```
<p v-bind:class="{error: !canBuy}">
  1000円以下は購入できません
</p>
```

その場に埋めないで、computedから引っ張る方が丁寧
```
JS
computed: {
  errorMessageClass: function () {
    return {
      error: !this.canBuy
    }
  }
}
HTML 
<p v-bind:class="errorMessageClass">
  1000円以下は購入できません
</p>
```


#### styleの場合

```
<p v-bind:style="{color: 'red'}">a</p>
```
<p style="color: red;">a</p>となる  
  
複合例としては
```
<p v-bind:style="{border: (canBuy ? '' : `1px solid red`), color: (canBuy ? `` : `red`)}">1000円以上でないと買えません</p>
```
読みづらいがこれでIF文らしい。Trueだと何も書かない  

computedから引っ張る
```
JS
errorMessageStyle: function () {
  return {
    border: this.canBuy ? '' : '1px solid red'
    color: this.canBuy ? '' : 'red'
  }
}

HTML
<p v-bind:style="errorMessageStyle">
1000円以上でないと買えません</p>
```

#### v-bindの省略記法

最悪
`v-bind:disabled` -> `:disabled`

```
<p v-bind:class="{error: !canBuy}">
  1000円以下は購入できません</p>
<p :class="{error: !canBuy}">
  1000円以下は購入できません</p>
```
このくらい書けばいいのにと思う


<details>

<summary>合計金額が1000円以下だと文字が赤くなる</summary>

daisample1.html
```
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
</head>
<script src="https://unpkg.com/vue@2.5.17"></script>
<div id="app">
  <div :style="errorMessageStyle">
    <p>{{ items[0].name }} : {{ items[0].price }} x {{ items[0].quantity }}</p>
    <p>小計: {{ totalPrice | numberWithDelimiter}} 円</p>
    <p>    合計(税込): {{totalPriceWithTax | numberWithDelimiter}}円</p>
    <p v-show="!canBuy">
      {{ 1000 | numberWithDelimiter}}円以上から購入できます
    </p>
  </div>
</div>
<script src="daisample2.js" charset="UTF-8">
</script>
```

daisample1.js
```
var items = [{
    name: '鉛筆',
    price: 300,
    quantity: 0
  },
  {
    name: 'note',
    price: 400,
    quantity: 0
  },
  {
    name: '消しゴム',
    price: 500,
    quantity: 0
  }
]

var vm = new Vue({
  el: '#app',
  data: {
    items: items
  },
  filters: {
    numberWithDelimiter: function (value) {
      if (!value) {
        return '0'
      }
      return value.toString().replace(/(\d)(?=(\d{3})+$)/g, '$1,')
    }
  },
  computed: {
    totalPrice: function () {
      return this.items.reduce(function (sum, item) {
        return sum + (item.price * item.quantity)
      }, 0)
    },
    totalPriceWithTax: function () {
      return Math.floor(this.totalPrice * 1.08)
    },
    canBuy: function () {
      return this.totalPrice >= 1000 //1000円以上であればtrueを返す
    },
    errorMessageStyle: function () {
      return {
        border: this.canBuy ? '' : '1px solid red',
        color: this.canBuy ? '' : 'red'
      }
    }
  }
})
```

</details>

### リストレンダリング(v-for)

v-for="要素 in 配列"  

```
HTML
<ul>
  <li v-for="item in arr" v-bind:key="item">{{ item }}</li>
</ul>
JS
var vm = new Vue({
  data: {
    arr: ['hoge','fuga','foo']
  }
})
```
v-bindでkeyを指定する必要がある、リアルタイムで書く都合で描画が早くなるらしい  
これでarrの内容をeachでliとして表示できる  
  
インデックスをつけることもできる（indexは配列の要素番号）
```
<ul>
  <li v-for="(item,index) in arr" v-bind:key="item">{{ index }} {{ item }}</li>
</ul>
```

### イベントハンドリング(v-on)

今までコンソールでdata:の中身を変更して反映させていた。  
この中身の変更をイベントと呼ぶ  
Vue.jsではv-onディレクティブを使ってイベントハンドリングを行う  

```
v-on: イベント名="式として実行したい属性値"
```

```
HTML
  <ul>
    <li v-for="item in items" v-bind:key="item.name">
      {{item.name}}の個数: <input type="number" v-on:input="item.quantity = $event.target.value" v-bind:value="item.quantity" min="0">
    </li>
  </ul>
JS
変更なし
```

changeイベントを使う

```
HTML
  <ul>
    <li v-for="item in items" v-bind:key="item.name">
      {{item.name}}の個数: <input type="number" v-on:change="item.quantity = $event.target.value" v-bind:value="item.quantity" min="0">
    </li>
  </ul>
JS
変更なし
```
v-onでinputではなくchangeにした場合、別の要素にフォーカスが移った時に表示の更新がされるようになる。  

v-onの省略記法

```
書き換え
v-on:click -> @click
書き換え例
<button :disabled="!canBuy" @click="doBuy">購入</button>
```

### フォーム入力バインディング　(v-model)

すべての入力箇所にv-on:changeで要素を当て込むのは規模が大きくなると煩雑  
v-modelを使うとフォームの数が増えても楽らしい  
v-modelは双方向データバインディング？を実現するディレクティブらしい？  

v-onで書いた部分をv-modelに置き換える
```
HTML
  <ul>
    <li v-for="item in items" v-bind:key="item.name">
      {{item.name}}の個数: <input type="number" v-model:change="item.quantity" min="0">
    </li>
  </ul>
JS
変更なし
```

### ライフサイクルフック

インスタンスの誕生から削除までの状態変化のこと






