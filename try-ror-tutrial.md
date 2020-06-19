## Rails Tutrialの実践

### 実践概要
今までの実績 
独学でスクリプト作成 -> RailsTutrial -> progate -> RoR超入門 -> progate -> RailsTutrial(Now)  
どうにかフレームワーク１つ使えるようになりたいところ  

### 事前知識
RoR超入門のころRailsTutrialの復習を検討していたが、AWScloud9を利用した開発環境に内容が更新されていたので  
AWS入門も同時に進めていた。  

### 開発環境の準備
別項のAWSの利用を参照し、EC2で作成したインスタンスが外部にアクセスできるVPCの作成まで行う。  
確認に使用したインスタンスは削除してよい
- [アカウントを作る](AWS_mkid)
- [インスタンス(仮想マシン)を作る](AWS_ec2)
- [VPCでネットワークを作る](AWS_VPC)

#### Cloud9のセットアップ  
1. コンソールトップからcloud9で検索
2. Create environment
3. 名前とDescriptionを決める
4. cloud9用のインスタンスのスペックとネットワークの選択
5. 内容を確認して作成
6. 完了するとIDEぽい画面が表示された
7. 画面内右ペインの右上の歯車で設定を開き、SoftTabsを 4 -> 2 に変更  
   pythonなら4のままかな？
8. Welcome タブに戻り、下ペインからコマンドが打てるので以下を入力する
```
printf "install: --no-rdoc --no-ri\nupdate:  --no-rdoc --no-ri\n" >> ~/.gemrc
```
gemのインストール時にdocumentをダウンロードしない設定
9. 8と同じく、コマンドでrailsのinstall
```
gem install rails -v 5.1.6
```
9. test時に多重ターミナル使用によるエラー出力を消す
```
sudo yum install -y tmux
```
ターミナルをwebIDEで複数動かしているとエラーがでるので、ターミナル仮想化パッケージをいれると解決するらしい

#### ハロワを作りながら残りの環境のセットアップ
移行コマンドはウェブIDEの下ペインに入力するものとする
1. `cd environment`
2. `rails _5.1.6_ new hello_app`
3. gemfileを書き換え、使用gemのversionを固定する
```

source 'https://rubygems.org'

gem 'rails',        '5.1.6'
gem 'puma',         '3.9.1'
gem 'sass-rails',   '5.0.6'
gem 'uglifier',     '3.2.0'
gem 'coffee-rails', '4.2.2'
gem 'jquery-rails', '4.3.1'
gem 'turbolinks',   '5.0.1'
gem 'jbuilder',     '2.6.4'

group :development, :test do
  gem 'sqlite3',      '1.3.13'
  gem 'byebug', '9.0.6', platform: :mri
end

group :development do
  gem 'web-console',           '3.5.1'
  gem 'listen',                '3.1.5'
  gem 'spring',                '2.0.2'
  gem 'spring-watcher-listen', '2.0.1'
end

# Windows環境ではtzinfo-dataというgemを含める必要があります
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]

```
4. bundle install でgemfileの内容通りにgemの更新を行う
5. 下ペインの緑の+からNewterminalすることで、新しい窓でコマンドが入力できることを確認する。
6. 新しい窓でrails serverし、ブラウザを起動してみる
7. 上部メニューでPreview -> PreviewRunningApplication を選択するとペインの右部に新しいペインが開く
8. 新しいペイン内のアドレスバーの右にある矢印のアイコンを押すことで、新しいタブでブラウザに結果が表示される
9. app/controllers/application_controller.rb を編集する
```
  def hello
    render html: "hello, world"
  end
```
10. config/routes.rb を編集する
```
  root 'application#hello'
```
11. 8の通りに再度ブラウザで表示を確認する

### RoRでのgit
rails new で作成されたDirでGitを使う  
アプリは時々使用されるたびにgemのversionが変わったりなどが起こるので、  
railsはbundleで生成される部分のファイルは含まないように自動でgitignoreに登録してくれる  

git管理下への登録
```
git init
git add -A
git status
git commit -m "init repo"
git log
```

一度commitしてしまえば、外部へ登録しなくても、commit時の状態に戻せるので便利
```
git checkout -f
```

#### 外部にプッシュする
- gitサービスにログインする（新規作成してもよい）
クラウドIDEなので、HTTPSでpushするつもり
- そのgitサービスで新規プロジェクトを作る(プロジェクト名もここで決まる)
- git remote add origin https://github.com/ユーザ名/プロジェクト名.git (ex.)
https://github.com/yamamo10/ror_hello.git(例)
- git push -u origin master
idpassを要求されるのでgitサービスで作成時のものを入れる  
originがgit上での位置指定、masterが位置名ぽいが、後述  
- gitサービスの管理画面でpushされているか確認する
  
SSHでpushする場合  
- vi ~/.ssh/config
```
Host github.com
  Hostname github.com
  User git 
  IdentityFile ~/.ssh/gitkey
```


#### デプロイ環境の用意
◆herokuとは  
本番公開するためのアプリケーションサービス。gitの中身をherokuにpushするとheroku上で実行してくれる
アクセス用のFQDNも払い出してくれる。   
herokuはDBがpostgresなので、defaultはsqliteだが本番時はpostgresが使用できるようにする必要がある  
  
◆アプリ側の変更  
gemfileでproduction時のみpgを読み込むよう編集する
```
group :production do
  gem 'pg', '0.20.0'
end
```
編集内容をgitに反映する
```
bundle update  
bundle install --without production  
git commit -a -m "update gemfile for heroku"  
```
◆heroku登録
id, accountを作成

作成後クラウドIDEへherokuをinstallする
```
source <(curl -sL https://cdn.learnenough.com/heroku_install)
heroku -v
vi /home/ec2-user/.bashrc 
  before
  export PATH=$PATH:$HOME/.local/bin:$HOME/bin
  after
  export PATH=$PATH:$HOME/.local/bin:$HOME/bin:/usr/local/heroku/bin
source /home/ec2-user/.bashrc
heroku -v
```

herokuでアプリを実行する
```
heroku login
heroku keys:add
heroku create
git push heroku master
```
gitのアップ先にherokuが登録されていることを目視する 

◆herokuの運用コマンド
- heroku logs
  実行ログの表示

#### 新たにappを作成し、アプリ作成の流れを確認する

◆デプロイまで新規にセットアップ
プロジェクトの作成
```
rails _5.1.6_ new toy_app
cd toy_app
```
gemfile編集によるversion固定
```
source 'https://rubygems.org'

gem 'rails',        '5.1.6'
gem 'puma',         '3.9.1'
gem 'sass-rails',   '5.0.6'
gem 'uglifier',     '3.2.0'
gem 'coffee-rails', '4.2.2'
gem 'jquery-rails', '4.3.1'
gem 'turbolinks',   '5.0.1'
gem 'jbuilder',     '2.7.0'

group :development, :test do
  gem 'sqlite3', '1.3.13'
  gem 'byebug',  '9.0.6', platform: :mri
end

group :development do
  gem 'web-console',           '3.5.1'
  gem 'listen',                '3.1.5'
  gem 'spring',                '2.0.2'
  gem 'spring-watcher-listen', '2.0.1'
end

group :production do
  gem 'pg', '0.20.0'
end

# Windows環境ではtzinfo-dataというgemを含める必要があります
# gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```
編集したgemfileの反映 
```
bundle update
bundle install --without production
```
gitの作成
```
git init
git add -A
git commit -m "init repo"
githubでror_toy_app.gitの設定
git remote add origin git@github.com:yamamo10hub/ror_toy_app.git
git push -u origin --all
```
herokuへのデプロイ
```
heroku create
git push heroku master
```

