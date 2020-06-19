## slackbotを作る

無料で作れるので、slackのworkspaceを作成する  
メールアドレスが必要。適当にchannelを作成する  
作成が終了したらブラウザでそのまま見れる  
ボットが作れるAppsは種類がある

### Botの種類
Slackのモジュールの種類  
  
◆Bots  
「App」を選択し、検索フォームで「bots」で検索すると別タブが開く  
検索結果から「Bots」を選択、botに名前を付けるようナビされる  
名前を付けて「addSlack」するとslackのタブで「命名した名前」でbotsが追加される  
```
アプリを検索する > カスタムインテグレーション > Bots > 設定を編集する
```
「インテグレーションの設定」の「APIトークン」をメモする
  
◆Hubots  
Slack側の準備
「App」を選択し、検索フォームで「hubots」で検索すると別タブが開く  
検索結果から「Hubots」を選択、botに名前を付けるようナビされる  
名前を付けて「addSlack」するとslackのタブで「命名した名前」でHubotが追加される  
  
Slackの左ペインから「チャンネルを追加する」で作成し、「アプリを追加する」で命名したhubotを追加する  
  
### 開発環境側の準備
  
git, vscode, node, yarnは準備済みとする  
yarn-hubot1 などでディレクトリを作り、そこを使う
```
npm install -g hubot coffee-script yo generator-hubot
yo hubot
// 一度npm root NODE_PATHがマッチしないで怒られたが、windowsの環境変数から
//「%NODE_PATH%」を消してvscodeごとターミナルを再起動したら通った
// npmのrootpath と printenvのNODE_PATHがあってればよいらしい
// 対話セットアップは基本YでOKだが、名前と掘ったDirは一致してる必要がある
// また、連携先はslackを指定する
// 対話がおわると構成ファイルが自動で配置される
touch scripts/hello.js
// scriptsは自動で掘られてるはず
```
hello.js
```
'use strict';
module.exports = (robot) => {
    robot.hear(/hello>/i, (msg) => {
        const user_id = msg.message.user.id;
        msg.send(`Hello, <@${user_id}>`);
    });
};
```
実行してみる
```
bin/hubot  
いろいろメッセージがでるが、Enterでプロンプトが出れば動いている
slack-hubot1>
入力待ちなので、トリガーとなる「hello>」をいれてみる
slack-hubot1> Hello, <@1>
上記がでれば成功
```
slackで実行してみる
```
env HUBOT_SLACK_TOKEN=「hubotのapitoken」 bin/hubot --adapter slack
// この場合、slackに接続するのでプロンプトは出ない
slackでbotのいるチャンネルでトリガーとなる「hello>」と発言すると応答が返ってくれば成功
```
  
### 処理をモジュール化する
  
npmモジュール化することでメンテナンスが楽になる

モジュール化した処理の正常性を確認する手段として、要件・CRUD・フローを使用する  
◆要件  
仕様のこと。例としてToDoをしゃべるBotを作るので、  

* 「Bot名 todo XXする」という入力で「XXする」というタスクを作成
* 「Bot名 done XXする」という入力で「XXする」というタスクを完了状態にする
* 「Bot名 list」という入力で未完了タスクを表示
* 「Bot名 donelist XXする」という入力で完了したタスク一覧を表示
    
◆CRUD(クラッド)  
Create Read Update Deleteという情報を保存するアプリケーションの動作の種別を表す  
要件の内容を上記に分類して不足が無いかを確認することで欠陥を探せる  
  
◆フロー  
オブジェクトのライフサイクルともいう。  
作成したtodoタスクに状態を付与したり、削除ができる仕組みが無いと困るので  
CRUDで作成した項目を図にして矛盾がないか確認する  
  
以上に注意してアプリを作成する  
  
### npmパッケージの作成

ディレクトリの作成
```
mkdir ~/work/space/todo
cd ~/workspace/todo
yarn init
```
init後のコマンド
```
name: (todo)
version: (1.0.0)
description: 
entry point: (index.js)
reposiory url: 
author:
license: (MIT) -> ISC
private:
```
実行で作成されたpackage.jsonに追記をする
```
{
  "name": "todo",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "node test.js"
  },
  "license": "ISC"
}
```
追記することでtestできる状態になる  
ファイルを作成する  
touch index.js  
touch test.js  
  
ここでpackageとしての設定が終了  

### コーディング

#### todo の実装
  
index.js
```
'use strict';
// 連想配列をMapで作成
// key: todoの内容(string) value: 完了しているかどうか(Boolean)
const tasks = new Map();

/**
 * TODOを追加するfunction
 * @param {string} task
 */
function todo{task} {
    tasks.set(task, false);
}
// 外部から実行を受け付ける関数はここに記載する
module.exports = { todo };

```
  
