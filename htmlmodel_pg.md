## ウェブサイト設計のための手法
架空のサイトの構築を例に、企画から構築完了までの流れを追う

### 要件定義
診断〇ーカーのようなページを作りたいとすると、要件は以下のようになる
  
```
１．いいところを診断できる
　　・名前を入力すると診断できる
　　・同じ名前の場合、必ず同じ診断結果になる
　　・診断後に自分の名前の入った診断結果が表示される
２．診断結果をツイートボタンでツイートできる
```
要件定義やサイトの外観(モックアップ)を作成するためのソフトウェアもある
　Adobe XD , Prott など
  
* この段階でプログラムを作らず、外観だけを作成する  
* 文字やフォーム、ボタンの配置などをここで決める
* 手書きでモックアップを作ることもある
  
### 構築(pre)
大体同意が取れたらモックアップをコードで作る  
* buttonとかは動かなくていい
* htmlとcssで要件で話した通りの画面が表示できればよい
  
### プログラム部分の作成
要件通りのプログラムを作る  
作るためにはプログラム的な要素に置き換える必要がある  
```
診断結果のパターンのデータが取得できる
名前を入力すると診断結果が出力される関数
    入力が同じ名前なら、同じ診断結果を出力する処理
    診断結果の文章のうち、名前の部分を入力された名前に置き換える処理
```

診断結果のパターンのデータを用意する  

<details>

<summary>パターンデータ例</summary>

```
const answers = [
    '{userName}のいいところは声です。{userName}の特徴的な声は皆を惹きつけ、心に残ります。',
    '{userName}のいいところはまなざしです。{userName}に見つめられた人は、気になって仕方がないでしょう。',
    '{userName}のいいところは情熱です。{userName}の情熱に周りの人は感化されます。',
    '{userName}のいいところは厳しさです。{userName}の厳しさがものごとをいつも成功に導きます。',
    '{userName}のいいところは知識です。博識な{userName}を多くの人が頼りにしています。',
    '{userName}のいいところはユニークさです。{userName}だけのその特徴が皆を楽しくさせます。',
    '{userName}のいいところは用心深さです。{userName}の洞察に、多くの人が助けられます。',
    '{userName}のいいところは見た目です。内側から溢れ出る{userName}の良さに皆が気を惹かれます。',
    '{userName}のいいところは決断力です。{userName}がする決断にいつも助けられる人がいます。',
    '{userName}のいいところは思いやりです。{userName}に気をかけてもらった多くの人が感謝しています。',
    '{userName}のいいところは感受性です。{userName}が感じたことに皆が共感し、わかりあうことができます。',
    '{userName}のいいところは節度です。強引すぎない{userName}の考えに皆が感謝しています。',
    '{userName}のいいところは好奇心です。新しいことに向かっていく{userName}の心構えが多くの人に魅力的に映ります。',
    '{userName}のいいところは気配りです。{userName}の配慮が多くの人を救っています。',
    '{userName}のいいところはその全てです。ありのままの{userName}自身がいいところなのです。',
    '{userName}のいいところは自制心です。やばいと思ったときにしっかりと衝動を抑えられる{userName}が皆から評価されています',
];
```
  
</details>

#### ロジックの用意
任意の名前と用意した結果をどう紐づけるか  
名前を文字コードに変換して合算し文字数で割った余り値をanswer配列のindexとして使う  
  
#### コーディング

```
function assessment(userName) {
    // userNameを一文字ずつ数値化して、合計値を求める
    let sumOfCharCode = 0;
    for (let i = 0; i < userName.length; i++) {
        sumOfCharCode = sumOfCharCode + userName.charCodeAt(i);
    }
    // 文字数で割って余りを出す
    const index = sumOfCharCode % answers.length;
    let result = answers[index];
    result = result.replace(/\{userName\}/g, userName);
    return result;
}
```

#### テストと手法

シンプルにconsole.logに値を出力する
```
console.log(assessment('スマホ太郎'));
console.log(assessment('デスマ二郎'));
console.log(assessment('スマホ太郎'));
```
chromeのコンソールに表示されればよい  
  
親切なテスト
```
console.assert(
    assessment('スマホ太郎') === 'スマホ太郎のいいところは決断力です。スマホ太郎がする決断にいつも助けられる人がいます。',
    '診断結果の文言の特定の部分を名前に置き換える処理が正しくありません'
);
```
assertに想定する正解の出力と偽の時のコンソールメッセージを書いておける  

#### 結果をHTML側で使う

html側で表示用の領域を用意する
```
<body>
    <h1>あなたのいいところは？</h1>
    <p>診断したい名前を入れてください</p>
    <input type="text" id="user-name" size="40" maxlength="20">
    <button id="assessment">診断する</button>
    <div id="result-area"></div>
    <div id="tweet-area"></div>
    <script src="shindan.js"></script>
</body>
```
html側で用意した領域にjsの関数出力を表示する
```
// htmlへの接続
const userNameInput = document.getElementById('user-name');
const assessmentButton = document.getElementById('assessment');
const resultDivided = document.getElementById('result-area');
const tweetDivided = document.getElementById('tweet-area');

/**
 * 指定した要素の子供を全て削除する
 * @param {HTMLElement} element HTMLの要素
 */
function removeAllChildren(element) {
    while (element.firstChild) {
        element.removeChild(element.firstChild);
    }
}

//assessmentButton.onclick = function() {
assessmentButton.onclick = () => {
    const userName = userNameInput.value;
    if (userName.length === 0) {
        return;
    }
    //console.log(userName);
    // 診断結果表示エリアの作成
    removeAllChildren(resultDivided);
    const header = document.createElement('h3');
    header.innerText = '診断結果';
    resultDivided.appendChild(header);

    const paragraph = document.createElement('p');
    const result = assessment(userName);
    paragraph.innerText = result;
    resultDivided.appendChild(paragraph);
};
```