◆モデル設計とは
作成物のオブジェクトについて構造を決める
マイクロブログを作成するとして

- 投稿するユーザー と 投稿内容(マイクロポスト) の構造を決める  

|オブジェクト|エレメント|エレメントの型|description|
|---|---|---|---|
|users|id|integer|keyとしてのID|
|users|name|string|投稿者名|
|users|email|string|投稿者メールアドレス|
|---|---|---|---|
|microposts|id|integer|投稿メッセージID|
|microposts|content|txt|投稿内容|
|microposts|user_id|integer|投稿者ID|

設計の結果、ユーザー管理機能が必要であることがわかる

◆scaffoldでユーザー管理機能を追加

使用法は以下となる  
なおidについてはgenerateした時点で自動的に付与される  
この作業でモデルの作成を行ったことになる
```
rails generate scaffold User [要素１:型名] [要素２:型名] ...
rails generate scaffold User name:string email:string
rails db migrate
```
dbについては後述するのでまだおまじないでよい  
rails4以前では`bundle exec rake db:migrate`と書いたらしい  

起動
```
rails server
```

scaffoldを使うとuser管理のためのURL等も自動で作成される  
つまりどのURLで何ができるのかを把握する必要がある

|URL|アクション|用途|
|---|---|---|
|/users/|index|ユーザー一覧|
|/users/1|show|ユーザー詳細|
|/users/new|new|ユーザー作成|
|/users/1/edit|edit|ユーザー編集|

scaffoldに入力データのvalidationはない

micropostモデルを生成する
```
rails generate scaffold Micropost content:text user_id:integer
rails db:migrate
```

### 静的ウェブコンテンツの作成の流れ

rails new static-pages
git へ追加

追加後、ブランチを作成する
```
 git checkout -b static-pages
```

controllerを追加する
一度にhome, help両方のアクションを追加することもできる。(一つでもいい)
```
rails generate controller StaticPages home help
```

特定のブランチをgitにpushする
```
git push -u origin static-pages
```

作成したcontrollerを削除できる
```
rails destroy controller StaticPages home help
```

#### コンテンツのテストについて

`rails generate controller static_pages ` で作成を行うと、  
test.rbファイルの生成とactionのテスト設定が自動で記載される  
テストは `rails test` で実行される  

test/controllers/static_pages_controller_test.rb
```
  test "should get about" do
    get static_pages_about_url
    assert_response :success
  end
```
controllerのテストはアクションごとに上記のような文言で設定できる  

試しに存在しないaboutアクションのテストを記載して`rails test`を実施してみると  
下記のようにエラーがでる。
```
  1) Error:
StaticPagesControllerTest#test_should_get_about:
NameError: undefined local variable or method `static_pages_about_url' for #<StaticPagesControllerTest:0x0000000005b05770>
    test/controllers/static_pages_controller_test.rb:15:in `block in <class:StaticPagesControllerTest>'

3 runs, 2 assertions, 0 failures, 1 errors, 0 skips
```
static_pages_about_url が undefined である、といっているのでroute.rbで追加してみる

config/route
```
get  'static_pages/about'
```

rails test
```
1) Error:
StaticPagesControllerTest#test_should_get_about:
AbstractController::ActionNotFound: The action 'about' could not be found for StaticPagesController
    test/controllers/static_pages_controller_test.rb:15:in `block in <class:StaticPagesControllerTest>'

3 runs, 2 assertions, 0 failures, 1 errors, 0 skips
```
aboutアクションがないというエラーに変わった

vi static_pages_controller.rb
```
  def about
  end
```

rails test
```
1) Error:
StaticPagesControllerTest#test_should_get_about:
ActionController::UnknownFormat: StaticPagesController#about is missing a template for this request format and variant.
```
missing a template とは正常な動作として views/static_pages に表示に使用するテンプレートが無いという意味  
なのでファイルを作成する必要があるとわかる  
(このコントローラの親クラスまで確認しないと、正常動作なんてわからないけども)  

touch ./views/static_pages/about.html.erb
rails test

これで 0 errors を達成できるはず。  
つまり動作テストによるデバッグができたことになる。

ただし、rails generate controller以外で手動でアクションを足した場合、テストの前提が崩れるので
どうしているんだろうか？  
テスト内容も手書きで足しているんだろうか。  

### 動的要素の追加とコードリファクタリング

作成した静的コンテンツに動的要素を追加しつつ、コード書き換え時のリファクタリングを行ってみる  

mv layouts/application.html.erb ./layoutfile

各アクションのテスト内容に追加を行ってみる
```
  test "should get home" do
    get static_pages_home_url
    assert_response :success
++  assert_select "title", "Home | Ruby on Rails Tutorial Sample App"
  end
```
上記++ を追加することでtitleタグに"Home | Ruby on Rails Tutorial Sample App"が含まれることをテストできる  
`rails test` すると failures となる。railsで実行不能なミスだとerrorとなり、それ以外はfailureとなるようだ  

<details>
<summary>home.html.erb</summary>

```
<!DOCTYPE html>
<html>
  <head>
    <title>Home | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Sample App</h1>
    <p>
      This is the home page for the
      <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
      sample application.
    </p>
  </body>
</html>
```

</details>

<details>
<summary>help.html.erb</summary>

```
<!DOCTYPE html>
<html>
  <head>
    <title>Help | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Help</h1>
    <p>  Get help on the Ruby on Rails Tutorial at the
      <a href="https://railstutorial.jp/help">Rails Tutorial help
      page</a>.
      To get help on this sample app, see the
      <a href="https://railstutorial.jp/#ebook">
      <em>Ruby on Rails Tutorial</em> book</a>.
    </p>
  </body>
</html>
```

</details>

<details>
<summary>about.html.erb</summary>

```
<!DOCTYPE html>
<html>
  <head>
    <title>About | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>About</h1>
    <p>
      <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
      is a <a href="https://railstutorial.jp/#ebook">book</a> and
      <a href="https://railstutorial.jp/#screencast">screencast</a>
      to teach web development with
      <a href="http://rubyonrails.org/">Ruby on Rails</a>.
      This is the sample application for the tutorial.
    </p>
  </body>
</html>
```

</details>

rails test すれば 0 failures となるはず  

#### rayoutfileの機能
上記でもブラウザで表示することは可能だが、タグをいちいち書くのは労力となる

前項で外したlayoutファイルを復帰させ、rails本来のコンテンツの書き方を確認する。  
mv ./layoutfile layouts/application.html.erb  

<details>
<summary>home.html.erb</summary>

```
<% provide(:title, "Home") %>
<h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
  sample application.
