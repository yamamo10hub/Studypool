## LineとGASでchatbotつくりたい

以下を確認し、コンテナバインドスクリプトとextrasetupまでしておく  
<a href="https://gitlab.com/yamamo10/basictech-via-doc/wikis/google_gas">GAS</a>  
gitを使う場合scriptIDがあるので、privateにする  
  
以下を設定しておく  
<a href="https://gitlab.com/yamamo10/basictech-via-doc/wikis/WebAPI_Line">Line側の設定</a>
webhookURLは後述  
  
### postとpush
Line側では上記２種類のメッセージAPIがある  
```
postはLineからwebhook先へ  
pushはwebhook先からLineへ  
```
それぞれサンプルを動かしてみる  
  
### スクリプトプロパティ
そのGASプロジェクト上での環境変数  
コードにAPIを直書きするとgitの公開制限が必要になる  
公開して運用したい場合、こちらを使う  
  
コードエディタから「ファイル」「プロジェクトのプロパティ」「スクリプトのプロパティ」で  
任意のIDと値を投入する
  
コード上でスクリプトプロパティは次のように呼び出す  
```
PropertiesService.getScriptProperties().getProperty('プロパティのID');
```
直書きの値の代わりに上記を記載する  


### postのサンプル
Lineへ送信した内容をそのまま返すpost

<details>

<summary>サンプルコード</summary>

```
//LINE Developersで取得したアクセストークンを入れる
var CHANNEL_ACCESS_TOKEN = PropertiesService.getScriptProperties().getProperty('LINE_CH_TOKEN');
var line_endpoint = 'https://api.line.me/v2/bot/message/reply';
var SS = SpreadsheetApp.openById(PropertiesService.getScriptProperties().getProperty('SHEET_ID'));

//SpreadsheetのURL
var data_ss = SS.getSheetByName("りすと"); 

//ポストで送られてくるので、送られてきたJSONをパース
function doPost(e) {
  var json = JSON.parse(e.postData.contents);
  // logが取れないので、sheetに書きだす
  for(var i=2; i<=100; i++) {
    if(!data_ss.getRange(i,1).getValue()){
      data_ss.getRange(i,1).setValue(json);
      break;
    }
  }
  //返信するためのトークン取得
  var reply_token = json.events[0].replyToken;
  if (typeof reply_token === 'undefined') {
    return;
  }
  //送られたメッセージ内容を取得
  var message = json.events[0].message.text;  
  // メッセージを返信    
  UrlFetchApp.fetch(line_endpoint, {
    'headers': {
      'Content-Type': 'application/json; charset=UTF-8',
      'Authorization': 'Bearer ' + CHANNEL_ACCESS_TOKEN,
    },
    'method': 'post',
    // JSONを作成している
    'payload': JSON.stringify({
      'replyToken': reply_token,
      'messages': [{
        'type': 'text',
        'text': message,
      }],
    }),
  });
  return ContentService.createTextOutput(JSON.stringify({'content': 'post ok'})).setMimeType(ContentService.MimeType.JSON);
}
```

</details>

### ログを取得する
GASではconsole.logやprintができないので、動作確認がタイヘン  
しかし、シートに書き出すことでコンソールの代わりにできる  

```
// logging用のシートを用意しておく
var log_ss = SS.getSheetByName("ログ");
// logging用メソッド
function logging (str) {
  ts = new Date().toLocaleString('japanese', {timezone: 'Asia/Osaka'});
  log_ss.appendRow([ts, str]);
}

logging(対象の変数やオブジェクト);
```

### LineからDoPostに来る情報について
```
{
"parameter":{},
"contextPath":"",
"contentLength":289,
"queryString":"",
"parameters":{},
"postData":{
  "type":"application/json",
  "length":289,
  "contents":"{
    "events":[{
      "type":"message",
      "replyToken":"9547e0dff63f49999999999999999999",
      "source":{
        "userId":"U6682c9ef590999999999999999999999",
        "type":"user"
        },
      "timestamp":1564625800744,"message":{
        "type":"text","id":"10313050999999","text":"ほげ"
        }
      }],
    "destination":"U8ecbb5f07f0dc9619999999999999"
    }",
  "name":"postData"
  } //ここまでPostData (type, length, contents, name)
} // parameter, contextPath, contentLength, queryString, parameters, postData, name
```
  
実質使用する情報はpostDataの中になるので、必要情報をトリミングすると以下のようになる
```
var json = JSON.parse(e.postData.contents);
```
  
```
"contents":"{
  "events":[{
    "type":"message",
    "replyToken":"9547e0dff63f49999999999999999999",
    "source":{
      "userId":"U6682c9ef590999999999999999999999",
      "type":"user"
      },
    "timestamp":1564625800744,"message":{
      "type":"text","id":"10313050833398","text":"ほげ"
      }
    }],
  "destination":"U8ecbb5f07f0dc9619999999999999"
}"
```
  
### lineに返信するために必要な情報

urlfetchを使うと、HTTPリクエストを行うことができる
```
UrlFetchApp.fetch(line_endpoint, {
  'headers': {
    'Content-Type': 'application/json; charset=UTF-8',
    'Authorization': 'Bearer ' + CHANNEL_ACCESS_TOKEN,
  },
  'method': 'post',
  // JSONを作成している
  'payload': JSON.stringify({
    'replyToken': reply_token,
    'messages': [{
      'type': 'text',
      'text': message,
    }],
  }),
});
return ContentService.createTextOutput(JSON.stringify({'content': 'post ok'})).setMimeType(ContentService.MimeType.JSON);
}
```
第一引数 が URL, 第二引数はパラメータ
  
<details>

<summary>サンプルコード</summary>

```
//LINE Developersで取得したアクセストークン
var CHANNEL_ACCESS_TOKEN = PropertiesService.getScriptProperties().getProperty('LINE_CH_TOKEN');
var line_pushmsg  = 'https://api.line.me/v2/bot/message/push';

//SpreadSheetのURL
var SS = SpreadsheetApp.openById(PropertiesService.getScriptProperties().getProperty('SHEET_ID'));
var data_ss = SS.getSheetByName("りすと"); 
var log_ss = SS.getSheetByName("ログ"); 

//log用SpreadSheetをつかったロギング
function logging (str) {
  ts = new Date().toLocaleString('japanese', {timezone: 'Asia/Osaka'});
  log_ss.appendRow([ts, str]);
}

//push送信のお試し
var USER_ID = PropertiesService.getScriptProperties().getProperty('USER_UKI');
function pushMessage() {
  var postData = {
    'to': USER_ID,
    'messages': [{
      'type': 'text',
      'text': 'コニチワー',
    }]
  };

  var url = line_pushmsg;
  var headers = {
    'Content-Type' : 'application/json; charset=UTF-8',
    'Authorization': 'Bearer ' + CHANNEL_ACCESS_TOKEN,
  };
  var options = {
    'method' : 'post',
    'headers': headers,
    'payload': JSON.stringify(postData) 
  };
  var response = UrlFetchApp.fetch(url, options);
}
```

</details>


