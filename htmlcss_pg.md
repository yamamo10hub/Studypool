## HTMLとCSS

Progateの教材でHTMLを学んだ時のノート  
CSSとごっちゃになっている  


### HTMLでよく使うタグ

よく使うタグ集
```
h1,h2,h3　見出し
p　インライン要素の段落
div　ブロック要素の段落
span　
ul li　黒点付きインライン要素リスト
ol li　数字付きインライン要素リスト
table,tr,td　ブロック要素リスト

<a href="https://hogehoge.com/">ほげ</a>　リンク
<img src="https://hogehoge.comf/hogeimg.jpg">　画像
```

HTML5の空ページ
```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Progate</title>
<link rel="stylesheet" href="stylesheet.css">
</head>

</html>
```

### よく使うCSS

特定の要素にだけスタイルを適用する  
その１  
```
h1 {
 /* comment */
 color: #ff0000;
 font-size: 40px;
 font-family: "Lucida Grande";
 background-color: f3f372;
 height: 50px;
 width: 70px;
}
```

その２
```
html
  <li class="selected">HTML</li>
  <li>PHP</li>
  <li>Ruby</li>
css
li {
  color: #444;
}
.selected {
  color: #ff0000;
}
```

余白  
```
.logo1 { padding: 20px; } 上下左右に余白
padding-top , padding-bottom , padding-left , padding-right など個別もある
padding: 20px 10px; 一つ目が上下、二つ目が左右の余白
padding: 20px 10px 20px 10px; 上右下左の余白指定
```

特定の文字にスタイルを適用する  
```
<p>hoge<span>fuga<span>hoge</p>
span{ color: #ff0000; }
など
```

枠線をつける  
```
<div class="sono1">hoge</div>
.sono1 { border: 5px solid red;}
```

border-top, border-bottom, border-left, border-rightなどもある  
色はcolorコードでもよい  
  
枠線外の余白  
marginを使う。書き方はpaddingと同じ
  
余白の構成について  
文字 → padding → border → margin
  
borderを使っていないが文字表示位置をずらしたいときは、marginから使うべき？  

### headerタグとスタイル

html5で追加されたbody内のタグ
```
<header>
<footer>
```

コンテンツ作成において定着した上下に配置するためのコンテンツの識別  
一回しか使っちゃいけないわけではなく、段落やブロックごとに追加してもよい  
タグに対してスタイルの適用もできる  


上記標準で実装されたbody内のタグを使わず、divで作るケースもある  
だいたいヘッダー・メイン・フッターにわかれる  
```
<div class="header"></div>  
<div class="main"></div>  
<div class="footer"></div>  

.header { background-color: #f6f6f6; height: 90px;}
.main { background-color: #ccccff; height: 600px;}
.footer { background-color: #444444; height: 180px;}
```

最近ではページ全体でヘッダーメインフッターは１つではなく、  
各要素ごとにもヘッダーやフッターが存在することもある  
  
ヘッダーについて
```
タイトルとメニュー用のリンクがつけられる。
<div class="header">
   <div class= header-logo> タイトル名　</div>
   <div class="header-list">
     <ul>
        <li>メニュー１</li>
        <li>メニュー２</li>
        <li>メニュー３</li>
     </ul>
</div>

stylesheet
li { list-style: none;
     float: left;
     font-size: 36px;
     padding: 20px 40px;
} /* リストの黒点を消したり、リストを横並びにしたり、項目間の余白を調節したり。 */
```

フッターについて
```
<div class="footer">
   <div class="footer-logo">Progate</div>
      <div class="footer-list">
        <ul>
        <li>会社概要</li>
        <li>採用</li>
        <li>お問い合わせ</li>
        </ul>
      </dev>
   </div>
</div>

stylesheet
//footerと名付けたクラスだけにclassを適用する。
.footer {
background-color: #2f5167;
   color: #fff;
   height: 120px;
   padding: 40px;
}

//このclassのliにだけスタイルを適用する。
.footer-list li {
   padding-bottom: 20px;
}
```

### HTML上の入力コンテンツ

#### フォーム
```
<input type="text/submit/hidden/button/image" name="" id="" value="" />
<textarea name="" id= rows="8" cols="40"></textarea>
```

#### ボタン
```
<input class="contact-submit" type="submit" name="" id="" value="送信" />
```
  
