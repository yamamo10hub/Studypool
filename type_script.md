## TypeScript

[[_TOC_]]

## とは

Microsoftが作ったjavascriptの派生的な言語  
コンパイルするとjavascriptになる  
  
最近よく聞くウェブフレームワークで使用されている  

### 特徴など

静的型付け
`let hoge:string `のように、変数宣言の後ろに型をつける  
コンパイル時にstring以外が入っているとエラーがでる  

### インターフェースとクラス

```
interface Person {
    firstName: string;
    lastName: string;
}
function greeter(person: Person){
    return "Hello, " + person.firstName + " " + person.lastName;
}
let user = {firstName:"Fuga", lastName:"Hoge"};
document.body.innerHTML = greeter(user);
```

interfaceの特徴
* 型付けされた変数宣言のみが存在する  
* クラスやメソッドの引数として使う  
interfaceのメリット
* 異なるクラスやメソッドが、同じ変数を使って動作していることを表示できる

## @itで学ぶ

よくわからないので、解説を読む
### 大枠

#### 簡単な実行環境

http://www.typescriptlang.org/Playground
  
リアルタイムで見れる

#### 簡単なサンプルを見る

```
var Message:string;
class Cat {
  age:number;
  weight:number;
}
var myCat = new Cat();
myCat.age = 3;
myCat.weight = 5.1;
Message = "うちの猫は" + myCat.age + "歳で、体重は" + myCat.weight + "kgです";  
alert(Message);
```
  
実行用HTML
```
<!DOCTYPE html>
<html>
    <head><title>greeter ts ver</title></head>
    <body>
        <script src="sample.js"></script>
    </body>
</html>
```

### 列挙型(enum)

typescriptで使えるデータ型
```
enum SEASONS{SPRING, SUMMER, AUTUMN, WINTER};
alert(SEASONS.AUTUMN);
```
２が返る  
  
1234が春夏秋冬であると定義しなくても、確認ができる  
また、hashより書きやすい  
```
enum SEASONS{SPRING, SUMMER, AUTUMN, WINTER};
var season:SEASONS;  
season = SEASONS.SUMMER; 

if(season == SEASONS.SUMMER){
  alert("夏ですね");
} else {
  alert("夏ではないですね");
}
```

数値を定義することもできる
```
enum SEASONS{SPRING=1, SUMMER=2, AUTUMN=4, WINTER=8};
var season:SEASONS;
season = SEASONS.SPRING + SEASONS.SUMMER; 

switch(season){
case SEASONS.SPRING:
  alert("春物です");
  break;
case SEASONS.SPRING + SEASONS.SUMMER:
  alert("春夏物です");
  break;
default:
  alert("秋または冬にも使えます");
}
```

### 式と演算

```
var price, total, taxrate : number;
price = 1200;   // 代入
taxrate = 0.08; // 代入
total = price * (1 + taxrate);  // 計算
alert(total);
```
javascriptではnumberと型宣言していないが、`(1 + taxrate)`でエラーは出ずに正しく計算される  



## Angular赤本で学ぶ

### 変数

変数定義時に ": 型名" を宣言する
```
let age: number = 30;
console.log(age);
```

型名を宣言しなくても変数宣言はできる。  
代入する値を見て勝手に型を決める型推論という機能がある  
ただし型推論は宣言時のみで、初期値がない場合はany型というどんな値でも許容する  
型として決定してしまうので、注意する  
```
let message;
message = 'hoge';
message = 30;
console.log(message);
```

型の種類
```
let flag: boolean = false;
let age: number = 30;
let name: string = 'hoge fuga';
let pet: string[] = ['犬','猫','ハムスター'];
console.log(pet[1]); // 犬

// タプル型
let score: [number, string] = [100, 'great'];
console.log(score[0]);  // 100
//enum型
enum Color {Red, Green, Blue};
let red: Color = Color.Red;
console.log(red);         // 0
console.log(Color[red]);  // Red
```

テンプレート文字列
```
let message = `よく使われる汎用マジック変数として日本では
${name}が使われる`;
console.log(message);
```
バッククォートで囲んだ文字列のことをいう。特徴は「文字列の埋め込みができる」こと
「改行文字を含められる」ところ  
  
連想配列(ハッシュ)
```
let duet: {[index: string]: string;} = {first: '山田' , second: '鈴木'};
console.log(duet['first']);
```
左辺がキーとなるが、typescriptの型定義ではインデックスシグニチャと呼ばれる。  
"index"の部分は任意の値でも良い。  
```
interface MyMap {
  {index: string}: string;
}
let duet: MyMap = {first: '山田' , second: '鈴木'};
```
この２つの記述は同じ意味となる。  
Angularでは大抵２番目で書かれているので、こちらの記法にする    

### 定数

定数。オブジェクトの代入などにも使う
```
const PI: number = 3.14;
PI = 3.141; // 定数なのでエラーとなる

let redius: number = 2;
let area = radius * radius * PI
console.log('面積： ' + area);
```

### 型アサーション（型変換）
```
function greet(name: string) {
  return 'こんにちは、' + name;
}
console.log(greet(<any>108));
console.log(greet(108 as any));
```
どちらもany型となる。as構文はよくわからない  
any型に変換することで、引数の型違いでエラーが起きない  
  
### 関数
引数、戻り値なし
```
function greet(): void {
  console.log('Hello!');
}
greet();
```
引数がない場合 ": void" という型を指定する 
  
引数、戻り値あり
```
function add(x: number, y: number): number {
  return x + y;
}
console.log(add(4, 5));
```
引数、返り値の型宣言は省略してもよいらしい。(引数の隣が返り値の型宣言)
  
