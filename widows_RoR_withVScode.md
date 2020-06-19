# WindowsでRuby

### 履歴
2018/12/13 新規


### ダウンロード
1. rubyのダウンロード  
ここからダウンロードする  
https://rubyinstaller.org/downloads/
2. VScodeもググってダウンロードする。

3. Gitのダウンロード  
https://git-scm.com/download/win  
勝手にパッケージが選択され、ダウンロードされる。


### インストール
1. rubyのインストール  
殆どデフォルト  
終わったらShellが起動するので、1-3まで順番にEnterで実行していく。

2. VScodeのインストール  
デフォルト設定で適当にインストールする。

3. Gitのインストール  
SelectComponentsで、WindowsExplorerIntegrationのチェックを外す。


### セットアップ
Railsのセットアップ  
３つのインストールが終わったら、スタートメニューからgitbashを探して起動する。  
普通にlinuxコマンドが使えるので、  
`ruby -v` , `gem -v` , `which ruby` などで、インストールできてるかチェックし、  
`gem intall rails` でrailsをいれる。

`mkdir /c/rails`  
`cd /c/rails`  
`rails new RailsApp`  
`cd RailsApp`  
`ruby bin/rails server`  

http://localhost:3000  

以上でブラウザでyayできれば問題なし

VScodeのセットアップ  
VScodeを起動して `Ctrl + "+"` で設定が開き、`Ctrl + "@"` でコンソールが起動される。

「ユーザー設定」の部分に追記を行う  
変更前
```
{  
    "workbench.sideBar.location": "left"  
}
```
変更後(一行目に","を忘れずに)  
```html
{
    "workbench.sideBar.location": "left",
    "terminal.integrated.shell.windows": "C:\\Program Files\\Git\\git-bash.exe"
}
```
下窓の「問題」のところでエラーがでなければOK
VScodeを再起動して、Ctrl + @ でターミナルを起動して、右側プルダウンメニューが
PowerShellではなく、bashになっていることを確認する。

カレントディレクトリを維持しているらしく、先のrailsを確認したときの
カレントディレクトリになっているはず。