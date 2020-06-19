flaskの公式チュートリアル　その１
  
[[_TOC_]]
  
## アプリを作りながら仕組みを理解する  
ブログを作ってみる  
  
#### workspaceの準備
ディレクトリ
```
mkdir sample01
mkdir sample01/flaskr
mkdir sample01/flaskr/static
mkdir sample01/flaskr/template
```
ファイル
```
touch sample01/flaskr/schema.sql
touch sample01/flaskr/flaskr.py
```
  
これらは決められた配置ではないが、特に逆らう理由もない定石のようだ  
  
#### sqliteを使う

schema.sql
```
drop table if exists entries;
create table entries (
  id integer primary key autoincrement,
  title string not null,
  text string not null
);
```

この内容を元にDBが作られる  
  
#### アプリケーション部分

flaskr.py
```
# import module
import sqlite3
from flask import Flask, request, session, g, redirect, url_for, \
    abort, render_template, flash

# config param
DATABASE = '/tmp/flaskr.db'
DEBUG = True
SECRET_KEY = 'development key'
USERNAME = 'admin'
PASSWORD = 'default'

# create application
app = Flask(__name__)
app.config.from_object(__name__)
# 同ファイル内の大文字の変数をappに読み込むfrom_object

# 環境変数からconfigparamをappに読み込む場合はfrom_envvar
# app.config.from_envvar('FLASKR_SETTINGS', silent=True)

def connect_db():
    return sqlite3.connect(app.config['DATABASE'])

if __name__ == '__main__':
    app.run
```
この時点ではconfigparamの変数たちと、connect_dbの関数を持っているだけのアプリとなる  
  
#### dbの追加
schema.sql
```
sqlite3 /tmp/flaskr.db < schema.sql
```


flaskr.py
```
from __future__ import with_statement
from contextlib import closing

def init_db():
    with closing(connect_db()) as db:
        with app.open_resource('schema.sql') as f:
            db.cursor().executescript(f.read())
        db.commit()

# DBを利用するときの関数
def before_request():
    g.db = connect_db()

def after_request(response):
    g.db.close()
    return response
```

#### DBに書き込むためのページ

flaskr.py
```

```