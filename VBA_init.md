## エクセルVBA
エクセルに紐付いたスクリプト

### 準備
1. 新規のエクセルファイルを作る  
2. 窓ボタンから「Excelのオプション」を選択  
3. 「[開発]タブをリボンに表示する」にチェック  
4. 表示された[開発]タブを選択、VisualBasicを選択  
　→ 窓が開く(Alt + F11でもよい)  
5. 開いた窓で[ツール][オプション]「変数の宣言を強制する」にチェック  
6. [挿入][標準モジュール]でコードを書く窓が開く  
7. CTRL + G でイミディエイトウィンドウを開く(デバッグ窓)  
8. もとのエクセル窓側で保存を行う  
　→ ファイルの拡張子を .xlsm に変えろといってくるのでそうする  

### サンプルをつくる

```
Option Explicit '自動で書かれる
'コメントはシングルクォートらしい

Sub test() 'これをプロシージャというらし

Dim numTest As Long   '変数は宣言が必要
Dim strTest As String
Dim dateTest As Date

numTest = 505
strTest = "0505" '文字列と日付の代入はダブルクォートが必要
dateTest = "5/5"

Debug.Print numTest '窓下のイミディエイトに表示されるデバッグ出力
Debug.Print strTest
Debug.Print dateTest

End Sub
```