#### メインコンテンツについて
```
letter-spacing: 5px;  
```
  
複数クラスを関連付けるときは、半角スペースを挟む  
```
<a class="btn signup" href="#">新規登録はこちら</a>  
```

#### `<a>` タグでボタンを作る
スタイル使わないとただのリンクになってしまうが、`<a>`タグをブロックインライン要素の  
スタイルを適用することでbackground-colorによって塗り分けてボタンのように表示できる  

```
<div class=btn-wrapper >
   <a class="btn signup" href="#">新規登録はこちら</a>
</div>

.btn-wrapper {  margin: 20px 0;   }
.signup {  background-color: #239b76;   }

border-radius: 4px; でボタンの角を丸めることができる。

line-height: 65px; で<a>タグのリンクの縦の大きさを広げることができる。
paddingだと、文字自体がリンクの範囲なので、大きさが広がらない。
marginはタグ自体の外側の範囲なので、意味がない。
```

### CSSで可能な動き

#### マウスオンで動作を変える。
CSSでできる。CSSではマウスオンは:hoverとして定義されている
```
.btn {  color: white; }
.btn:hover {  opacity: 1;   }
hover状態と
```

#### アニメーション
hover時の変化などを指定の時間をかけるようにできる。
```
div {	transition: all 1s;	}
```

#### リンクへの工夫
対象の要素にカーソルが乗ったときの挙動  
cursor: (text;|pointer;|default;)  
  
対象の要素に影をつける。横範囲、下範囲、影の色を指定  
box-shadow: 0 7px #1a7940;  
  
クリックしたらボタンをへこませる  
```
.target:active {
 box-shadow: none;   //設定されているCSS要素を無効にするnone
 position: relative; //自身を基準位置とする。
 top: 7px;           //自身の位置から7px(つけたbox-shadowの分だけ)要素を移動する。
}
```

#### アイコンをつける
よくサービス名の頭についてるマークはアイコンらしい  
Font Awesomeというサービスがメジャーらしい  
cssを読み込み、決められたclassを指定することで対応する画像を設置できる  
```
<head>
   <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">
</head>
<a href="#" class="btn twitter">
   <span class="fa fa-twitter"></span>
    Twitterで登録
</a>
```

#### 透明度について
対象のブロック要素を透明にするには以下を使うが、文字と背景色両方が透明になる。  
```
opacity: 0.7;  
```
rgbaを使用すると背景色だけ透明化が可能となる。  
rgbアルファチャンネル  
```
background-color: rgb(255, 147, 30, 0.2);
```
数値は1-0の範囲で、0に近づく程透明になる。

#### divのネスト
親divに対して子divを並べることで横に配置できる  
改行やサイズが変わって下段にスライドするのを防ぐために、  
親divを100%として子divを指定の割合で大きさを指定することができる。  
```
.container_child {
div {	width: 25%;	}
}
```

#### 要素同士をCSSを使用して同じ位置に配置する
画像`<img>`の上に文字`<a>`を置きたい、とか
```
.child {
position: absolute; //サイト全体の左上を基準位置とする。
top: 50px;
left: 70px; }

.parent { position: relative; //この要素の子はこの要素を基準位置とする。
}
.child {
position: absolute;
top: 50px;
left: 70px; }
```
親要素がある場合、leftではなくwidth: 100%を指定すると横幅は親要素と同じになる  
text-align: center;など指定してあれば、小要素は親要素上の中心にレイアウトされる  


#### 要素を画面に固定する。
スクロールしても動かないヘッダをつくれる。
```
header {
position: fixed;
top: 0;      //topを0にすることで画面の一番上に固定する。
z-index: 10; //xは左右, yは上下, zは重なりで大きいほど前に表示される
}
```

### レスポンシブデザイン

クライアントの横幅に応じて表示を変更したい  

#### メディアクエリとは
レスポンシブデザインに必要となる  
閲覧端末によって設定を変える機能  
  
stylesheetで設定する  
```
@media(条件) {中身}  
```
  
条件は「max-width」「min-width」が指定できる。  
これでブラウザの横幅に応じた挙動を設定可能となる。  
条件部分は半角スペース  
  
メディアクエリを使うためには、headにviewportの設定が必要  
metaタグで設定する。これによって端末の幅を取得しないと、CSSが動かない  
  