</p>
```
</details>

<details>
<summary>help.html.erb</summary>

```
<% provide(:title, "Help") %>
<h1>Help</h1>
<p>  Get help on the Ruby on Rails Tutorial at the
  <a href="https://railstutorial.jp/help">Rails Tutorial help section</a>.
  To get help on this sample app, see the
  <a href="https://railstutorial.jp/#ebook"><em>Ruby on Rails Tutorial</em>
  book</a>.
</p>
```
</details>

<details>
<summary>about.html.erb</summary>

```
<% provide(:title, "About") %>
<h1>About</h1>
<p>
  <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
  is a <a href="https://railstutorial.jp/#ebook">book</a> and
  <a href="https://railstutorial.jp/#screencast">screencast</a>
  to teach web development with
  <a href="http://rubyonrails.org/">Ruby on Rails</a>.
  This is the sample application for the tutorial.
</p>
```
</details>

rails test で fail がないことを確認する

#### gitのブランチからのマージ
いままでstatic-pagesブランチで作業をしていた。
内容を作り終えたので、masterに反映したい
```
git commit -m "static-page is finish"
git checkout master
git merge static-pages
```
---
後で戻したい変更作業をする際は git commit までしておき、動作確認後 git checkout -f で戻れる  
IFとして変更作業する場合があるので、作成物に影響を与えないようにテストする場合はgitで巻き戻す  
今後はこれを「演習」と表する  

## ４章

引き続きsample_appを使用してrailsの仕組みを理解する
branchを切っておく
```
git checkout -b rails-flavored-ruby 
```

#### カスタムヘルパー
app/helpers にファイルがある

application_helper.rb(デフォルト)
```
module ApplicationHelper
end
```
application_helper.rb(変更後)
```
module ApplicationHelper
    # ページごとの完全なタイトルを返します。
  def full_title(page_title = '')
    base_title = "Ruby on Rails Tutorial Sample App"
    if page_title.empty?
      base_title
    else
      page_title + " | " + base_title
    end
  end
end
```

ここで定義したメソッドは、 app/views/layouts/application.html.erb で使用できる。
helper.rbファイルはコントローラごとに作成されるので、対応したヘルパーファイルに記載する。
viewsの下のerbに使えるかどうかはわからない。
  
内容としてはapplication内でpage_titleを指定されていないページを検知して、
指定のテキストをタイトルとして挿入してくれるヘルパーとなる。  

表示内容が変わるので、test内容を書き換える
app/test/controllers/static_pages_controller.test.rb
```
  test "should get home" do
    get static_pages_home_url
    assert_response :success
    assert_select "title", "Ruby on Rails Tutorial Sample App"
  end
```
Home | がテストテキストにないので、エラーとなるはず

testがOKとなるように作成したメソッドをerbに埋め込む
app/views/layouts/application.html.erb
```
<title><%= full_title(yield(:title)) %></title>
```

rails testでOKとなることを確認する。  
以上でヘルパーによる各ページのタイトル自動挿入ができるようになった。  

#### railsにおける文字列とメソッド

rail consoleを使ったrubyのおさらいなので、気になったとこだけ  
* rubyで文字列はダブルクォートにしないと埋め込みを展開しない
* .length で文字数
* .empty? で空かどうかを真偽値で応答する
* 論理値はそれぞれ && (and) や || (or)、! (not) オペレーターで表す
* .nil? でnilかどうかを確認できる
* puts "x is not empty" if !x.empty? というtrue時の処理を先に書く省略形がある
* .reverse で文字列の逆化ができる
* メソッド定義の引数に空の引数を指定しておくと、呼び出し時に引数が無くても応答できる
* .split で文字列をスペース区切りの配列に格納できる  
.split("x")で区切り文字をxに変えることができる
* 配列確認用のメソッドがいくつかある。a.first , a.second , a.last , a.sort , a.shuffle など
"!"をつけると元のオブジェクトまで影響を及ぼす
* "{|s|}"と"do |s| end"は同じブロック要素
* .inspect でそのオブジェクトの構成がわかる

#### hashとシンボル
```
基本的なhash
user = { "first_name" => "Hoge Fuga", "email" => "hoge@example.com" }
キーにシンボルを使用したhash
user = { :name => "Hoge Fuga", :email => "hoge@example.com" }
シンボルを使う利点がわからない  "を打たなくていい、とか？

hashはネストできる
params[:user] = { name: "Hoge Fuga", email: "hoge@example.com" }
```

#### railsのコードで省略記法を学ぶ

app/views/layouts/application.html.erb
```
<%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
```
上記は丸括弧を省略したstylesheet_link_tagメソッドの呼び出しで、後半部は引数となる  
後半部の一部引数は : からhashであるように見えるが、{} がない。  
メソッドの最後の引数がhashである場合、{}は省略していいらしい。ひどい  
  
結果的にerbへの埋め込みは以下のようなhtmlになる
```
<link data-turbolinks-track="true" href="/assets/application.css"
media="all" rel="stylesheet" />
```

#### railsでクラスの継承を確認する

ただのstring変数のオブジェクトでクラス階層を確認してみる
```
2.6.0 :025 > s = String.new("foobar")
 => "foobar" 
2.6.0 :026 > s.class
 => String 
2.6.0 :027 > s.class.superclass
 => Object 
2.6.0 :028 > s.class.superclass.superclass
 => BasicObject 
2.6.0 :029 > s.class.superclass.superclass.superclass
 => nil 
```
頂点としてBasicObjectが確認できる。

Stringクラスを継承したWordクラスを作成し、インスタンスメソッドpalindrome?を作る
```
rails console
>> class Word
>>   def palindrome?
>>     self == self.reverse
>>     #self == reverse ※上と同一
>>   end
>> end
=> :palindrome?
s = Word.new("level")
s.palindrome?
=> true
s.length
=> 5
```

railsアプリ上でクラスの作成と実行を試してみる
```
vim example_user.rb
------
class User
  attr_accessor :name, :email
  
  def initialize(attributes = {})
    @name = attributes[:name]
    @email = attributes[:email]
  end
  
  def formatted_email
    "#{@name} <#{@email}>"
  end
  
end
----------

> require './example_user'
 => true 
> example = User.new
 => #<User:0x0000000004c16750 @name=nil, @email=nil> 
> example.name = "Hoge Fuga"
 => "Hoge Fuga" 
> example.email = "hoge@test.com"
 => "hoge@test.com" 
> example.formatted_email
 => "Hoge Fuga <hoge@test.com>" 
```

## ５章

### flameworkによるwebレイアウトを実践し理解する

#### basicなレイアウトを作成し表示してみる

gitでブランチを切る  
git checkout -b filling-in-layout

レイアウトの作成を行う  
app/views/layouts/application.html.erb
<details>
<summary>内容</summary>

```
<!DOCTYPE html>
<html>
  <head>
    <title><%= full_title(yield(:title)) %></title>
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag    'application', media: 'all',
                               'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application',
                               'data-turbolinks-track': 'reload' %>
    <!--[if lt IE 9]>
      <script src="//cdnjs.cloudflare.com/ajax/libs/html5shiv/r29/html5.min.js">
      </script>
    <![endif]-->
  </head>
  <body>
    <header class="navbar navbar-fixed-top navbar-inverse">
      <div class="container">
        <%= link_to "sample app", '#', id: "logo" %>
        <nav>
          <ul class="nav navbar-nav navbar-right">
            <li><%= link_to "Home",   '#' %></li>
            <li><%= link_to "Help",   '#' %></li>
            <li><%= link_to "Log in", '#' %></li>
          </ul>
        </nav>
      </div>
    </header>
    <div class="container">
      <%= yield %> #ここにURL別のviewsの中身が展開される
    </div>
  </body>
