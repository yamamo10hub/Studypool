## openrestyが動く環境を作る
カスタムされたnginx  
安全性やセキュリティフィックスは誰が担保するのか。闇を感じる  
nginxでできないけどopenrestyならできることが結構ある  
  
コンテナで使われることが多いが、あげなおすの面倒なので検証はオンプレでやっていた  
  
## インストール

#### オンプレ
自サーバーがCentos7なので、それを含めてsetup方法をググる
```
# yum utilsが入っていることを確認(普通はいっている)
yum info yum-utils

# openrestyはyumパッケージレポジトリがあるみたいなのでそれを登録する
yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo

# 登録できたらyum installが普通にできるはず
yum install openresty openresty-resty openresty-opm openresty-doc
```

◆重複の削除  
使っているサーバーがすでにepelレポジトリを読み込んでnginxをインストールしていた  
openresty は /usr/local/openresty/nginx/sbin/nginx にnginxを内包しているらしいので、  
先に入れていたnginxを消してopenresty側のnginxをpathで通すことにした  
```
yum remove nginx
cd ~
which nginx
echo 'export PATH=/usr/local/openresty/nginx/sbin:$PATH' >> .bashrc
source .bashrc
which nginx
```

### コンテナを使う
使いやすい  
  
Dockerfile
```
FROM openresty/openresty:alpine

RUN apk --no-cache add net-tools tcpdump curl tzdata && \
    cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

ADD nginx.conf /usr/local/openresty/nginx/conf/nginx.conf

CMD ["/usr/local/openresty/bin/openresty","-g","daemon off;"]
```
  
<details>

<summary>nginx.conf</summary>


```
#user  nobody;
worker_processes  auto;
worker_rlimit_nofile 10000;

error_log  logs/error.log;
pid        logs/nginx.pid;

events {
    worker_connections  65535;
    multi_accept on;
    accept_mutex_delay 100ms;
    use epoll;
}
http {
    server_tokens   off;
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] $host "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log logs/access.log  main;

    client_body_temp_path /var/run/openresty/nginx-client-body;
    proxy_temp_path       /var/run/openresty/nginx-proxy;
    fastcgi_temp_path     /var/run/openresty/nginx-fastcgi;
    uwsgi_temp_path       /var/run/openresty/nginx-uwsgi;
    scgi_temp_path        /var/run/openresty/nginx-scgi;

    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  65;
    reset_timedout_connection  on;
    #gzip  on;

    server {
        client_max_body_size 20M;
        listen       80;
        server_name  s3.amazonaws.com;

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        proxy_redirect          off;
        proxy_cache             off;
        proxy_buffering         off;
        proxy_connect_timeout   600;
        proxy_read_timeout      600;
        proxy_send_timeout      600;

        # proxy
        set $hostheader www.mysite.info;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host   $hostheader;
        proxy_set_header X-Forwarded-Proto https;

        #proxy_hide_header Accept-Encoding;
        location /webp-conv {
            internal;
            resolver 8.8.8.8 8.8.4.4 valid=15s;
            set $hostheader $webp_backend;
            rewrite '^/(.*)$' '/?url=https://s3.amazonaws.com/$1&op=format&fmt=webp' break;
            proxy_pass https://gazou.henkan.com;
        }
        location / {
            # redirect dst setting
            resolver 8.8.8.8 8.8.4.4 valid=15s;
            set $org_backend "www.mysite.info";
            set $webp_backend "gazou-henkan.com";
            # accept base imagefile check
            set $webp_flag 1;
            if ($http_accept !~* image/webp) {
                set $webp_flag 0;
            }
            #if ($webp_flag = 1) {
                default_type text/plain;
                content_by_lua_block {
                    ngx.say("hello, world!")
                    --hoge = ngx.req.get_headers()
                    --outou = ngx.location.capture(\"/webp-conv\")
                    --if outou.res.header["content-type"] == "application/json" then
                    --    ngx.var.webp_flag = 0;
                    --end
                    --ngx.log(hoge)
                }
            #}
            #if ($webp_flag = 1) {
            #   set $hostheader $webp_backend;
            #   rewrite '^/(.*)$' '/?url=https://www.mysite.info/$1&op=format&fmt=webp' break;
            #   proxy_pass https://$webp_backend;
            #}

            #if ($webp_flag = 0) {
            #   proxy_pass http://$org_backend;
            #}
        }
    }
}
```

</details>
  
## openrestyの使い方

#### 基本的な書き方から構造を把握する
nginx.conf内で以下のようなlua文を埋め込む  
「コンテナで使う」のnginx.confを参照  
```
# 始める前にdefault_typeを定義するみたい
default_type text/plain;

#luaの書き始め。contentの部分はopenresty内の使用対象モジュールで異なる
content_by_lua_block {
  -- ngx.printはコンテンツのbodyに埋め込む機能
  ngx.print("hogehoge")

  -- ngx.sayもprintと同じだが末尾で改行される。printと上書きせず追記される
  ngx.say("fugafuga")

  -- logを残したいとき　※例のerrは変数
  -- (出力先, "出力メッセージ", 出力メッセージの後に表示したい変数名)
  ngx.log(ngx.ERR, "error detail :", err)
}
```
  
#### 用途と記述例
ほぼほぼリバースプロキシとしての利用を想定  
  