#### 要素の表示サイズ調整
ブラウザに応じて横に並べる要素数を調整する  
width: 100%  
width: 50%  
など。  
サイトを作るとき横幅は％で書かないと柔軟なサイズ変更ができなくなる  
  
また、ブロック要素のサイズを％で書いている場合は、その要素にmax-widthもつけておかないと  
横幅が想定以上に広いクライアントの場合、表示が間延びしてしまう  
  
しかし、paddingを設定している場合、padding分が勘定されないため、  
100%を超えた最後の要素が意図せず下段に表示されてしまう。  
※borderでも同じ  
  
以下で全ての要素を対象とした設定を入れることで、padding等を勘定した％設定となる  
```
*{ box-sizing: border-box; }
```
  
親子関係の要素(divの中にdivとか)でdivをfloatで使用していると、親のheightを固定値にできない  
レスポンシブデザインを設定するとき、画面が狭くなった際の子の並びを二段にした場合  
固定値だと子要素がはみ出す  
  
親要素のheightを設定せず子要素の大きさに任せるとよきにはからってくれると思うが、  
子要素が全てfloatだと、親要素はheightが0になってしまう  
  
なので、float設定をキャンセルする設定のみが入った空の要素が末尾に必要となる  
```
<div class="clear"></div> //空のDivを用意
.clear { clear: left; }   //左に寄せる設定
```

#### 要素の表示、非表示
通常サイズでは文字のリンク、スマホサイズではアイコンに変更したい  
通常サイズではstyleで以下を入れておくと表示されない  
display: none;  
  
表示する  
display: block;  
  
#### DOMについて
document.getElementById  
ブラウザでは標準でjavascriptに対応していて、コンソールから実行して対象の表示した  
HTMLを操作することができる  
  
例えば以下のコンテンツを含むHTMLがある場合
```
<h1 id=fugafuga>ほげふが</h1>

document.getElementById("fugafuga").textContent="ほげふが"
```

id はそのHTML上で一回きりしか使えない  
class はたくさん指定できる  
  
classのが便利そうだが、CSS上での指定が増えて可読性がさがるので、わかりやすくしたい  

### HTMLタグの要素タイプの違いについて
インライン、ブロックレベル要素で二分できる  
  
インライン  
```
<a><span><button><input><img>など  
```
複数のインライン要素は前の要素に横一列に追加される  
横幅が足りなくなった場合、改行されて下段に表示される  
  
ブロックレベル  
```
<h1><h2><p><div><ol><ul><li>  
```
画面横幅全てを使用するので、下方向に追加されていく  
  
ブロックレベルの中にインラインを記載していくことになる  
  
インラインブロック要素  
インライン要素として横に追加されるが、ブロックレベルのように幅や高さを設定することができる  
対象の`<a>`タグに.btnクラスをつけたら、stylesheetで以下の様に指定するとpaddingが効くようになる  
```
.btn {
   padding: 8px 24px;
   display: inline-block;
}
```

### ボックスモデル

全ての要素はボックスという四角で構成されている  
この四角の中のスタイルを指定することができる。ただしブロックレベル要素のみ  
```
margin 　他のブロックレベル要素との間を広げる
border 　ブロック要素の境界そのもの、枠線
padding　ブロックレベル要素と中のインライン要素の間
height 　インライン要素の高さ
width  　インライン要素の横幅

-top -bottom -left -right 後ろにつけると上下左右、特定の指定となる。

border-width: 90px;
border-style: solid;
border-color: red;
border: 2px solid blue; まとめて指定もできる。
```

### white-space

対象ソースの連続する半角スペースや改行の表示方法を操作できる  
```
対象タグ {white-space: 値}
例
p {white-space: pre-wrap;}
```
値について  
nomal(初期値)  
ソース中のホワイトスペースを無視、改行を半角スペースとして表示  
ボックスサイズがある場合、合わせて自動改行する  
  
nowrap  
ソース中のホワイトスペースをそのまま表示、改行もそのまま表示  
ボックスサイズが指定されていても自動改行しない(はみ出すことがある)  
  
pre-wrap  
ソース中のホワイトスペースを無視、ソース中の改行をそのまま表示  
ボックスサイズが指定されている場合は合わせて自動改行する  
  
テキストファイルの読み込み表示などで指定に使用した  