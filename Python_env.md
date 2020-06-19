Pythonの環境構築とflaskのチュートリアル
  
[[_TOC_]]
  
### VScode でpythonを書く時にしたこと<hr>  

* windows用pythonを検索し、インストールする
* VScodeの拡張でmicrosoft謹製のpluginがあるのでインストールする  
* コマンドラインで pip install flask
* コマンドラインで pip install pylint
  
lintはインデント等を整形してくれるモジュールで、多言語でも存在する  
pythonの場合は実行時にインデントがおかしいとエラーになるので、lintを必ず使った方がいい  
(人の手でインデントを確認する機会が減るので)  
  
### 実行ディレクトリの決定<hr>
作成するプログラムを置くディレクトリを決めたほうがよい
```
ex. C:python-pg/ として作成し、プログラムごとにサブDirを作成し、その中で使う
```
pythonはわからないが、フレームワークなどはディレクトリ単位で構成ファイルが展開される

### サンプル実行<hr>
最小のコードがあったので、それを書いてみる  
```  
mkdir -p D:py/sample01
cd D:py/sample01
touch app.py
```
app.py
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello_world():
    return "Hello, World"
```
  
書けたらshellを起動させる
```
cd D:py/sample01
set FLASK_APP=app.py
flask run
```
ブラウザで 127.0.0.1:5000 でアクセスすると、アプリが実行される
また、コンソールにアクセスログが残る

  
**自動再起動**  
flaskはコード書き換え後にflask runを手動でしなければならないが、  
以下を設定することで自動になる。(デバッグモードで動作するので、本番では使わない)  
```
if __name__ == "__main__":
    app.run(debug=True)
```
flask run --debugger --reload

  
### routingについて<hr>
サンプルにrouteを足して、URLのしくみを学ぶ
```
@app.route("/about")
def about():
    return "<h1>About</h1>"

@app.route("/hello/<whom>")
def hello(whom):
    return f"<h1>Hello {whom}</h1>"
```
アクセスさせたいURIを / 以降につけることで、コンテンツの出し分けができる  
route指定内で <> を使用して囲むと変数として値を受け付けることができる  
変数を扱う場合は return f を使う必要がある  

また、末尾に/ が付いている場合、トップに移動と解釈する。  
/ がついていない場合、404を出す  
  
### コンバータ<hr>
プログラムに渡すパラメータの型を変えたい
```
@app.route("/100_plus/<int:n>")
def adder(n):
    return f"100+{n}={100+n}"
```
URIに値を含む場合文字列になってしまうが、コンバータをつけることで型を変えることができる  
routeで <型:n> のように書く。受け取る関数側では特に変わらない  
他にも string／int／float／path／uuid など指定可能な型がある
  
また、このroutingに対してintで解釈できないURLを入力されることはあり得るが、  
その場合、コンバータの実装によって404を返す。(有効な数字のみを含むURIの場合のみ正しく動作する)  
  
### methodの制限

routeをGETとPOSTだけに制限する
```
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```
例えばDELETEを受けたい場合、routeをもう一つ同じURLを指定して作成して受け取ることもできる。  

  
### テンプレートの使用<hr>
いちいち return に表示為のHTMLを書くととても見ずらい  
templateというHTMLのひな形を作って設置し、変数の部分だけプログラムが入力することで、  
繰り返しやコードの難読化を防げる  
```
mkdir D:py/templates
touch D:py/templates/hello.html
```
  
hello.html
```
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Hello</title>
</head>
<body>
  <h1>Using Jinja2 Template engine</h1>
  <h2>Hello {{whom}}</h2>
</body>
</html>
```
Jinja2というテンプレートエンジンらしい。  
サンプルを貼っておく  
  
app.py
```
from flask import Flask, render_template
app = Flask(__name__)
@app.route("/hello/<whom>")
def hello(whom):
    return f"<h1>Hello {whom}</h1>"
```
  
公式の例  
hello.html
```
<!doctype html>
<title>Hello from Flask</title>
{% if name %}
  <h1>Hello {{ name }}!</h1>
{% else %}
  <h1>Hello, World!</h1>
{% endif %}
```
  
app.py
```
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```


### CSSの使用
静的ファイルであることをコード中で示す必要がある  
また、設置さきは ./static/ とする必要がある  

```
url_for('static', filename='style.css')
```
  
### 埋め込みで特定の関数を動かす

url_forの使い方として関数を埋めることができる  
printの内容表示やリンク先として使用する等
```
 from flask import Flask, url_for
 app = Flask(__name__)
 @app.route('/')
   def index(): pass
 @app.route('/login')
   def login(): pass

>>> @app.route('/user/<username>')
 def profile(username): pass

>>> with app.test_request_context():
  print url_for('index')
  print url_for('login')
  print url_for('login', next='/')
  print url_for('profile', username='John Doe')
/
/login
/login?next=/
/user/John%20Doe
```

### セッション

### メッセージフラッシング
### ログ取り





