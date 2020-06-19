## javascript

[[_TOC_]]

## 簡素な例

```
<!DOCTYPE html>
<html>
<head>
<script>
	function calc() {
		var w = document.getElementById("weight").value;
		var h = document.getElementById("height").value;
		var r = w / (h * h); //bmi
		
		document.getElementById("result").textContent = "あなたのBMIは" + r + "です";
	}
</script>
</head>
<body>
	<h1>BMIの計算</h1>
	体重(kg) : <input id="weight" /><br />
	身長(m)  : <input id="height" /><br />
	計算 : <button onclick="calc()">計算</button>

	<p id="result"></p>
</body>
</html>
```
何らかの事象に応じて呼び出される関数をイベントハンドラと呼ぶ  
CGI時代のボタンとは違う  


## 変数とスコープ（ES6）
命名規則があるらしい  
```
workNum -> 二語以上は大文字で区切る
1gouki -> 数字で変数名を開始してはならない
```
  
let と var それぞれ変数を宣言するときにつける  
* let はブロックをスコープとしている
* var は関数をスコープとしている
関数内でブロックを挟む場合、letは同名の変数が干渉しない  
var は干渉する。(ブロック内外で同じ個体として扱われる。)  
  
const hoge = "65536";  
定数の宣言  
  
a++; とかかけるらしい  
a += 1; //１を足す  
a *= 3; //３をかける  
この書き方、四則演算剰余算全部できるの知らなかった  
  
リテラルという単語がよく出てくる  
上記のようにコード内で変数に対して直接指定した値(65536,1,3)をリテラルと呼ぶ  
テキストなどへの変数の埋め込み記載をテンプレートリテラルというらしい  
JSではバッククォートで囲む必要がある  
```
let name = "資本主義経済";        //これもリテラル(資本主義経済)
console.log(`ほげ${name}　ふが`); //これはテンプレートリテラル(${name})
```
他に後述する「関数」で「関数リテラル」という単語も出てくる  
  
### jsの比較演算子
a === b //等しい  
a !== b //異なる  
  
複数条件  
a > 5 && a < 10 //6以上かつ9以下  
a < 5 || a > 10 //4以下または11以上  
書き方が特殊  
  
## 配列とオブジェクト
配列  
特に他言語と変わらない  
```
const characters = ["ore", "sanjyou"];
console.log(characters[0]);
```
  
オブジェクト  
ハッシュのことをオブジェクトと呼ぶらしい  
配列と違って{}で囲む  
  
配列と合わせてつかえる。改行したほうが可読性はよくなる。
```
const characters = [
  {name: "にんじゃわんこ", age: 14},
  {name: "ひつじ仙人", age: 1000}
];
console.log(characters[1].name); //ひつじ仙人
```

上記でcharacters[2]とするとundefinedという値が返ってくる  
これはstringではなく、そうゆうオブジェクトみたい  
```
characters[3].age === undefined //true
characters[3].age === "undefined" //False
```

ifでundefinedであれば、それ用の処理を例外として書いておく  
関数をオブジェクトに埋める  
  
## クラス
定義
```
　class Animal {}
　const animal = new Animal();
```
### コンストラクタ
このクラスを利用して作成されたインスタンスに対して最初に一度だけ行う処理のこと
```
　class Animal { constructor(){ } }

　const animal = new Animal();
　上記のようにnewで生成したときに実行される。console.logを設定すると実行されることが確認できる
```
クラスは複数個作成されるので、thisでこのオブジェクトを指定している  
  
作成時に引数を入れることで、中身を変えたオブジェクトを作ることができる  
```
class Animal {
   constructor(name, age) {
      this.name = name;   this.age = age;
   }
}
const animal = new Animal("モカ", 8);

console.log(`名前：${animal.name} 年齢：${animal.age}`);
```
### メソッド
クラス内で呼び出される関数のこと
```
class Animal {
   constructor(name, age) {
      this.name = name;   this.age = age;
   }
   greet() {
      console.log("こんにちは");
   }
   info() {
      this.greet(); //メソッド内でメソッドも呼べる。
      console.log(`名前は${this.name}です`); //変数を拾うときはthis.変数
      console.log(`年齢は${this.age}です`);  //で使用する。
   }
}
const animal = new Animal("モカ", 8);

console.log(`名前：${animal.name} 年齢：${animal.age}`);

animal.greet();  //メソッドはこの形で呼び出す
```
### クラスの継承
Animalクラスを継承しDogクラスをつくり、インスタンスを作る
```
import Animal from "./animal.js";
class Dog extends Animal {}
```
  
１ファイルで複数の項目をエクスポートできる
```
export {dog1, dog2};
import {dog1, dog2} from "./dogdata";
```
  
### オーバーライド
クラスメソッド  
子クラスで、親クラスと同名のメソッドを作ると、  
内容は子クラスで定義した内容となる。これをオーバーライドと呼ぶ  
  
コンストラクタ  
子クラスで親クラスに無い、追加の変数を使いたい場合、コンストラクタを上書きする必要がある  
  
作成例
```
constructor (hoge, fuga, new){
   super(hoge, fuga) // 親クラスで定義済みの変数
   this.new = new    // 新規の変数
}
```
  
