## golang の基本
文法、記載内容、実行方法、ルールなど  
参照先：http://cuto.unirita.co.jp/gostudy/  
  
## 実行
サンプルプログラム
```
package main

import "fmt"

func func01() {
   var num int
   num = 3

   fmt.Printf("hoge is %d\n", num)
}
```
  
#### package main  
golangはpackageという単位で名前空間を分ける仕組みを持つ  
* コードはいずれかの名前空間に属する。  
* つまりコードに１つは名前空間が必須ということ。
    
他の名前空間の変数を使う場合はこうなる  
```
package sub01

import "main"

func sub01() {
   main.func01()
}
```
必ず使用するpackageを可読性のために分けている場合、いちいちpackage名を付けるのが面倒な場合もある  
  
その場合、packageへの別名付与を利用して以下のようにすると省略が許される  
※あまり推奨されないらしい。(テストの為に本番のpackageを読み込む時等は有用とのこと)  
```
package sub01

import . "main"

func sub01() {
   func01()
}
```
package内に記載される変数及び関数をmemberと呼ぶ  
memberには種類がある（public , static）  
* publicmember  
名前の頭文字を英字大文字にした場合はimport文を使用して他パッケージからの参照が可能  
* staticmember  
名前の頭文字を小文字にした場合はimport文を使用しても他パッケージから参照できない  
  
#### importの動作について  
他所のパッケージを使う時importで名前を指定しているが、本来のimportの挙動を  
把握しておく必要がある。  
```
gopathというv1.08以降自動で設定されるOS環境変数を起点とした相対pathで指定された
ファイルやモジュール？を読み込む機能
```  
package名にmainを使った場合、そのファイルの位置がエントリポイントとなり、  
gopath以下に置かなくても読み込みが可能となる    
  
#### fmt
フォーマットに関するパッケージ  
http://golang.jp/pkg/fmt  
今回はprintするために必要なのでimportしている  
表示物の型によって指定するメタ文字が変わるので、覚えたほうがよさそう  
  
## 記法について
サンプル
```
package main

import "fmt"

func main() {
    var num int
    var pow int
    var result = 1

    num = 2
    pow = 4
    for i := 0; i < pow; i++ {
       result *= num
    }

    fmt.Printf("%dの%d乗は%dです。\n", num, pow, result)
}
```

* 変数宣言に型の指定が必要  
* 同行に２つ文を書かない限り、末尾のセミコロンは省略可能らしい(for文のところ)  
* 変数宣言と同時に初期化する場合は、型名を省略できる(初期化内容で型推論してくれる)  
* 関数内に限ってvarの接頭辞も省略できる（scopeが関数内だからかな）  
` i := 0; は var i = 0; に等しい`  
* 括弧()が不要  
```
if a < 5 {}   // OK
if (a < 5) {} // OKだが無駄
```

#### 定数がある
後から変更できない。関数内に宣言した場合、scopeは変数と同じで関数内だけ  
```
const title = "Golang基礎"
fmt.Println(title);
```
  
## 制御文
#### 条件分岐
if文
```
import (
    "fmt"
    "time"
)
func main() {
    hour := time.Now().Hour()
    if hour >= 6 && hour < 12 {
        fmt.Println("朝です。")
    } else if hour < 19 {
        fmt.Println("昼です。")
    } else {
        fmt.Println("夜です。")
    }
}
```
switch文
```
dayOfWeek := "月"
switch dayOfWeek {
case "土":
    fmt.Println("大概は休みです。");
case "日":
    fmt.Println("ほぼ間違いなく休みです。")
default:
    fmt.Println("仕事です・・・。")
}
```
goではcaseを文字列判別できる。また、breakは記載しない場合暗黙で付与される  
breakしたくない場合はfalsethroughと明記する  
```
weekday := "月"
switch weekday {
case "土":
    fallthrough
case "日":
    fmt.Println("土日は休み")
default:
    fmt.Println("土日も仕事")
}
```
switch条件は,で複数指定できる
```
weekday := "月"
switch weekday {
case "土", "日":
    fmt.Println("土日は休み")
default:
    fmt.Println("土日も仕事")
}
```
分岐後の処理内容が簡潔であれば、if文より見栄え良く書ける
```
hour := time.Now().Hour()
switch {
case hour >= 6 && hour < 12:
    fmt.Println("朝です。")
case  hour < 19:
    fmt.Println("昼です。")
default:
    fmt.Println("夜です。")
}
```
ifはどうしても{}を使うので、数や位置を気にしてしまう  
  
## 繰り返し
#### for 
```
for i := 1; i < 100; i++ {
    if i / 2 != 0 {
        fmt.Println(i)
    }
}
```
forの後ろに何も書かないと無限ループになるので注意  
変数の内容を使ったfor文  
```
weekdays := [...]string{"月", "火", "水", "木", "金", "土", "日"}
for arrayIndex, weekdays := range weekdays {
    fmt.Printf("%d番目の曜日は%s曜日です。\n", arrayIndex + 1, weekdays)
}
```
  
## 構造体
クラスらしきものが無いので、オブジェクト作る場合これでやるのかな  
C言語触った時にみた。ポインタもあるしCに似ている？  
  
#### 作り方
```
package main
import "fmt"

func main() {
    var vector struct {
        X int
        Y int
    }
    vector.X = 2
    vector.Y = 5
    fmt.Println(vector) // {2 5}
}
```
定義の可読性を上げつつ省略もできるようにする
```
package main
import "fmt"

type Vector struct {
    X int
    Y int
}

func main() {
    var v Vector
    v.X = 2
    v.Y = 5
    fmt.Println(v)
    //上記と同位
    //v := Vector{X: 2, Y: 5}
    //fmt.Println(v)
}
```
`type Vector` で宣言しているが、vectorにした場合、異なるパッケージに呼び出すことができない  
  
## 関数・メソッド
#### 一般的サンプル
```
package main
import "fmt"

func main() {
    DisplayHello()
}
func DisplayHello() {
    fmt.Println("Hello!")
}
```
main関数外に定義し、main関数内で呼び出す。これもC っぽい  
  
#### 可変長の引数を使う  
引数がいくつになるかわからん場合、明示的な記載ルールがある
```
package main
import "fmt"

func main() {
    DisplaySumAll(2, 5, 8, 11) // 26が表示される
}
func DisplaySumAll(values ...int) {
    sum := 0
    for _, value := range values {   // この '_' は？ ループ用変数無い場合の省略法？
        sum += value
    }
    fmt.Println(sum)
}
```
  
#### 戻り値

戻り値を返したい場合、返す型を引数の後ろにつけなければならない
```
package main
import "fmt"

func main() {
    fmt.Println(Sum(2, 5)) // 7が表示される
}
// {}の前のintが戻り値の型で、必要な記載
func Sum(left int, right int) int {
    return left + right
}
```
複数の戻り値を指定する場合、型の指定を()で囲む  
```
package main

import "fmt"

func main() {
    result, remainder := Div(19, 4)
    fmt.Printf("19を4で割ると%dあまり%dです。\n", result, remainder)
}

func Div(left int, right int) (int, int) { // ここ
    return left / right, left % right
}
```
  
#### レシーバ変数
この辺からつらくなってきた
メソッドを使ってオブジェクト指向のクラスのようにふるまわせるための準備のようななにか  
  
一旦中断して書いてみることにする  
