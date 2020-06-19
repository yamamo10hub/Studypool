[[_TOC_]]

## Node.jsを試す
Angularjsの時にインストールをしている  
またAngularjsを試す前にexpressを少し試していたが、メモを紛失したので  
N高プログラミング学習コンテンツを写経するついでに書いていく  
  
## setup
Angularjsの項目を参照だが、簡単な概要として書けば以下となる
```
windowsのインストーラーがあるのでインストール
環境変数設定
gitbashで使用
必要に応じてパッケージ管理も入れる(npm,nvm,yarn)
```
node -v できればOK
  
## サンプルプログラムの実行

１つのサンプルを作成し、流れを確認する
node.jsでは任意のディレクトリをそのプログラムのworkdirとして、関連するファイルを  
全てworkdir配下に置くのが定石となっている
```
mkdir nk-nodejs-01
cd nk-nodejs-01
touch app.js
```
  
app.js
```
//デバッグの厳格モード
'use strict';

// process.argvはプリセットされているコマンドライン引数だが、
// 0はnodeコマンドのfilepath, 1は実行プログラムのfilepathが入っており
// 2にコマンドライン引数が入っている
// ||をつけると左辺がfalse的な値の時に0を代入するようにできる(null, undefined,0等)
const number = process.argv[2] || 0;
let sum = 0;
for (let i = 1; i <= number; i++) {
    sum = sum + i;
}
console.log(sum);
```
  
実行してみる
```
node app.js 100
```

__assertによるテストコード__  
階乗を計算するプログラムでテストコードを確認する

```
'use strict';
/**
* 与えられた自然数の階乗を返す
* 階乗とは、1からその与えられた自然数までの数をすべてかけたものです
* @param {Number} n
* @returns {Number}
*/
function factorial(n) {
  let result = 1;
  for (let i = 1; i <= n; i++) {
    result = result * i;
    console.log(result);
  }
  return result;
}
const assert = require('assert');
assert.equal(factorial(1), 1, `1の階乗は1ですが、実際は${factorial(1) }でした`);
assert.equal(factorial(2), 2, `2の階乗は2ですが、実際は${factorial(2) }でした`);
assert.equal(factorial(3), 6, `3の階乗は6ですが、実際は${factorial(3) }でした`);
assert.equal(factorial(10), 3628800, `10の階乗は3628800ですが、実際は${factorial(10) }でした`);
console.log('すべてのテストを通過しました');
```

## コード内容と実行速度
新しいサンプルとして、フィボナッチ数列の計算表示というサンプルを作成する  
sample-fibo.js
```
'use strict';
// フィボナッチ数列を40番目まで出力する
// 指定数の-1した値と、-2した値を足したもの
function fib(n) {
    if (n === 0) {
        return 0;
    } else if (n === 1) {
        return 1;
    }
    return fib(n - 1) + fib(n - 2);
}
const length = 40;
for (let i = 0; i <= length; i++) {
    console.log(fib(i));
}
```
  
関数の中で自分自身を呼んでいて奇妙だが、ちゃんと動くらしい  
これを再帰呼び出しという  

普通に実行する  
```
node sample-fibo.js
```
  
時間が計れる。その場合は以下のように実行する
```
time node sample-fibo.js
```
末尾に計測結果が表示される
```
real    0m5.049s
user    0m0.000s
sys     0m0.015s
```
  
__遅くなるプログラムとは__  
sample-fibo.jsを見ると変数に足し算の結果を延々と追加していくプログラムだが、  
一回の関数起動で再帰により２回呼び出されるので、自乗されるため100回程度で一日で終わらなくなる  
  
自乗程度に処理回数が増えるものを指数オーダーと呼ぶ  
  