## ファイル分割でimport,export
ただファイルを分けるだけではなく、追加の記載が必要  
クラスしかおいてない側  
export default 「クラス名」  
　→こちらは末尾に書く  
  
読み込むメイン側のjsファイル  
imprort Animal from "./animal.js";  
　→こちらは行頭に書く  
  
クラスだけでなく、変数も関数もexport,importも受け取りができる  
  
exportするときのdefaultの意味  
そのファイルで複数エクスポートするとき、一つだけ指定できる  
エクスポートが指定されたファイルをインポートするとき、エクスポート指定と名前が異なってもインポートが成功する  

名前付きエクスポート
```
export { dog1 };
import { 値の名前 } from "./ファイル名";
```

## パッケージのimport

いくつか使ってみる  
  
文字色を変えられるchalk  
```
import chalk from "chalk";
console.log(chalk.yellow(`名前は${this.name}です`));
console.log(chalk.bgCyan(`犬種は${this.breed}です`));
```

インタラクティブな入力ができるreadline
```
readlineSync.question("犬種を入力してください: ");
const name = readlineSync.question("名前を入力してください: ");
const age = readlineSync.questionInt("年齢を入力してください: ");
```

## 配列の操作
```
hoge = ["hoge" , "fuga" , "foo"]
hoge.push("moge");
```

forEachによる繰り返しでの配列内容への処理方法  
characters.forEach((character) => {console.log(character)});  
下記例でしているが、console.logではなくreturnで返すと、表示せずとも結果を扱える  
  
配列から特定条件の要素を探す  
```
const numbers = [1, 3, 5, 7, 9];
const foundNumber = numbers.find(
  (number) => {return number % 3 === 0;}
);
```
一つ一致する条件があればメソッドは終了するので、上記だと9は入らない  
「find」を「filter」とすることで、一致するすべての要素を配列で返す  
  
特定配列の加工  
  
## 三項演算子
```
i++ とかは一項演算子　iという1つの項目に演算ができるもの。
+ - / * は二項演算子　a + b とか2つの項目に演算ができるもの。
 ? :　　が三項演算子　if elseなど条件式を3つの項目演算を行う。

v = 8
f = (v % 2 == 0) ? "偶数" : "奇数";
print f //偶数
```
## 制御文
if文
```
if (level > 10) {
  console.log("レベルが10より大きい");
}
```

ifによる真偽値のチェック
```
const check = (number) => {
  return number % 3 === 0;
};
if (check(123)) {
  console.log("3の倍数です");
} else {
  console.log("3の倍数ではありません");
}
```

for文ループ
```
for (初期値; 継続条件; カウンタ更新式) { nakami }
for (var i = 0; i < 10; i++) {
	//処理
}
```
カウンタ更新式にセミコロンは必要ない  
  
while文
```
while (継続条件){
処理
継続条件を満たさなくする処理を入れること。
}

continue;
break;

while (条件式１) {
  if (条件式２) {
    continue; //上の処理に戻りループを継続する。
  }
  if (条件式３) {
    break; //ループ処理を抜ける
  }
}

case = 2;
switch (case) {
 case 1:
  console.log("1");
 break;
 case 2:
  console.log("2");
 break;
 default:
  console.log("reigai!");
 break;
}
```

## 関数
定義方法
```
const greet = function(hoge) {
  console.log("こんにちは！");
  console.log(`${hoge}を学習していきましょう！`);
};
```
定数に埋めていくらしい。特殊な書き方だと感じるが、関数名を変更するのはおかしいのでこれで正しい  
これを無名関数と呼ぶ  
  
アロー関数  
ES6から使えるらしい  
無名関数 function() を () => として書ける省略記法だが、普通に書く場合と挙動が変わるので  
必ず覚えなければならない。  
`(引数,...)=>{...関数の本体...}`  
  
sample(円の面積を求める)
```  
var area1 = function (r) {
  return r * r * Math.PI;
};  
```

```
// sampleをアロー関数で省略するとこうなる
var area1 = (r) => {
  return r * r * Math.PI;
};
// 関数の内容が一文の場合、{}を外して書ける
var area1 = (r) => r * r * Math.PI;
// 引数が１つの場合、()を外して書ける(引数が無い場合は()を書く)
var area1 = r => r * r * Math.PI;
// 定義と実行を同時に行う
console.log(((r) => r * r * Math.PI)(radius));
```  
アロー関数で書いた場合、結果がreturnで値を返したことになる  
  

## Domの操作
bodyに以下の要素があって
```
<ol id="container"></ol>
```

javascriptでその要素に対して追加と削除ができる。
```
var item = container.lastChild;
container.appendChild(item);
container.removeChild(item);
```

getElementByIdで取得した要素を削除しようとすると
```
var target = document.getElementById("target");
var parent = target.parentNode;
parent.removeChild(target);
```

## スタイルの指定
```
var index = 1;
var colors = ["red", "green", "blue", "yellow"]
document.getElementById("ufo").style.color = colors[index];
```
idがufoとなっているコンテンツタグについて文字色がgreenになる  

## body読み込み時に関数を呼びたい
```
<body onLoad="hoge()">
```

## オブジェクトの操作
```
window.addEventListener("keydown", keydown)
```

## ファイル読み込み

非同期処理すらしない簡単な奴  
javascriptの洗礼を受ける  
  












