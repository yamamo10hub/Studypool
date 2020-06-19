### 事前にやったこと
windowsでRubyonRailsとgitとVScodeをセットアップした


### railsファイル解説

アプリ直下のファイル確認
* .gitignore  
 gitでアップロードしないファイルたち。
 railsのコンポーネントやパッケージまで上げる必要はないので、それらが書かれるはず
* config.ru  
 その場で実行するときに必要な設定ファイル。

* gemfile  
gemのパッケージリスト

* rakefile

* readme.md  
 gitのトップにあるアレ

* appディレクトリの中身概要
	* asset  
css, javascript など
	* channels  
websocketなどのセッションファイルなど
	* controllers  
MVCのコントローラーファイル
	* helpers  
ヘルパーファイルを置く。繰り返し部分のhtmlファイル
	* jobs  
Activejobというバックグラウンドでなにかできるプログラムで使う
	* mailers  
Activemailというメール送信系のプログラムでつかう。
	* models  
MVCのモデルファイル。データベース
	* views  
MVCのビューファイル。メインのerbファイルを置く

railsの仕組みの中にいる間は、cssとjavascriptは書き方が異なる。  
coffescript =~ javascript  
scss =~ css

### controllerについて

rails generate controller helo

app/config/route.rb
```
#書き方で挙動が違う
# progate
get 'helo/index'
get 'helo/index' => 'helo#index'
# 超入門(ruby2.3)
get 'helo/index'
get 'helo', to: 'helo#index' 
```

ruby2.4の環境では、progateはindexが表示できず、超入門ではどちらでもアクセスできた。  
バージョンに依る挙動の違いだろうか。

```
class HeloController < ApplicationController
    def index
        msg = '
        <html><body>
        <h1>samplepage<h1>
        <p>testpage is works...</p>
        </body></html>'
        render html: msg.html_safe
	#render plain: 'hogehoge'
    end
end
```

* render plainでプレーンテキストが出力できる。
* render html でHTMLが出力できる。
* html_safe クラスで<>等が文字化けせず表示される。(つけないと文字化けする。)

### クエリストリングスとしてのparams

<details>
<summary>ソースを確認</summary>

```
routes.rb
Rails.application.routes.draw do
  get 'helo/index'
  get 'helo', to: 'helo#index' 
end

helo_controller.rb
class HeloController < ApplicationController
    def index
        if params['msg'] != nil then
            msg = 'Hello,' + params['msg'] + '!'
        else
            msg = 'samplepage is works' 
        end
        html = ' <html><body> <h1>Samplepage</h1> <p>' + msg + '</p> </body></html>'
        render html: html.html_safe
    end
end
```

</details>

http://localhost:3000/helo?msg=ukiyama  
paramsで待ち受けておくと、railsはURLのクエリを受け取ることができる。

railsのHTML描画の基本セット  

* controller/helo.controller.rb
* views/helo/(dir)
* helo_controller_test.rb
* helo.helper.rb
* helo.coffee
* helo.scss

### コントローラーで作った変数をerbに埋め込む

helo_controller.rb  
```
class HeloController < ApplicationController
    def index
        @title = "Viewサンプル"
        @msg = "コントローラーに定義した値です。"
    end
end
```
views/helo/index.html.erb
```
<h1><%= @title %></h1>
<p><%= @msg %></p>
```

</details>

### redirectを試す

controllerで`redirect_to`を記載すると別のURLへ移動できる。

<details>
<summary>ソースを確認</summary>

helo_controller.rb  
```
class HeloController < ApplicationController
    def index
        if params['msg'] != nil then
            @title = params['msg']
        else
            @title = 'index'
        end
        @msg = 'this is redirect sample...'
    end

    def other
        redirect_to action: :index, params: {'msg': 'from other page'}
    end
end
```
routes.rb
```
Rails.application.routes.draw do
  get 'helo/index'
  get 'helo', to: 'helo#index' 
  get 'helo/other'
end
```