__指数オーダーの確認__
実行したプログラムの分析を行う機能がある  
```
オプションをつけて実行する
node --prof app.js
isolateから始まるログファイルが生成されるので、特定コマンドと合わせて結果を表示する
node --prof-process isolate-0000013D5F82AC20-20636-v8.log
```
結果表示でどこをみるか
```
// 実行プログラム割合
 [Summary]:
   ticks  total  nonlib   name
   2524   89.1%   91.9%  JavaScript
      0    0.0%    0.0%  C++
      1    0.0%    0.0%  GC
     88    3.1%          Shared libraries
    221    7.8%          Unaccounted
// 時間かかった処理時間順
// *fib はプログラム内の関数
 [Bottom up (heavy) profile]:
   ticks parent  name
   2524   89.1%  LazyCompile: *fib D:\js-dojo\nk-nodejs01\sample-fibo.js:4:13
   2524  100.0%    LazyCompile: *fib D:\js-dojo\nk-nodejs01\sample-fibo.js:4:13
   2524  100.0%      LazyCompile: *fib D:\js-dojo\nk-nodejs01\sample-fibo.js:4:13
   2524  100.0%        LazyCompile: *fib D:\js-dojo\nk-nodejs01\sample-fibo.js:4:13
   2524  100.0%          LazyCompile: *fib D:\js-dojo\nk-nodejs01\sample-fibo.js:4:13
   2524  100.0%            LazyCompile: *fib D:\js-dojo\nk-nodejs01\sample-fibo.js:4:13
```
__アルゴリズムの改善__
作成したfib関数が遅いことが調査でわかった  
これを言い換えると、「指数オーダーを含んでいるfib関数は、実行速度が遅いアルゴリズムを使用している」となる
  
毎回足し算しないアルゴリズムに変更することを考える  
今回はMapというHashではないHashのようなjavascriptの機能をつかって、足し算結果を配列にいれることで、  
ループによる足し算数が増加するアルゴリズムを改善する  
fib関数の下に追加する
```
const memo = new Map();
memo.set(0,0);
memo.set(1,1);
function fibmap(n) {
    if(memo.has(n)) {
        return memo.get(n);
    }
    const value = fibmap(n - 1) + fibmap(n - 2);
    memo.set(n, value);
    return value;
}
```
実行部分を書き換える
```
const length = 40;
for (let i = 0; i <= length; i++) {
    //console.log(fib(i));
    console.log(fibmap(i));
}
```

実行してみるととても速い。アルゴリズムが改良された。  

## サンプルその３

githubで既にあるコードを取ってきていじくる方法  
* 自分のgithubでログインする
* 対象のgithubもURLへ移動する
* forkをクリックするとコピーが始まる
* コピーができた自分のGitのURLでローカルにcloneする
  
CSVは以下の内容が,区切りで県別に入力されている
```
集計年,都道府県名,10〜14歳の人口,15〜19歳の人口
2010,北海道,237155,258530
2015,北海道,217089,236840
各県分データが存在
```
  
CSVからデータを取り出すプログラムを作る
```
'use strict';
// requireでモジュールを呼んでいる
// file の取り扱いと 読み込みのモジュールを呼んでる
const fs = require('fs');
const readline = require('readline');
// Streamとはデータを流す動作を表す。(１文字ずつループ文とか効率悪いので)
// ファイルとして取り込んだ中身を処理するのに必要ということ
const rs = fs.createReadStream('./popu-pref.csv');
// Streamを受け止めるにはinterfaceが必要らしい
const rl = readline.createInterface({ 'input': rs, 'output': {} });
// Interfaceに１行ずつファイルの中身が入るたびに無名関数が起動する
rl.on('line', (lineString) => {
    console.log(lineString);
});
```
１行ずつ読み込む部分に処理をつけてテキスト加工を行う
2010年と2015年の15~19歳の人口だけ取り出して表示する
```
rl.on('line', (lineString) => {
    //console.log(lineString);
    // 配列にする
    const columns = lineString.split(',');
    // yearに年部分、prefectureに件名、popuに15-19の人口を代入
    const year = parseInt(columns[0]);
    const prefecture = columns[1];
    const popu = parseInt(columns[3]);
    // 2010or 2015に該当した時だけ実行
    if (year === 2010 || year === 2015) {
        console.log(year);
        console.log(prefecture);
        console.log(popu);
    }
});
```
  
サンプルを使用して都道府県別の人口変化率を計算させる  
  