</html>
```

</details>

#### 画像を置いて表示させる

homeで表示する内容を作る  
app/views/static_pages/home.html.erb
```
<div class="center jumbotron">
  <h1>Welcome to the Sample App</h1>
  <h2>
    This is the home page for the
    <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
    sample application.
  </h2>
  <%= link_to "Sign up now!", '#', class: "btn btn-lg btn-primary" %>
</div>

<%= link_to image_tag("rails.png", alt: "Rails logo"),
            'http://rubyonrails.org/' %>
```

表示用画像のダウンロード
```
curl -o app/assets/images/rails.png -OL railstutorial.jp/rails.png
```
app/assets/images/ が link_to image_tag における / となる

#### レイアウトのためのCSSとbootstrap
 
Gemfile
```
gem 'rails',          '5.1.6'
gem 'bootstrap-sass', '3.3.7'
```
    bundle install

新規作成  


<details>
<summary>app/assets/stylesheets/custom.scss</summary>

```
@import "bootstrap-sprockets";
@import "bootstrap";

/* universal */

body {
  padding-top: 60px;
}

section {
  overflow: auto;
}

textarea {
  resize: vertical;
}

.center {
  text-align: center;
}

.center h1 {
  margin-bottom: 10px;
}

/* typography */

h1, h2, h3, h4, h5, h6 {
  line-height: 1;
}

h1 {
  font-size: 3em;
  letter-spacing: -2px;
  margin-bottom: 30px;
  text-align: center;
}

h2 {
  font-size: 1.2em;
  letter-spacing: -1px;
  margin-bottom: 30px;
  text-align: center;
  font-weight: normal;
  color: #777;
}

p {
  font-size: 1.1em;
  line-height: 1.7em;
}

/* header */

#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
}

#logo:hover {
  color: #fff;
  text-decoration: none;
}

/* footer */

footer {
  margin-top: 45px;
  padding-top: 5px;
  border-top: 1px solid #eaeaea;
  color: #777;
}

footer a {
  color: #555;
}

footer a:hover {
  color: #222;
}

footer small {
  float: left;
}

footer ul {
  float: right;
  list-style: none;
}

footer ul li {
  float: left;
  margin-left: 15px;
}
```

</details>

bootstrapとCSSを使用してサイトの見た目が変わったはず。  
コメント機能を駆使して各項目が表示にどのような影響を与えるか調べる  


#### rails の partialについて

httpの206のことではない。  
対象erbファイル内の記載を別のファイルに分けることができる  




<details>

<summary>app/views/layouts/application.html.erb</summary>
```
<!DOCTYPE html>
<html>
  <head>
    <title><%= full_title(yield(:title)) %></title>
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag    'application', media: 'all',
                               'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application',
                               'data-turbolinks-track': 'reload' %>
    <%= render 'layouts/shim' %>
  </head>
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <%= yield %>
      <%= render 'layouts/footer' %>
    </div>
  </body>
</html>
```
</details>

```
<%= render 'layouts/shim' %>
<%= render 'layouts/header' %>
```
render で記載内容を分割したファイルを指定して呼び出している


<details>

<summary>layouts/_shim.html.erb</summary>
```
<!--[if lt IE 9]>
  <script src="//cdnjs.cloudflare.com/ajax/libs/html5shiv/r29/html5.min.js">
  </script>
<![endif]-->
```
</details>


<details>

<summary>layouts/_header.html.erb</summary>
```
<header class="navbar navbar-fixed-top navbar-inverse">
  <div class="container">
    <%= link_to "sample app", '#', id: "logo" %>
    <nav>
      <ul class="nav navbar-nav navbar-right">
        <li><%= link_to "Home",   '#' %></li>
        <li><%= link_to "Help",   '#' %></li>
        <li><%= link_to "Log in", '#' %></li>
      </ul>
    </nav>
  </div>
</header>
```
</details>

<details>

<summary>layouts/_footer.html.erb</summary>
```
<footer class="footer">
  <small>
    The <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
    by <a href="http://www.michaelhartl.com/">Michael Hartl</a>
  </small>
  <nav>
    <ul>
      <li><%= link_to "About",   '#' %></li>
      <li><%= link_to "Contact", '#' %></li>
      <li><a href="http://news.railstutorial.org/">News</a></li>
    </ul>
  </nav>
</footer>
```
</details>

#### scssの設置場所とアセット

app/assets: 現在のアプリケーション固有のscssの置き場
lib/assets: 開発チームによって作成されたライブラリ用のscssの置き場
vendor/assets: サードパーティが生成するscssの置き場

railsアプリはチームが組めるらしい。(異なるアプリ間で共通のライブラリを適用できるということか)

実際にアセット(SCSS群の管理)をしているのはsprocketsというGemなので、これがないと使用できない

#### アセットパイプライン

必要なアセットをディレクトリに配置しておくとRailsはさまざまなプリプロセッサエンジンを
介してそれらを実行し、ブラウザに配信できるようにマニフェストファイルを用いて結合し、  
サイトテンプレート用に準備する。  
Railsはどのプリプロセッサを使うのかをファイル名の拡張子を使って判断している  
(scss, coffee, erb)  

アセットパイプラインの良い点は、開発者がファイルをpartialにしても、本番で生成されるときは
application.cssや(javascripts.jsにすべてまとめてくれる点である  
ファイルがpartialだと読みやすいが、表示速度に影響がでる。  

#### scssをsassで記述する

Sass はスタイルシートを記述するための言語で、ネストと変数という便利な機能がある  

◆ネスト

<details>
<summary>例１</summary>

```
/* before */
.center {
  text-align: center;
}
.center h1 {
  margin-bottom: 10px;
}

/* after */
.center {
  text-align: center;
  h1 {
    margin-bottom: 10px;
  }
}
```

</details>

classとclass+特定のタグ で２ブロック書いている場合は、`タグ {}` でclassのみの記述に  
ブロック要素をネストすることができる  

<details>
<summary>例２</summary>

```
/* before */
#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
}

#logo:hover {
  color: #fff;
  text-decoration: none;
}

/* after */
#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
  &:hover {
    color: #fff;
    text-decoration: none;
  }
}
```

</details>

属性を持つスタイルをネストする場合は、`&:属性{}` としてネストする


◆変数  
変数を定義することで、繰り返しを除去することができる
また、bootstrapなどcssフレームワークにはデフォルトで定義されている変数があり、  
それを使用するには変数で呼び出すことになる。
```
/* define */
$light-gray: #777;

/* use case */
color: $light-gray;
```

#### Contactページを新規につくり、レイアウトとの兼ね合いを復習する

作成予定のページのテストを先に作成する  
test/controllers/static_pages_controller_test.rb
```
  test "should get contact" do
    get static_pages_contact_url
    assert_response :success
    assert_select "title", "Contact | Ruby on Rails Tutorial Sample App"
  end