</details>

### フォームを使う

フォームに入力した内容をページのテキストに反映させる  
railsでHTML5標準のformをそのまま記載するとCSRFというなりすましクエリ対策
でブロックされる。  
formを安全に使用するためにrailsの用意した使い方でformを作る。  

<details>
<summary>ソースを確認</summary>

index.html.erb
```
<h1><%= @title %></h1>
<p><%= @msg %></p>
# htmlのformタグ生成
<%= form_tag(controller: "helo", action: "index" ) do %>
# 入力フィールドinput1を作る
    <%= text_field_tag("input1") %>
# 送信ボタンClickを作る
    <%= submit_tag("Click") %>
# form作成用ループ分の終了end
<% end %>
```

helo_controller.rb
```
class HeloController < ApplicationController
    def index
        if request.post? then
        # form入力があった時の表示内容
            @title = "Result"
            @msg = 'you typed: ' + params['input1'] + '!'
            @value = params['input1']
        else
        # form入力がない時の表示内容
            @title = 'Index'
            @msg = "type text is none."
            @value = ''
        end
    end
end
```

routes.rb
```
Rails.application.routes.draw do
  get 'helo/index'
  get 'helo', to: 'helo#index' 
  post 'helo', to: 'helo#index'
  post 'helo/index'
end
```

</details>

### チェックボックスの利用

<details>
<summary>ソースを確認</summary>

index.html.erb
```
<h1><%= @title %></h1>
<p><%= @msg %></p>
<%= form_tag(controller: "helo", action: "index" ) do %>
    <%= check_box_tag("check1") %>
    <%= label_tag("check1", "check box") %>
    <%= submit_tag("Click") %>
<% end %>
```
helo_controller.rb
```
class HeloController < ApplicationController
    def index
        if request.post? then
            @title = "Result"
            if params['check1'] then
                @msg = 'you checked !'
            else
                @msg = 'not checked ...'
            end
        else
            @title = 'Index'
            @msg = "check It"
        end
    end
end
```
</details>

### ラジオボタンを作る
どれか一つを選択した状態と、何も選択していない状態が存在するので、`if request.post? then`を使用し、  
何も選択されていない場合を検知できるようにする必要がある。

<details>
<summary>ソースを確認</summary>

index.html.erb
```
<h1><%= @title %></h1>
<p><%= @msg %></p>
<%= form_tag(controller: "helo", action: "index" ) do %>
    <%= radio_button_tag("r1", "radio1") %>
    <%= label_tag("r1_radio_1", "radio button 1") %>
    <%= radio_button_tag("r1", "radio2") %>
    <%= label_tag("r1_radio_2", "radio button 2") %>
    <%= submit_tag("Click") %>
<% end %>
```

helo_controller.rb
```
class HeloController < ApplicationController
    def index
        if request.post? then
            @title = "Result"
            if params['r1'] then
                @msg = 'you selected: ' +  params['r1']
            else
                @msg = 'not selected...'
            end
        else
            @title = 'Index'
            @msg = "select radio button..."
        end
    end
```
</details>

### セレクトボックスを使う

<details>
<summary>ソースを確認</summary>

index.html.erb  
```
<h1><%= @title %></h1>
<p><%= @msg %></p>
<%= form_tag(controller: "helo", action: "index" ) do %>
    <%= select_tag('s1', options_for_select(["Windows", "macOS", "Linux"])) %>
    <%= submit_tag("Click") %>
<% end %>
```

helo_controller.rb
```
class HeloController < ApplicationController
    def index
        if request.post? then
            @title = "Result"
            if params['s1'] then
                @msg = 'you selected: ' +  params['s1']
            else
                @msg = 'not selected...'
            end
        else
            @title = 'Index'
            @msg = "select List..."
        end
    end
end
```

</details>

### 複数選択可能なセレクトボックス

<details>
<summary>ソースを確認</summary>