CSVは2010と2015でデータがばらけているので集計の為に項目をまとめる必要がある  
どちらの年も県名は存在するので、それをキーとして新しいデータを作る
新しいデータはプログラム内でMapにまとめる
実際のコード
```
'use strict';
const fs = require('fs');
const readline = require('readline');
const rs = fs.createReadStream('./popu-pref.csv');
const rl = readline.createInterface({ 'input': rs, 'output': {} });
// Mapの作成
const prefectureDataMap = new Map(); //key: 都道府県 value: 集計データのオブジェクト
// node.jsのモジュールで、第一引数はEvent
// Eventはあらかじめ決められているのでマニュアルを参照する
rl.on('line', (lineString) => {
    const columns = lineString.split(',');
    const year = parseInt(columns[0]);
    const prefecture = columns[1];
    const popu = parseInt(columns[3]);
    if (year === 2010 || year === 2015) {
        // Map内に今回読み込んだ県名をkeyとしたMapを代入する
        let value = prefectureDataMap.get(prefecture);
        // 中身が空なら(初回なら)0,0,nullを初期値として代入する
        // データには2010と2015があるので、Mapが存在する場合もある
        if (!value) {
            value = {
                popu10: 0,
                popu15: 0,
                change: null
            };
        }
    // 読み込んだデータが2010であればpopu10に代入
    if (year === 2010) {
        value.popu10 = popu;
    }
    // 読み込んだデータが2015であればpopu15に代入
    if (year === 2015) {
        value.popu15 = popu;
    }
    // Mapに代入した内容を反映する
    prefectureDataMap.set(prefecture, value);
});
// 2010と2015をまとめた結果を表示する
rl.on('close', () => {
    console.log(prefectureDataMap);
});
```
実施結果
```
$ node app.js
Map {
  '北海道' => { popu10: 258530, popu15: 236840, change: null },
  '青森県' => { popu10: 67308, popu15: 61593, change: null },
  以下都道府県分の表示
}
```

変化率の計算を行う
```
// 2010と2015をまとめた結果を表示する
rl.on('close', () => {
    // Mapの要素の数だけループし、その要素のkeyとvalueを代入する(分割代入)
    for (let [key, value] of prefectureDataMap) {
        // 人口で除算して変化率を代入する
        value.change = value.popu15 / value.popu10;
    } 
    // 変化率の計算はできた   
    //console.log(prefectureDataMap);
    // さらに表示を変化率でソートしたい
    // Mapを配列化する。[0]には県名、[1]は人口と変化率をハッシュで格納
    // sortはjavascriptの関数で、命名自由な引数２つを指定した無名関数で
    // returnで返す値でsortすることができる
    // 配列を前から処理して大きい値を前にしたい場合、
    // 後の値から前の値を引けばよい
    const rankingArray = Array.from(prefectureDataMap).sort((pair1, pair2) => {
        return pair2[1].change - pair1[1].change;
    });
    //console.log(rankingArray);
    // 配列ではなく、整形して表示する
    // mapは上述のデータ形式ではなく関数名。この関数はデータの要素に対して同じ処理
    // を行う関数でeachに似ている。結果は格納される
    const rankingStrings = rankingArray.map(([key, value]) => {
        return key + ': ' + value.popu10 + '=>' + value.popu15 + '変化率:' + value.change; 
    });
    console.log(rankingStrings);

});
```

## ライブラリを学ぶ
__種類について__

node.jsが使用できるライブラリは複数存在する。
1. JavaScript標準ビルトインオブジェクト
node.jsはJSでできているので使用できる
2. Node.js
node.jsにも標準パッケージ群が存在する
3. yarn
facebookが公開しているnpm互換のパッケージマネージャ
facebookはreactjsというwebフレームワークを作ってるのでパッケージマネージャも自作したと思われる

__パッケージのインストールについて__  
グローバルとローカルの２種類のインストールがあり、  
基本的にローカルインストールを使用する  
  
ローカルにインストールするとgitの範囲内のnode_modulesディレクトリにパッケージが配置されるので  
gitの対象から外さないといけない(無駄なファイルはgitで管理しない)  
.gitignoreファイルのnode_modulesを記載しておくと同期の対象から外れる  

packageの使い方
```
forkする
git clone (https://github.com/yamamo10hub/yarn-training) 
cd yarn-training
yarn install request
ls node_modulus
```

app.js
```
'use strict';
const request = require('request');
request('https://www.google.com', (error, response, body) => {
    console.log(body);
});
```
  
__npmパッケージを作る__  
作成して登録することもできる  
```
mkdir sum
cd sum
yarn init 
touch index.js
```
  
index.js
```
'use strict';
function add(numbers) {
    let result = 0;
    for (let num of numbers) {
        result = result + num;
    }
    return result;
}
// 特定の関数をモジュールとして公開する場合にmodule.exportsオブジェクトの
// プロパティとして関数を登録する
module.exports = {
    add : add
};
```
  
作ったaddを、追加でパッケージを作成して呼び出す  
```
cd ../sum-app  
yarn init  
yarn add ../sum
// pathを指定しないとyarnパッケージ内のaddが呼び出されるので注意
```
  
app.js
```
'use strict';
const s = require('sum');
console.log(s.add([1,2,3,4]));
```

## slackでbotを作る