content_by_luaとlocation.captureの使い方  
```
# location.captureで取得できるのはinternalのみ
location /webp-conv {
    internal;
    resolver 8.8.8.8 8.8.4.4 valid=15s;
    proxy_read_timeout 10;
    set $hostheader "gazou-henkan.com";
    proxy_pass https://$hostheader/$webp_uri;
}
location /conv-fail {
    internal;
    resolver 8.8.8.8 8.8.4.4 valid=15s;
    proxy_read_timeout 10;
    proxy_set_header Host "www.mysite.info";
    set $hostheader "org-www.mysite.info";
    proxy_pass https://$hostheader/$request_uri;
}
location / {
    resolver 8.8.8.8 8.8.4.4 valid=15s;
    set $org_backend "org-www.mysite.info";
    set $webp_backend "gazou-henkan.com";
    set $webp_uri "";
        default_type text/plain;
        content_by_lua_block {
            if string.match(ngx.req.get_headers()["Accept"], "image/webp") then
                outou = ngx.location.capture("/webp-conv" , { vars = { webp_uri = "?url=https://www.mysite.info"..ngx.var.uri.."&op=format&fmt=webp" }});
                if (outou.status == 200) then
                    ngx.header.content_type = "image/webp";
                    ngx.header.content_length = outou.header["Content-Length"];
                    ngx.header.last_modified = outou.header["last-modified"];
                    ngx.say(outou.body)
                else
                    outou2 = ngx.location.capture("/conv-fail" , { vars = { dst_host = ngx.var.http_host }});
                    ngx.header.last_modified = outou2.header["last-modified"];
                    ngx.header.content_length = outou2.header["Content-Length"];
                    ngx.say(outou2.body)
                end
            else
                outou2 = ngx.location.capture("/conv-fail" , { vars = { dst_host = ngx.var.http_host }});
                ngx.header.last_modified = outou2.header["last-modified"];
                ngx.header.content_length = outou2.header["Content-Length"];
                ngx.say(outou2.body)
            end
        }
}
```
  
  
◆書き始めのluablock{}について  
一行程度なら{}で囲わない書き方がある  
```
content_by_lua ngx.log(ngx.ERR, "host header is : ", ngx.header.host);
```
  
たくさん種類があり、やりたいことや、どの時点のnginx処理に割り込ませるかに応じて使い分ける必要があるみたい  
<a href=https://www.nginx.com/resources/wiki/modules/lua/>→ 公式のluaページ</a>  
以下に一部を記載する
```
content_by_lua  
　コンテンツの中身を操作する機能を中心に持つ。１locationに１blockしか書けない  
init_by_lua  
　HUPやconfをreloadされたとき、一度だけ呼び出される  
rewrite_by_lua  
　ngx_http_rewrite_moduleの後に実行される  
access_by_lua  
　IP等によるアクセス制限ができる。タイミングはnginx標準のアクセス制限処理の後となる  
```
各luaでできることは公式のページからリンクするgithubから内容を確認する必要がある  
  
nginxのリクエスト処理フロー  

|順番|フロー名|conf上の処理|lua|
|---|---|---|---|
|start|request||
|1|nginx起動||init_by_lua|
|2|post read|rewrite|||
|3|server rewrite|location|set_by_lua|
|4|location|if/set/rewrite||
|5|rewrite|limit_req/allow/deny/authbasic|rewrite_by_lua|
|6|preaccess / access|autoindex / proxy_pass|access_by_lua|
|7|content handler||content_by_lua|
|8|log|access_log|log_by_lua|
|end|response||



## openrestyの為のlua基礎
openrestyに埋め込むluaはエラーが曖昧で、問題点を探すのが大変なので注意    
  
文末に ; をつける。()で終わる場合はつけなくてもよさそう  
つけなくても大抵動くけど、つけてもエラーは出ない  
ちなみにopenrestyのgithub見ると半分ついてない  
  
### 変数
a = 1
b = "hoge"
c,d  = 5,10
--e = "-- is comment line"
f = "6"
g = "mojiretu wo"..a.."tu ni renketu"
  
### 制御文
判定に使用する内容とか
```
a ~= 1    aが1じゃなければtrue
a == 1    aが1ならtrue
a <= 10   aが10以下
```

__if文__
```
if a == 1 then
    -- openrestyでlogに出力する方法。
    -- ngxはnginxの変数を取り出す方法で、nginx上のログ機能を呼び出している
    ngx.log(ngx.ERR, "a is ", a)
elseif b == "hogefuga" then
    -- luaでの標準出力方法。nginxでは使えない
    print("a = %d" ,a)
else
    -- nginxで終了するときの機能を呼び出してstatus500で終了する
    ngx.exit(500)   
end
```
__for__
```
for i = 1, 10 do
    print( i .. "回目:Hello world!")
end
```


  
while文あるけど省略  
関数もあるけどnginx埋めるだけなら使わなそうなので省略  
  
### 便利なプリセット関数
```
dofile("call.lua")    -- 同ディレクトリにあるcall.luaが起動する
type(b)               -- string 等、型名が返ってくる
tonumber(f)           -- intへ型変換
tostring(c)           -- stringへ型変換
```
  
### table型
変数の一部だが、重要なので例外として最後に書く  

__table(ハッシュ)__
```
変数名 = { key=value }
tbl = { word1=hoge, word2=fuga, word2=piyo }
-- keyを指定しない場合は、数字がkeyになる（後述している配列として扱える）
```
openrestyではheaerなどをluaで処理する為に取り出すと、tableになっている  
```
headers = { Connection=keepalive, Content-Type=image/png, status=200 }
-- こんな感じ
```
tableから値を取り出す
```
for t, val in pairs( members ) do
   for k, v in pairs(val) do
      print( k, v );
   end
end
```

__配列__  
keyのないtableが配列となる
```
local a = { "x", "y", "z" };
  print( a[1] );
  table.sort(a);
```

