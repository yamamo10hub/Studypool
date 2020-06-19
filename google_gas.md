[[_TOC_]]


## GASとは

GoogleAppScriptはjavascriptベースでgoogleのサービスを操作できるプログラム  

## セットアップ（初級）
chromeの拡張でgoogleappscriptがあるので入れておく  
googleアカウントを作成する  
  
スプレッドシートを開き、「ツール」「スクリプトエディタ」を選択するとコードエディタが開く  
ドライブで新規ファイルでgooglescriptを選択してエディタを開くこともできる  
  
前者をコンテナバインドスクリプト、後者をスタンドアロンスクリプトと呼ぶ  
  
コンテナバインドは外部サービスとの連携でメリットがあるらしい  
(この場合スプレッドシートと連携しやすくなっている)  
  
<h3>セットアップ その２(中級)</h3>  
GASはブラウザ上のエディタで打ち込む必要がある  
ローカルで書くこともできないし、gitで管理もできない  
以下で環境構築すると、ローカルで使えるようになる  
  
GSuiteDevelopperHubへアクセス(ログイン済みのはず)  
「設定」Google Apps Script APIを有効にする  
  
node.jsをローカルにインストールしておく  
  
<a href="https://github.com/google/clasp">github_clasp</a>  
<a href="https://codelabs.developers.google.com/codelabs/clasp/#0">claspハンズオン</a>  

```
npm i @google/clasp -g    
clasp login  
ブラウザでログインを要求されるのでログインし、claspが使用することを許可する  
成功すると ~/.clasprc.json ができる  
clasp logout するとこのファイルは消える  
```
  
新規作成する場合
```  
clasp create "名前" --type sheet  
既存のプロジェクトを継承する場合  
clasp clone <スクリプトID>  
IDは既存スクリプトエディタの「ファイル」「プロパティ」で確認できる  
```
  
編集したファイルをpushする  
```
clasp push  
```
  
一度cloneしてから再度大元からファイルを取得したい  
```
clasp pull
```  
  
編集と作成の注意  
```
cloneしても .clasprc.json が作成されるので、Dirを移動することはできない  
うまくいかないときはlogin/logoutなど繰り返してみると改善するかもしれない  
```
  
<h3>セットアップ その３(上級)</h3>

その２より良い作成環境を作る場合に実施する  
GASで使用できるjavascriptはES6より古く、新しい記述方法の恩恵を受けられない  
※一部classが作れない、等  
新しいバージョンやtypescriptで書きたい場合、npmモジュールの追加インストールなどが必要  
  
一応claspをupdateした
```
which clasp
npm install -g npm
npm ls -g @google/clasp
npm udpate -g @google/clasp
```

プロジェクトの作成とローカルパッケージのinstall
```
# create project
mkdir new-project
cd new-project
clasp create project-name --rootDir ./src
  選択内容は、どのみちspreadsheetはIDで書くのでstandaloneでいいかと
  ./srcが作成される
# package local install
npm init -y
npm install @google/clasp tslint -D
npm install @type/google-apps-script -S
tslint --init
```

環境がwindowsでgitbashなので、localpackageが使えるようにする
```
gitbash起動
vim ~/.bashrc
 export PATH=$PATH:./node_modules/.bin
gitbash閉じて再起動しておく
一度だけ表示がでるが気にしない
```

Code.gsをリネームし、Code.tsにしておく。  
これによってclasp pushした時に、記述した言語から動作可能言語へトランスパイルされる  
  



## コードについて
* {}をブロックと呼び、 ; で終わる一行をステートメントと呼ぶ  
* 保存は自動ではないので注意  
* コードはプロジェクトの一部として保存されるので、プロジェクト名をつける必要がある  
* defaultで書かれているmyFunctionの通り、関数を起点として実行される  
  
サンプルスクリプト
```
function myFunction() {
  Logger.log("Hello World!");
}
```
Logger.logは、ログに引数を記載する関数  
  