index.html.erb  
```
<h1><%= @title %></h1>
<p><%= @msg %></p>
<%= form_tag(controller: "helo", action: "index" ) do %>
    <%= select_tag('s1', options_for_select(["Windows", "macOS", "Linux"]), {size:5, multiple:true}) %>
    <%= submit_tag("Click") %>
<% end %>
```

helo_controller.rb
```
class HeloController < ApplicationController
    def index
        if request.post? then
            @title = "Result"
            if params['s1'] then
                @msg = 'you selected: '
                for val in params['s1']
                    @msg += val + ' '
                end
            else
                @msg = 'not selected...'
            end
        else
            @title = 'Index'
            @msg = "select List..."
        end
    end
end
```

</details>

### レイアウトについて

ページ内のヘッダー、フッター、メニューなど、ページ遷移で変化しないウェブ表示部分について  
railsのレイアウト機能としてまとめることで、今までのように毎回ヘッダやフッタを書かなくてもよい機能です  
  
/app/views/layouts/ にファイルが生成されている  
* application.html.erb  
デフォルトのレイアウトを記載するファイル
* mailer.html.erb  
* mailer.text.erb  
上記はrailsのメール機能で使用される  

##### レイアウトファイルの記載方法

デフォルトのファイルを使用して意味を確認する  
/app/views/layouts/application.html.erb
```
<!DOCTYPE html>
<html>
  <head>
    <title>RailsApp</title>
    <%= csrf_meta_tags %>
    # 外部からのパラメータ直打ちを防止するCSRF機能でHTML出力する
    # <meta name="csrf-param" ...>
    # <meta name="csrf-token" ...>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    # railsでスタイルシートが二種類あり、全体とそのページ(View)単体両方のCSSを適用する。
    # HTML化した時そのスタイルシートへのリンクを生成するための記述
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
    # railsが画面表示するために必要な予めくみこまれているjavascriptを読み込ませるための
    # HTML用リンクを生成するための記述
  </head>

  <body>
    <%= yield %>
    # yieldという関数が起動する。コントローラー上でのアクションの内容をすべて出力する
  </body>
</html>
```

### オリジナルのレイアウトを作成する

helo.html.erb
```
<!DOCTYPE html>
<html>
<head>
    <title><%= @title %></title>
    # titleを@titleの変数
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
</head>
<body>
    <%= render template:'layouts/helo_header' %>
    <%= yield %>
    <div> </div>
    <hr size='1' color='#aaa'>
    <%= render template:'layouts/helo_footer' %>
</body>
</html>
```

以下でヘッダー、フッターを描画(render)させる  
```
<%= render template:'layouts/helo_header' %>
<%= render template:'layouts/helo_footer' %>
```

app/views/layouts/helo.html.erb
```
<!DOCTYPE html>
<html>
<head>
    <title><%= @title %></title>
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
</head>
<body>
    <%= render template:'layouts/helo_header' %>
    <%= yield %>
    <div> </div>
    <hr size='1' color='#aaa'>
    <%= render template:'layouts/helofooter' %>
</body>
</html>
```

app/views/layouts/helo_header.html.erb
```
<h1 style="font-size:22pt;color:#ffaaaa;">
    <%= @header %>
<h1>
```
app/views/layouts/helo_header.html.erb
```
<div style="text-align:right; font-size:9pt;">
    <%= @footer %>
</div>
```
app/views/helo/index.html.erb
```
<p><%= @msg %></p>
```
controllers/helo_controller.rb
```
class HeloController < ApplicationController
    layout 'helo'
    def index
        @header = 'layout sample'
        @footer = 'its copylight hogehoge 2018'
        @title = 'new layout'
        @msg = 'this is sample page'
    end
end
```

＠に各値を入れることで、その中身を各コンテンツに割り当てることができる。

### 新しいURLで表示できるページを追加する。

ruby bin/rails generate controller dengonban index