コメントはこの記法(/** * */)で書くとVSCode内でカーソルを当てたときに  
ポップアップで表示される  
  
#### list の実装

プログラムを書いて動かしながら確認していく  
index.js
```
'use strict';
// 連想配列をMapで作成
// key: todoの内容(string) value: 完了しているかどうか(Boolean)
const tasks = new Map();

/**
 * TODOを追加するfunction
 * @param {string} task
 */
function todo{task} {
    tasks.set(task, false);
}
/**
* タスクと完了したかどうかが含まれる配列を受け取り、完了したかを返す
* @param {array} taskAndIsDonePair
* @return {boolean} 完了したかどうか
* jsではエラーで無いものはtrueな為、これでtodoの要素１にtrueにする処理となる
*/
function isDone(taskAndIsDonePair) {
  return taskAndIsDonePair[1];
}

/**
* タスクと完了したかどうかが含まれる配列を受け取り、完了していないかを返す
* @param {array} taskAndIsDonePair
* @return {boolean} 完了していないかどうか
*/
function isNotDone(taskAndIsDonePair) {
  return !isDone(taskAndIsDonePair);
}

/**
* TODOの一覧の配列を取得する
* @return {array}
*/
function list() {
  // Map tasks を配列に変換[[key,value],[key,value],[...]] して
  // filterは戻り値がtrueに該当したものだけ返すので、
  // ES6のアロー関数の書き方で最後の値がreturn文の値になる特徴を利用し
  // isNotDone関数引数でvalueがtrueの配列をフィルターし、
  // falseの配列をmap関数に渡して要素[0]のみのMapを作っている
  // 結果的に未達成todoを取得することができる
  return Array.from(tasks)
    .filter(isNotDone)
    .map(t => t[0]);
}

// 外部から実行できる関数にlistを追加する
module.exports = { todo, list };

```
  
#### todoとlistのテストコードを書く
index.jsを動作させるコードとしてtest.jsを書き、呼び出して使用する  
まずはnodejsの動作とnpmモジュールが呼び出せるかを確認で最小のテストを書く  
  
test.js
```
'use strict';
const todo = require('./index.js');
console.log('テストが正常に完了しました');
```
  
consoleで `yarn test` を実行してみる  
エラーが出ず、`Done in 0.25s.` とか表示されればOK  
  
次にモジュール内で公開されている関数を呼び出してみる  
  
test.js
```
'use strict';
const todo = require('./index.js');
const assert = require('assert');

// todoとlistのテスト
todo.todo('ノートを買う');
todo.todo('鉛筆を買う');
// テストコマンド
// 上で作成した内容と中身が一緒であれば成功する *1
assert.deepEqual(todo.list(),['ノートを買う','鉛筆を買う']);

console.log('テストが正常に完了しました');

// *1
// jsでは 1==1 はtrueだが[1]==[1]は false になる
// オブジェクトや配列が == で true になるにはそのオブジェクト自身以外に無い
// 例の配列は中身は同じだが、配列としては別の存在同士なのでfalsとなる
// deepEqual を使用すると、オブジェクトの中身の値を比較できる
```
`yarn test` で実行し、エラーが出なければ成功  

#### doneとdonelistの実装
index.jsにさらに機能を追加する  
(todo, listのコードと、module.exportsの間に記載していく)
```
/**
* TODOを完了状態にする
* @param {string} task
*/
function done(task) {
  if (tasks.has(task)) {
    tasks.set(task, true);
  }
}

/**
* 完了済みのタスクの一覧の配列を取得する
* @return {array}
*/
function donelist() {
  return Array.from(tasks)
    .filter(isDone)
    .map(t => t[0]);
}
// doneとdonelistに作成内容を追加
module.exports = {
  todo,
  list,
  done,
  donelist
};
```
#### done,donelistのテストを書く
test.jsに追記する
```
'use strict';
const todo = require('./index.js');
const assert = require('assert');

// todo と list のテスト
todo.todo('ノートを買う');
todo.todo('鉛筆を買う');
assert.deepEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);

// done と donelist のテスト
// todoのうち片方をdoneする
todo.done('鉛筆を買う');
// listにdoneしてないtodoがあれば成功
assert.deepEqual(todo.list(), ['ノートを買う']);
// donelistにdoneしたtodoがあれば成功
assert.deepEqual(todo.donelist(), ['鉛筆を買う']);

console.log('テストが正常に完了しました');
```

#### delの実装
index.jsにさらに機能を追加する  
(donelistのコードと、module.exportsの間に記載していく)
```
/**
* 項目を削除する(Maptaskから)
* @param {string} task
*/
function del(task) {
  tasks.delete(task);
}
// exportsにdelを追加する
module.exports = {
  todo,
  list,
  done,
  donelist,
  del
};
```