```

rails test を実行し、errorsがカウントされることを確認する。

app/config/routes.rb
```
#以下を追加
get 'static_pages/contact'
```

app/controllers/static_pages_controller.rb
```
#以下を追加
def contact
end
```

app/views/static_pages/contact.html.erb (新規作成)
```
<% provide(:title, 'Contact') %>
<h1>Contact</h1>
<p>
  Contact the Ruby on Rails Tutorial about the sample app at the
  <a href="https://railstutorial.jp/contact">contact page</a>.
</p>
```

#### レイアウトのリンク

名前付きルートを使うのが作法らしい  
名前付きルートとは `config/routes.rb` に記載するときのURLとアクセス先の紐付けのこと

|ページ名|URL|名前付きルート|
|---|---|---|
|Home|/	|root_path|
|About|/about|about_path|
|Help|/help|help_path|
|Contact|/contact|contact_path|
|Sign up|/signup|signup_path|
|Log in|/login|login_path|

routes.rb
```
# before
  root 'static_pages#home'
  get 'static_pages/home'
  get 'static_pages/help'
  get  'static_pages/about'
  get 'static_pages/contact'
# after
  root 'static_pages#home'
  get '/help', to: 'static_pages#help'
  get  '/about', to: 'static_pages#about'
  get '/contact', to: 'static_pages#contact'
```

routesを変更したので、testを修正する
app/test/controllers/static_pages_controller_test.rb

```
# before
# after
get root_path
get static_pages_home_url
get help_path
get static_pages_help_url
get about_path
get static_pages_about_url
get contact_path
get static_pages_contact_url
```
ルーティングの変更で名前付きルート (*_path) が使えるようになっているらしい。  
/help == help_path となるのが名前付きルート、ということなのだろう。  


上記を加味すると、以下のファイルで'#'になっているリンク先を名前つきルートに変更する  
app/views/layouts/application.html.erb
app/views/layouts/_header.html.erb
app/views/layouts/_footer.html.erb
app/views/layouts/_shim.html.erb
```
<li><%= link_to "Home",   'root_path' %></li>
<li><%= link_to "Help",   'help_path' %></li>
<li><%= link_to "Log in", '#' %></li>
```
loginはこの後変更する


#### リンクのテスト方法

IntegrationTest(統合テスト)用のgenerateをする
```
rails generate integration_test site_layout
```
app/test/integration/site_layout_test.rb が作成される
```
require 'test_helper'

class SiteLayoutTest < ActionDispatch::IntegrationTest

  test "layout links" do
    get root_path
    assert_template 'static_pages/home'
    assert_select "a[href=?]", root_path, count: 2
    assert_select "a[href=?]", help_path
    assert_select "a[href=?]", about_path
    assert_select "a[href=?]", contact_path
  end
end
```

assert_selectの例文

|Code|マッチするHTML|
|---|---|
|assert_select "div"|<div>foobar</div>|
|assert_select "div", "foobar"|<div>foobar</div>|
|assert_select "div.nav"|<div class="nav">foobar</div>|
|assert_select "div#profile"|<div id="profile">foobar</div>|
|assert_select "div[name=yo]"|<div name="yo">hey</div>|
|assert_select "a[href=?]", ’/’, count: 1|<a href="/">foo</a>|
|assert_select "a[href=?]", ’/’, text: "foo"|<a href="/">foo</a>|


テストの概要  
1. ルートURL (Homeページ) にGETリクエストを送る.
2. 正しいページテンプレートが描画されているかどうか確かめる.
3. Home、Help、About、Contactの各ページへのリンクが正しく動くか確かめる.

testの実行方法
```
# integration部分のテスト
rails test:integration
# 全テスト項目のテスト
rails test
```
失敗したときは、testの内容の行数とエラー内容で判断するひつようがある

### User機能の実装

機能ごとにcontrollerを作る
```
rails generate controller Users new
```

現時点で /users/new のページができている。generateコマンドで new をつけたため  

/users/new から /signup にURLを変更したい  

routes.rb
```
get '/signup', to: 'users#new'
```

users_controller_test.rb
```
get signup_path
```

home.html.erb
```
#before
  <%= link_to "Sign up now!", '#', class: "btn btn-lg btn-primary" %>
#after
  <%= link_to "Sign up now!", signup_path, class: "btn btn-lg btn-primary" %>
```

new.html.erb
```
# before
<h1>Users#new</h1>
<p>Find me in app/views/users/new.html.erb</p>
# after
<% provide(:title, 'Sign up') %>
<h1>Sign up</h1>
<p>This will be a signup page for new users.</p>
```

GETtestを可能にする  

test/integration/site_layout_test.rb
```
    get contact_path
    assert_select "title", full_title("Contact")
    get signup_path
    assert_select "title", full_title("Sign up")
```

test/test_helper.rb
```
class ActiveSupport::TestCase
include ApplicationHelper
```

rails test でerrorやfailがないことを確認する

## ６章

### Userのモデル作成

userの作成や保存方法、データのvalidationを学ぶ

git checkout -b modeling-users

モデルの作成
```
rails generate model User name:string email:string
```
* 作成すると db/migrate/yyyymmddhhmmss_create_users.rb が生成される  
* この時点でDBは生成されない  
* このファイルをマイグレーションファイルという  
初回作成時以外にもgenerateすることがあり(カラムの追加など)、その都度タイムスタンプもつ  
別のマイグレーションファイルを生成し、DBへの変更履歴となる

* ファイル内部ではgenerateで指定していない２つのエレメントもDBのtableに書き込むプログラムが付与される。
`timestumps`はcreated_at:datetime updated_at:datetime という情報を持つ
またこれら２つを`マジックカラム`と呼ぶ  

マイグレーションを実行する
```
rails db:migrate
```

実行するとdb/development.sqlite3というファイルが生成される  
sqliteはファイル一つでDBなので、DBが作成されたということになる  
また db/schema.rb というファイル内にデータベース構造と追跡するためのプログラムが生成される  

ロールバックする  
マイグレーションの取り消しが可能
```
rails db:rollback
```
再度migrateすればロールバック前に戻るので実行しておく

model ファイル  

上記DB周りの生成以外に、modelファイルが生成される
app/models/user.rb
```
class User < ApplicationRecord
end
```
ApplicationRecord < ActiveRecord::Base なので上記はそれらのクラスを継承している  
よって親クラスのメソッドを使って中身を作るのだが、中身はプリセットされているため覚えるしかない。  

Userを使用したデータ登録をconsoleで試す
```
rails console --sandbox
# sandboxで実行しないと、これから作成するテストuserが実際にDBに書き込まれてしまう

2.6.0 :001 > User.new
=> #<User id: nil, name: nil, email: nil, created_at: nil, updated_at: nil>
# User.newに定義されるDBカラムが表示される

2.6.0 :002 > user = User.new(name: "hoge", email: "fuga@hoge.com")
 => #<User id: nil, name: "hoge", email: "fuga@hoge.com", created_at: nil, updated_at: nil> 
# 登録内容が返る

2.6.0 :003 > user.valid?
 => true
# オブジェクトuserについて、valid?メソッドはvalidationを掛けることができる。

