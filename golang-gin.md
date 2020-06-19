## 何ぞよ
golangで作られたウェブフレームワーク  
日本語訳があるので、他のフレームワークより良さそうに見える  
  
## 使い方

#### golangをインストールする  
今回はwin10でVScodeをつかう  
```
yamamoto-desk@yamamo-desk-PC MINGW64 /d
$ go version
go version go1.13.8 windows/amd64
```
  
環境変数の設定  
```
GOROOT    C:\Go\
golangをinstallした場所かな

GOPATH    D:\go\
ワーキングディレクトリの設定らしい。
ginだとプロジェクト毎に変わらないすか？

PATH    ここはmsiインストーラーが設定してくれる
```
無くても動いてしまったが、設定した方がいいだろう
  
ワーキングディレクトリを作る
```
mkdir -p /d/go/exam-gin
mkdir /d/go/exam-gin/templates
cd /d/go/exam-gin
```
exam-ginの部分をプロジェクト毎にdirectory作る感じで使おうと思う  
templates以下はstaticなファイルを置く感じ
  
#### フレームワークの読み込み

1. ダウンロードして使う
```
プロジェクトdirectory以下にginのコンポーネントを置く
go get -u github.com/gin-gonic/gin
どうにもwin10だとカレントdirではなくhomedirにダウンロードしてくる気がする
homedirに来たら、mvでプロジェクトのdir下に置く？
これでインストールできてる説がなんかある
/c/go/bin/ 以下に似たディレクトリがあるので、これとマージする感じか
```
2. コードに埋める
```
import "github.com/gin-gonic/gin"
12は同じかもしれない（どちらも必須的な）
```
  
#### ハローなコード
最小基本書式で動かしてみる  

/d/go/exam-gin/exam.go
```
package main

import "github.com/gin-gonic/gin"

func main() {
	//r = router
	r := gin.Default()

	r.LoadHTMLGlob("templates/*.html")
	r.GET("/", func(c *gin.Context) {
		c.HTML(200, "index.html", gin.H{})
	})

	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{"message": "pong",})
	})

	data := "variable data"
	r.GET("/variable", func(ctx *gin.Context){
		ctx.HTML(200, "variable.html", gin.H{"data": data})
	})
	//渡すパラメーターをgin.H に書くってことは.H はハンドラのことかな

	r.Run()
}
```
goのプログラム書き始めるとVScodeではプラグインのinstallを薦められるので入れておく  
  
/d/go/exam-gin/templates/index.html
```
<!DOCTYPE html><html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>text index page</title>
    </head>
    <body>
        <h1>gin world</h1>
    </body>
</html>
```
/d/go/exam-gin/template/variable.html
```
<!DOCTYPE html><html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>v page</title>
</head>
<body>
    <h1>{{.data}}</h1>
</body></html>
```
  
起動する
```
go run exam.go
runが通ると、windowsがginアプリの起動許可を表示するので、許可する
VScodeのplugin入れておくと、起動後にターミナルがgoになる
```
ブラウザで以下のURLにアクセスすると返事があるはず  
```
localhost:8080/
localhost:8080/ping
localhost:8080/variable
```

#### DBを使いたい
CRUDなアプリケーションを作るなら必要になる  
golangではモジュールを追加して使えるようにする  
  
```
go get github.com/jinzhu/gorm
go get github.com/mattn/go-sqlite3
```
  
gccでpathが無い的なエラーでこけた場合、環境変数を確認する
```
go env
以下が1だったら0に変える必要がある
CGO_ENABLED="1"
変え方は以下
export CGO_ENABLED=0
```
  