◆実行後のログ  
「表示」「ログ」を選択するとログが無いとでるが、下部のAppscriptダッシュボードのリンクを選択すると  
GSuiteDevelopperHubの画面に飛ばされるが、実施したログが見れる  

## 変数
letが使えないのでvarで宣言する必要がある  
バッククォートも使えないので、 " か ' を使う  
  
テキストへの埋め込み書式  
  
変数の中身をテキストに埋められる
```
function myFunction() {
  var x = 10, y = 5;
  Logger.log('%s と %s の和は %s です。', x, y, x + y);
  
  var msg = 'Hello GAS!';
  Logger.log('GASによるメッセージ: %s', msg);
}
```
シェルっぽい書き方  
  
## アプリケーションと承認

スプレッドシート画面にメッセージをモーダルで表示する
```
function myFunction() {
  Browser.msgBox('Hello GAS!');
}
```
実行しようとすると確認されていないアプリとして、承認を行うよう表示される  
Gsuiteでないgmail.comのアカウントでは、承認以外にブロック画面が表示されるので、  
左下の詳細から実行するを選択する必要がある  
  
## GASにおけるオブジェクト

各サービスの操作対象とするモノ  
* スプレッドシート：Spreadsheetオブジェクト  
* シート：Sheetオブジェクト  
* セル範囲：Rangeオブジェクト  
  
◆シートの値をとるにはどうすればいいか  

サンプルのシート
```
シート名：cs_list
シートページ：cs_view
circle name	day	position	menu	price	kind	notice	check_flag
hoge1	1	45ab	wall	3000	shutter	wait long
hoge1	1	45ab	book1	1000	shutter	goodsset
hoge1	1	45ab	goods1	8000	shutter	makura
fuga1	3	14ab	book1	500	shutter	5gen
fuga1	3	14ab	book2	1000	shutter	1gen
fuga1	3	14ab	wall	4000	shutter	sukunai
fuga1	3	14ab	card	3000	shutter	sukunai
```
サンプル仕様  
* リストの内容をランダムで一つ表示する  
* 一度表示した内容はcheck_flagにokを入力する  
* すべてのcheck_flagにokがあれば、それらを削除する    
  
スプレッドシート「cs_list」のシート「cs_view」のA3セルの値を取得したい、とする  
1. スプレッドシート「cs_list」をSpreadsheetオブジェクトとして取得する
2. そのSpreadsheetオブジェクトの配下にあるシート「cs_view」をSheetオブジェクトとして取得する
3. そのSheetオブジェクトの配下にあるA3セルをRangeオブジェクトとして取得する  
4. そのRangeオブジェクトの値を取得する  
  
以下、段階的にやり方を調べていく  

Activeなスプレッドシートを取得する
```
function myFunction() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  Logger.log(ss.getName());
}
```
現在のスプレッドシートを変数ssに取得する  
スプレッドシートの名前をログに書き込む、となる  
注意として上記でスプレッドシートが取得できるのはコンテナバインドスクリプトのみとなること  
  
ActiveなスプレッドシートからActiveなシート名を取得する
```
function myFunction() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('cs_view');
  Logger.log(sheet.getName());
}
```
シート名の指定が必要なので、名前は慎重に決めなければならない  
  
コンテナバインドスクリプトであればアクティブなシートを取得するメソッドがあり、  
名前を指定せずに取得できる。  
```
function myFunction() {
  var sheet = SpreadsheetApp.getActiveSheet();
  Logger.log(sheet.getName());
}
```
複数のシートを使う場合、Activeなシートのコントロールが必要になるので注意  
  
セルや範囲を取得する  

まず単体のセルを取る方法
```
function myFunction() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var range = sheet.getRange('A3');
  Logger.log(range.getValue());
}
```

次に複数のセルを取る方法
```
function myFunction() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var range = sheet.getRange('A3:G3');
  Logger.log(range.getValues());
}
```
getValues()では値が配列に入った状態で戻る  
同じ行数であれば同じ配列要素に入り、異なる場合は配列要素が異なる  