2.6.0 :004 > user.save
   (0.1ms)  SAVEPOINT active_record_1
  SQL (1.4ms)  INSERT INTO "users" ("name", "email", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["name", "hoge"], ["email", "fuga@hoge.com"], ["created_at", "2019-05-01 15:55:37.808870"], ["updated_at", "2019-05-01 15:55:37.808870"]]
   (0.1ms)  RELEASE SAVEPOINT active_record_1
 => true 
# 実際にDBに書き込む
# 書き込みに成功するとtrueが返る
# また書き込んだ時にマジックカラムに日時データが挿入される

2.6.0 :007 > user.name.class
 => String 
2.6.0 :008 > user.email.class
 => String 
2.6.0 :009 > user.created_at.class
 => ActiveSupport::TimeWithZone 
# マジックカラムの２つは時間を表示する専用のクラスになっている
# 各カラムはオブジェクトとして扱うことができる

user.destroy
# そのオブジェクトの削除ができる、ただしメモリには残留してしまう
# (destroy後もuserオブジェクトは確認できる)
# またオブジェクトとDBとのつながりを確認する方法が必要なため、次項で学ぶ
```

ユーザーオブジェクトの検索
```
# id=1のユーザを探した場合
2.6.0 :011 > User.find(1)
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
 => #<User id: 1, name: "hoge", email: "fuga@hoge.com", created_at: "2019-05-01 15:55:37", updated_at: "2019-05-01 15:55:37"> 

# 探して失敗した場合
2.6.0 :012 > User.find(3)
  User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 3], ["LIMIT", 1]]
Traceback (most recent call last):
        1: from (irb):12