省略可能な引数
```
function showCost(price: number, discount?: number) {
  if(discount) {
    console.log(`価格：${price - discount}円`);
  }else{
    console.log(`価格：${price}円`);
  }
}
showCost(1000);
showCost(1000,200);
}
```
?を引数につけると、それがなくてもエラーにならない。  
ただし内部で?をつけた引数がなくてもつまらないように作る  
または`discount?: number = 0`としてデフォルト値を入れておく  
  
可変長引数
```
function sum(...values: number[]): number {
  let result: number = 0;
  for (let i: number = 0; i < values.length; i++) {
    result += values[i];
  }
  return result;
}
console.log(sum(100,200,300));
```
...は可変長の引数とみなされ、指定された引数を配列に格納する  
  
匿名関数
```
let add = function(x: number, y: number): number {
  return x + y;
}
```
変数に直で入れる関数ならば、関数名はいらない

### アロー関数
匿名関数を省略した記法で書くことができる。
```
let add = (x: number, y:number): number => {
  return x + y;
};
```
関数本体の文が一つしかない場合{}を省略できる  
また、その文が戻り値と判断されるので、returnも書かなくて良い  
```
let add = (x: number, y: number): number => x + y;
```
引数も一つしか無く、戻り値の型宣言がいらない場合、引数の()も省略可能   
```
let circle = radius => radius * radius / Math.PI;
```
引数が無い関数では省略はできない 
```
let hello = () => console.log('Hello!');
```

### 関数のオーバーロード
同じ関数名で、引数の型や並びが異なる関数を複数定義すること  
```
function search(str: string, start: number): string;
function search(str: string, start: string): string;
function search(str: string, start: any): string {
  if (typeof start === 'number') {
    return str.substring(start);
  } else {
    return str.substring(str.indexOf(start));
  }
}
let msg = 'いろはにほへとちりぬる'
console.log(search(msg, 3));
console.log(search(msg, 'と'));
```
substringは対象文字列に対して引数で数字をもらった以降の文字列を返す  
searchはstringで引数指定された場合は、indexOfでnumberに変換する  

### 共用型
引数の型宣言をorでできる。  
オーバーロードより読みやすい  
```
function search(str: string, start: number | string): string {
  if (typeof start === 'number') {
    return str.substring(start);
  } else {
    return str.substring(str.indexOf(start));
  }
}
```
  
### クラス

```
class Dog {
constructor(private category: stnring, private weight: number){}
  public toString(): string {
    return this.category + ' : ' + this.weight + 'kg'
  }
}
let mame = new Dog('豆柴', 8);
console.log(mame.toString());
```
引数の頭にアクセス修飾子(private等)を付与することで、内部で変数宣言したのと同じ  
となり、内部に記載する必要がなくなる
```
class Dog {
  private category: string;
  private weight: number;

  public constructor(categoty: string, weight: number) {
    this.categoty = category;
    this.weight = weight;
  }
}
```

アクセス修飾子  
|アクセス修飾子|概要|
|---|---|
|public|どこからでもアクセスできる|
|protected|現在のクラスと、その派生クラスからアクセス可能|
|private|現在のクラスからのみアクセス可能|
指定しないときのデフォルトはpublicとなる

### getter/setter

クラスには、外部から直接値を設定できる(引数に書かれる)変数と、  
直接値を設定できず、クラスに作成されたgetter/setterを通して読み書きする変数がある
前者がpublicプロパティ  
後者がprivateプロパティ    
  
publicプロパティは値の加工や検証、読み書きの制御ができない  
privateプロパティは値の加工や検証、読み書きの制御ができるが、直接アクセスできない  
  
```
class Dog {
  private _weight: number;
  get weight(): number {
    return this._weight;
  }

  set weight(value: number) {
    if (value < 0) {
      throw new Error('weightプロパティは整数で指定してください');
    }
    this._weight = value;
  }
}
let poodle = new Dog();
poodle.weight = -13;
//エラーがでて怒られる
```
getterはprivateな変数を返すだけ  
setterは引数で変数を受け入れ値の検証を行い、privateな変数に変更を加えている  

### クラスの継承

```
class Dog {
  constructor(protected category: stirng, protected weight: number){}
  toString(): string {
    return this.category + ' : ' + this.weight + 'kg'
  }
}
class UltraDog extends Dog {
  toString(): string {
    return 'スーパー' + super.toString();
  }
}
let dober = new UltraDog('ドーベルマン', 35);
console.log(dober.toString());
```

### インターフェース

```
interface Pet {
  name: string;
  call(): void;
}
class Dog implements Pet {
  name: string;
  call(): void {
    console.log(this.name + 'ちゃん！');
  }
}

let pochi = new Dog();
pochi.name = 'ぽち';
pochi.call();
```
クラスを作るときに、この変数(プロパティ)を使えと指示するような機能  
このインターフェースを使うクラスはみな同じプロパティを使っているという共通をもてる  
  
型注釈としてのインターフェース  
  
インターフェースで型宣言しておけば、適用先のクラスで型宣言がいらなく、可読性があがる？  
```
interface Member {
  name: string;
  greet: void;
}
let mem: Member = {
  name: '山田太郎',
  greet() {
    console.log('こんちわ！、' + this.name);
  }
};
mem.greet();
```
変数だけでなく関数も宣言できる
```
interface AddFunc {
  (a: number, b: number): number;
}
let add: AddFunc = function (x: number, y: number): number {
  return x + y;
};
```
  
オブジェクト型リテラル  
  
その場限りの型でインターフェースまではいらないが、型宣言はしたい場合に使う  
```
let mem1: {name: string, age: number} = {name: '山田太郎', age: 18};
let mem1: {name: string; age: number} = {name: '山田太郎', age: 18}; //セミコロンでもよい

let mem2: {name: string, age: number} = {name: '山田太郎', age: ''}; //エラーがでる
```