## loggingについて

スクリプトエディタの「表示→ログ」にログを出したい
```
  Logger.log(list);
```

log用SpreadSheetをつかったロギング
```
var log_ss = SS.getSheetByName("ログ");

function logging (str) {
  ts = new Date().toLocaleString('japanese', {timezone: 'Asia/Osaka'});
  log_ss.appendRow([ts, str]);
}

var hoge = 'fugafuga'
logging(JSON.stringify(hoge));
```
ログという名前のワークシートを作っておくと、そこにログが書き込まれる。  
ただし、以下のようにオブジェクトになる変数はそのまま出力できない。
```
var list = gomi_ss.getDataRange().getValues();
logging(JSON.stringify(list));
```
上記の場合は、素直にLogger.logで確認するか、JSONに変換するとシートに出力できる。
```
 logging(JSON.stringify(list));
 //Logger.log(list);
```

## 制御構文
for文
```
function myFunction() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var lastRow = sheet.getLastRow();
  for(var i=2; i<=lastRow; i++) {
    Logger.log(sheet.getRange(i,1).getValue());
  }
}
```
入力が存在する末尾行までループし、一列目の情報をログに書き込む
  
if文
```
function myFunction() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var lastRow = sheet.getLastRow();
  for(var i=2; i<=lastRow; i++) {
    //if(sheet.getRange(i, 8).getValue() === ''){ 
    if(!sheet.getRange(i,8).getValue()){
      Logger.log(sheet.getRange(i,1).getValue());
    }
  }
}
```
入力が存在する末尾行までループし、８列目が空白のセルでなければログにその行の内容を書き込む  

### シートへの入力

setValueを使うとシートに入力できる
```
function myFunction() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var lastRow = sheet.getLastRow();
  for(var i=2; i<=lastRow; i++) {
    if(!sheet.getRange(i,8).getValue()){
      Logger.log(sheet.getRange(i,1).getValue());
      sheet.getRange(i,8).setValue('ok');
      break;
    }
  }
}
```
ループにsetValueつけても使いづらいこともある  
そうゆう場合、getter,setterメソッドを用意する  
  
breakを抜くと、8列目が空白なセルすべてがログに出る  
  
### 対象列に対しての範囲的情報取得

```
Sheetオブジェクト.getRange(行番号,列番号,行数,列数)
```
列数はデフォルトで１かな  
  
４列目の２行目から入力のある最後の行までのセル情報を取得する場合
```
  if(i >= lastRow){
    Logger.log(sheet.getRange(2,4, lastRow - 1).getA1Notation());
  }
```
getA1Notation() : A1形式でセル範囲を取得するメソッド  
ログを見るとどの範囲が取れたのか確認できる  
  
### 対象セル範囲をクリアする方法
* すべてをクリアする　　：Rangeオブジェクト.clearContent()  
* 指定範囲をクリアする　：Rangeオブジェクト.clearContent()    
* データの入力規則をクリア：Rangeオブジェクト.clearDataValidations()  
* コメントのクリア　　　：Rangeオブジェクト.clearNote()  
  
前項で範囲が取得できているので、指定範囲のクリアを使うと消せそう  
```
function myFunction() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var lastRow = sheet.getLastRow();

  for(var i=2; i<=lastRow; i++) {
    //if(sheet.getRange(i, 8).getValue() === ''){ 
    if(!sheet.getRange(i,8).getValue()){
      Logger.log(sheet.getRange(i,1).getValue());
      sheet.getRange(i,8).setValue('ok');
      break;
    }
  }
  
  if(i >= lastRow){
    //Logger.log(sheet.getRange(2,8, lastRow - 1).getA1Notation());
    sheet.getRange(2,8, lastRow - 1).clearContent();
  }
}
```

ランダムを実現する
```
  var row = Math.floor(Math.random() * 6);
  var todo = sheetdata[row][0];
  //Logger.log(row);
```
ランダムな小数に項目数を掛けると、整数値がどれかを指す