ActiveRecord::RecordNotFound (Couldn't find User with 'id'=3)

# その他の探し方
2.6.0 :013 > User.find_by(email: "fuga@hoge.com")
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."email" = ? LIMIT ?  [["email", "fuga@hoge.com"], ["LIMIT", 1]]
 => #<User id: 1, name: "hoge", email: "fuga@hoge.com", created_at: "2019-05-01 15:55:37", updated_at: "2019-05-01 15:55:37"> 
2.6.0 :014 > User.first
  User Load (0.2ms)  SELECT  "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT ?  [["LIMIT", 1]]
 => #<User id: 1, name: "hoge", email: "fuga@hoge.com", created_at: "2019-05-01 15:55:37", updated_at: "2019-05-01 15:55:37"> 
2.6.0 :015 > User.all
  User Load (0.2ms)  SELECT  "users".* FROM "users" LIMIT ?  [["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<User id: 1, name: "hoge", email: "fuga@hoge.com", created_at: "2019-05-01 15:55:37", updated_at: "2019-05-01 15:55:37">]>
```

User.all[1]
等でインデックスを指定してアクセスはできるが、allは配列ではない  
id1はインデックス[0]となる  
```
2.6.0 :026 > User.all.class
 => User::ActiveRecord_Relation
```

User.all.lengthでレコード長がintで返ってくる。  

ユーザーオブジェクトの更新
```
> user.email = "foo@bar.com"
# 新しい情報を代入すれば更新される
> user.reload.email
# saveするとDBが更新されるが、reloadするとDBから前の値を読み込みrollbackできる
> user.updated_at
# saveすると更新日時が現在のものになる
> user.update.attribute(name: "fuga" , email: "hoge@fuga.com")
# trueが返ってきた場合(引数に問題がない場合)、更新とDBへの反映を同時に行う
> user.update.attribute(:email , "fuga@hoge.jp")
# すべてのカラムが対象でない場合は上記のよう別のメソッドとなる
```

ユーザーの検証  
不正や空の名前をエラーとしたり、emailがフォーマットとなっていることを確認したい  
ActiveRecordにはvalidationという機能があり、それを使って実現する  

test/models/user_test.rb
```
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end

  test "should be valid" do
    assert @user.valid?
  end
end
```
テスト用のユーザをsetupメソッド内に作る  
valid?でそのユーザーの有効性をチェックできる  
valid?でチェックする内容は後述する  

rails test:models  

存在性の検証方法  
空の入力を防ぐ  

test/models/user_test.rb
```
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end

  test "should be valid" do
    assert @user.valid?
  end

  test "name should be present" do
    @user.name = "     "
    assert_not @user.valid?
  end
end
```

testについて
```
https://qiita.com/duka/items/2d724ea2226984cb544f
http://ruby.studio-kingdom.com/rails/guides/testing#2
```

`assert_not @user.valid?` ではヴァリデーションが失敗すればtrueとなる

userにヴァリデーションをかける  
app/models/user.rb
```
class User < ApplicationRecord
  validates :name, presence: true
end
```
これでUserクラスにvalidateが有効となったので、consoleで確認してみる
```
r.yamamoto:~/environment/sample_app (modeling-users) $ rails console --sandbox
Running via Spring preloader in process 16912
Loading development environment in sandbox (Rails 5.1.6)
Any modifications you make will be rolled back on exit
2.6.0 :001 > user = User.new(name: "" , email: "hoge@fuga.com")
 => #<User id: nil, name: "", email: "hoge@fuga.com", created_at: nil, updated_at: nil> 
2.6.0 :002 > user.valid?
 => false 
2.6.0 :003 > user.errors.full_messages
 => ["Name can't be blank"] 
r.yamamoto:~/environment/sample_app (modeling-users) $ rails console --sandbox
Running via Spring preloader in process 21673
Loading development environment in sandbox (Rails 5.1.6)
Any modifications you make will be rolled back on exit
2.6.0 :001 > a = User.new
 => #<User id: nil, name: nil, email: nil, created_at: nil, updated_at: nil> 
2.6.0 :002 > a.valid?
 => false 
2.6.0 :003 > a.errors.messages
 => {:name=>["can't be blank"], :email=>["can't be blank"]} 
2.6.0 :004 > a.errors.messages[:name]
 => ["can't be blank"]
```
名前が空白であることをvalidがチェックしている  
errorsでfailの原因が確認できる  

◆長さに制限をかける  

test/models/user_test.rb
```
  test "name should not be too long" do
    @user.name = "a" * 51
    assert_not @user.valid?
  end

  test "email should not be too long" do
    @user.email = "a" * 244 + "@example.com"
    assert_not @user.valid?
  end
```
あまりテスト内のuser情報上書きに意味はない  
assert_not がついている以上、追記したテスト２つはfailする

models/user.rb
```
validatesに追記を行う
  validates :name, presence: true, length: { maximum: 50 }
  validates :email, presence: true, length: { maximum: 255 }
```

`rails test:models` するとfailしないはず  
テストで設定された長いuserとemailがvalidatesでブロックされたことでテストが通ったことになる  

◆フォーマットの検証  
emailは内容までチェックして有効なメールアドレスか確認したい。

test/models/user_test.rb
```
  test "email validation should accept valid addresses" do
    valid_addresses = %w[user@example.com USER@foo.COM A_US-ER@foo.bar.org
                         first.last@foo.jp alice+bob@baz.cn]
    valid_addresses.each do |valid_address|
      @user.email = valid_address
      assert @user.valid?, "#{valid_address.inspect} should be valid"
    end
  end
```

Userクラスへのバリデーションのかけ方
app/models/user.rb
```
class User < ApplicationRecord
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX }
end
```

◆ユニークさの検証

test/models/user_test.rb
```
  test "email addresses should be unique" do
    duplicate_user = @user.dup
    duplicate_user.email = @user.email.upcase
    @user.save
    assert_not duplicate_user.valid?
  end
```
* .dup は dupulicate_user を @user の複製オブジェクトにするメソッド
* mailaddressは大文字小文字を区別しないので、.upcaseでいったんアドレスをすべて大文字に変更している

app/models/user.rb
```
class User < ApplicationRecord
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
end
```
`uniqueness:true`で検証できるが、大文字小文字の区別を無効にする場合は  
`uniqueness: { case_sensitive: false }` となる

これでプログラム上ではメールアドレスはユニークになる  
しかし通信上ではメールアドレスが重複しうる仕様になっている  
DBに書き込む前に素早く同じリクエストが来た場合、DBにはそのアドレスはないので、  
二回目のリクエストがバリデーションをすり抜ける。  

DBでも同一メールアドレスを防ぐ仕組みがあればよい  
以下のようにDBの構成を変更する
```
# DBの変更内容を書き込むmigrationfileを生成する
rails generate migration add_index_to_users_email

# db/migrate/[timestamp]_add_index_to_users_email に以下を記載
add_index :users, :email, unique: true

# DBへ反映させる
rails db:migrate
```

当作業でtestが通らなくなる  
test/fixtures/users.yml で users のチェックをしているらしい  
```
one:
  name: MyString
  email: MyString

two:
  name: MyString
  email: MyString
```
上記の記載があるが、消すとテストがとおる
ひとまず消しておく  

DBにmailaddressを保存する前に小文字に変えたい  
前のテストでは大文字にしているのに、と思うが教材だからだろう多分  
app/models/user.rb
```
class User < ApplicationRecord
  before_save { self.email = email.downcase }
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
end
```
before_save { 内容 } で保存前に処理実行ができるんだろう  

◆パスワードのダイジェスト化
```
class User < ApplicationRecord
  has_secure_password
end
```
* ハッシュ化したパスワードを、DB内のpassword_digestという属性に保存できるようになる
* 2つのペアの仮想的な属性 (passwordとpassword_confirmation) が使えるようになる。
また、存在性と値が一致するかどうかのバリデーションも追加される
* authenticateメソッドが使えるようになる 
(引数の文字列がパスワードと一致するとUserオブジェクトを、間違っているとfalseを返すメソッド)

has_secure_password を使えるようにするには、条件が一つある  
モデル内にpassword_digest(string)という属性が含まれていること  
なので、またmigrationファイルを生成してDBを作り替える必要がある  

追加でmigrationファイル生成時に名前の末尾をto_usersにすると、usersテーブルにカラムを追加する  
マイグレーションがRailsによって自動的に作成される  

ダイジェストを使うためのdbマイグレーション
```
rails generate migration add_password_digest_to_users password_digest:string
rails db:migrate
```

Gemfile
```
gem 'rails',          '5.1.6'
gem 'bcrypt',         '3.1.12'
```

`bundle intall`  

app/models/user.rb を書き換える
```
class User < ApplicationRecord
  before_save { self.email = email.downcase }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
end
```

パスワードのバリデーションチェックがダイジェスト化した部分にもテストされると、  
テストではじかれるので、テスト用ユーザーの設定を変更する。  

test/models/user_test.rb のdef setupを書き換える
```
  def setup
    @user = User.new(name: "Example User", email: "user@example.com",
                     password: "foobar", password_confirmation: "foobar")
  end
```

準備ができたので、ダイジェスト化したパスワード使用してみる  
```
rails test
```

◆パスワードの最小文字数を設ける  

テストファイルの最後に追加する  
test/models/user_test.rb
```
  test "password should be present (nonblank)" do
    @user.password = @user.password_confirmation = " " * 6
    assert_not @user.valid?
  end

  test "password should have a minimum length" do
    @user.password = @user.password_confirmation = "a" * 5
    assert_not @user.valid?
  end
```

`@user.password = @user.password_confirmation = "a" * 5` 1,2項に3項目を代入できる書き方  
単純にテスト項目として空白とaを指定文字数いれたパスワードを二種類作っている  

実際の制限付与は以下で行っている。  

app/models/user.rb
```
class User < ApplicationRecord
  before_save { self.email = email.downcase }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }
end
```

◆Consoleでユーザーの作成と認証  

UIでユーザー管理ができないが、動作を確認したいときは consoleで作成することになるだろう  

```
rails console
User.create(name: "hoge fuga", email: "hoge@fuga.com", password: "hogefuga", password_confirmation: "hogefuga")
```

環境はDevelopmentなので、development.sqlite3 ファイルに書き込まれる  

passwordがダイジェスト化されていることを確認してみる  
```
user = User.find_by(email: "hoge@fuga.com")
user.password_digest
 => "$2a$10$UPnQ7vS7bnrLWthE/pMOOuYJMiN.Drno1AZCUy8cB7wyYXneaN.E6"
```
認証用に用意されたメソッドauthenticateを試す
```
2.6.0 :009 > user.authenticate("not_the_right_password")
 => false 
2.6.0 :010 > user.authenticate("foobaz")
 => false 
2.6.0 :011 > user.authenticate("hogefuga")
 => #<User id: 1, name: "hoge fuga", email: "hoge@fuga.com", created_at: "2019-05-13 02:47:39", updated_at: "2019-05-13 02:47:39", password_digest: "$2a$10$UPnQ7vS7bnrLWthE/pMOOuYJMiN.Drno1AZCUy8cB7w..."> 
2.6.0 :013 > !!user.authenticate("hogefuga")
 => true
```
正しいパスワードを入れるとユーザー情報を返し、違う場合はfalseが返る  
!!で論理値変換を行うとtrueが返ってくるので、認証の結果確認に便利な状態となる  

```
rails test
git add -A
git commit -m "userdb is finished"
git checkout master
git merge modeling-users
git push
```

## ７章
### ユーザー登録

### ユーザー表示

#### デバッグとRailsの環境について
ところでRailsの環境とは何だろう?  

◆ Railsのテスト環境について  
* テスト環境 (test)
* 開発環境 (development)
* 本番環境 (production) 

consoleで `Rails.env` で現在の環境がわかる  
consoleのデフォルトはdevelopment  
herokuにデプロイして動いているアプリは当然本番環境となる  


本番環境では表示されないデバッグメッセージを埋めることも可能  
app/views/layouts/application.html.erb
```
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <%= yield %>
      <%= render 'layouts/footer' %>
      <%= debug(params) if Rails.env.development? %>
    </div>
  </body>
```
debugとparams変数を使うため、bodyの中身を書き換える  
ifで本番環境では表示されないように条件を付けている  
  
デバッグ表示の整形のためのscssの変更
app/assets/stylesheets/custom.scss
```
/* 上についか */
@mixin box_sizing {
  -moz-box-sizing:    border-box;
  -webkit-box-sizing: border-box;
  box-sizing:         border-box;
}
/* 下についか */
/* miscellaneous */

.debug_dump {
  clear: both;
  float: left;
  width: 100%;
  margin-top: 45px;
  @include box_sizing;
}
```
上に追加したmixinとはscssの機能で、複数のCSS定義をまとめて適用できるらしい。  
整合性合わない場合はどうなるのか気になる  

`rails server`で起動してブラウザで表示すると、下部に情報がでるようになった。  

```
--- !ruby/object:ActionController::Parameters
parameters: !ruby/hash:ActiveSupport::HashWithIndifferentAccess
  controller: static_pages
  action: about
permitted: false
```

◆ Userリソースについて  
アプリにおけるUserデータの管理のためには  
作成、表示、更新、削除が必要。 これらをRESTで実現するためにはHTTPの  
POST、GET、PATCH、DELETEを使用する。  
 
ブラウザで架空のpathへアクセスすると`Routing Error`ページが表示されるが、
内容に現在使用できるpathが表示されているので確認しておく  

config/routes.rb
```
Rails.application.routes.draw do
  root 'static_pages#home'
  get  '/help',    to: 'static_pages#help'
  get  '/about',   to: 'static_pages#about'
  get  '/contact', to: 'static_pages#contact'
  get  '/signup',  to: 'users#new'
  resources :users
end
```
`resources :users`を追加することでuserに関連した作成、表示、更新、削除が  
あらかじめ組み込まれた機能によって可能になる  

使用可能なアプリに対する変更とURLとHTTPメソッドの組み合わせをまとめると、以下のようになる。
|リクエスト|URL|アクション|名前付きルート|用途|
|---|---|---|---|---|
|GET|/users|index|users_path|すべてのユーザーを一覧するページ|
|GET|/users/1|show|user_path(user)|特定のユーザーを表示するページ|
|GET|/users/new|new|new_user_path|ユーザーを新規作成するページ (ユーザー登録)|
|POST|/users|create|user_path(user)|ユーザーを作成するアクション|
|GET|/users/1/edit|edit|edit_user_path(user)|id=1のユーザーを編集するページ|
|PATCH|/users/1|update|user_path(user)|ユーザーを更新するアクション|
|DELETE|/users/1|destroy|user_path(user)|ユーザーを削除するアクション|

改めて架空のpathにブラウザでアクセスすると、resourcesの追加で  
上記のpathが認識できるようになっていることがわかる  
  
ルーティングは有効になったが、/users/1を表示するコンテンツがないので作成する  
ファイルも存在しないので、手動で作成する  
app/views/users/show.html.erb
```
<%= @user.name %>, <%= @user.email %>
```

showメソッドを追加  
app/controllers/users_controller.rb
```
  def show
    @user = User.find(params[:id])
  end
```
ブラウザでアクセスすると、id=1のユーザー名とメールアドレスが表示される

デバッグ用のパラメーター表示は、こうなる
```
--- !ruby/object:ActionController::Parameters
parameters: !ruby/hash:ActiveSupport::HashWithIndifferentAccess
  controller: users
  action: show
  id: '1'
permitted: false
```

演習でshowメソッドに別の情報表示を試した
app/views/users/show.html.erb
```
<%= @user.name %>, <%= @user.email %> <br>
<%= @user.created_at %>, <%= @user.updated_at %> <br>
<%= Time.now %>
```
time.nowはUTCが0なので、変える方法があるかもしれない  

◆debuggerメソッド  
app/controllers/users_controller.rb
```
class UsersController < ApplicationController

  def show
    @user = User.find(params[:id])
    debugger
  end

  def new
  end
end
```

ブラウザで/user/1にアクセスすると、`debugger`を設置した箇所で  
railsserverの表示部分にコマンド入力できるようになる  

```
[1, 9] in /home/ec2-user/environment/sample_app/app/controllers/users_controller.rb
   1: class UsersController < ApplicationController
   2:   def show
   3:     @user = User.find(params[:id])
   4:     debugger
=> 5:   end
   6:   
   7:   def new
   8:   end
   9: end
(byebug) @user.name
"hoge fuga"
(byebug) @user.email
"hoge@fuga.com"
(byebug) params[:id]
"1"
(byebug)
# Ctrl + D でエスケープ
```
使い終わったら`debugger`は消しておく

### 7-1 Gravatar

メールアドレスと写真を紐づける機能  
  
ひとまずgravatarのデータを表示できるように埋め込み変数を記載する  
app/views/users/show.html.erb
```
<% provide(:title, @user.name) %>
<h1>
  <%= gravatar_for @user %>
  <%= @user.name %>
</h1>
```

showから呼び出せる関数としてgravatarからのデータ取得用メソッドを用意する  
app/helpers/users_helper.rb
```
def gravatar_for(user)
  gravatar_id = Digest::MD5::hexdigest(user.email.downcase)
  gravatar_url = "https://secure.gravatar.com/avatar/#{gravatar_id}"
  image_tag(gravatar_url, alt: user.name, class: "gravatar")
end
```
/users/1 で画像が表示されるはず。  


tutorialで用意されているgravaterの画像を呼び出してみる  
```
r.yamamoto:~/environment/sample_app (sign-up) $ rails console
Running via Spring preloader in process 5554
Loading development environment (Rails 5.1.6)
2.6.0 :001 > user = User.first
2.6.0 :003 > user.update_attributes(name: "Example User", email: "example@railstutorial.org" , password: "foobar" , password_confirmation: "foobar" )                                                                               
   (0.1ms)  begin transaction
  User Exists (0.2ms)  SELECT  1 AS one FROM "users" WHERE LOWER("users"."email") = LOWER(?) AND ("users"."id" != ?) LIMIT ?  [["email", "example@railstutorial.org"], ["id", 1], ["LIMIT", 1]]
  SQL (2.5ms)  UPDATE "users" SET "name" = ?, "email" = ?, "updated_at" = ?, "password_digest" = ? WHERE "users"."id" = ?  [["name", "Example User"], ["email", "example@railstutorial.org"], ["updated_at", "2019-05-28 10:08:38.570384"], ["password_digest", "$2a$10$21XXbKkVS5sWMK6YUPzMdOpFOqEwbzUU9ij4qZ3VnC.a2Obn81fPi"], ["id", 1]]
   (6.8ms)  commit transaction
 => true 
2.6.0 :004 > 
```
gravatarはemailで応答すべき画像を判別しているので、user1のemailを変えてやればよい  
/users/1を表示すると画像が変わることがわかる

<details>

<summary>変更内容</summary>

app/views/users/show.html.erb
```
<% provide(:title, @user.name) %>
<div class="row">
  <aside class="col-md-4">
    <section class="user_info">
      <h1>
        <%= gravatar_for @user %>
        <%= @user.name %>
      </h1>
    </section>
  </aside>
</div>
```

app/assets/stylesheets/custom.scss
```
/* sidebar */
aside {
  section.user_info {
    margin-top: 20px;
  }
  section {
    padding: 10px 0;
    margin-top: 20px;
    &:first-child {
      border: 0;
      padding-top: 0;
    }
    span {
      display: block;
      margin-bottom: 3px;
      line-height: 1;
    }
    h1 {
      font-size: 1.4em;
      text-align: left;
      letter-spacing: -1px;
      margin-bottom: 3px;
      margin-top: 0px;
    }
  }
}

.gravatar {
  float: left;
  margin-right: 10px;
}

.gravatar_edit {
  margin-top: 15px;
}
```

</details>




