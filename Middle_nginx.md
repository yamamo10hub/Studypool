[[_TOC_]]


## nginxについて
◆プロセスの特徴  
プロセスを常時ループでぐるぐるまわすのではなく普段待機しておき、特定のトリガーに対応した機能を動作させる  
イベント駆動型で、apacheなどより負荷が少なく、配信のパフォーマンスが良いらしい  

イベントには「クライアントからの接続要求」「通信機能上の読み込み許可」「通信機能上の書き込み許可」等がある  
通信機能（モジュール）をファイルディスクリプタと呼ぶ  
  
◆プロセス管理について  
マスタープロセスとワーカープロセスがある  
実際の処理はワーカープロセスが行う。ワーカーは複数起動することができる  
マスタープロセスはワーカーの管理を行う。一つしか起動できない  
OSからの命令はマスターに送る必要がある  
  
## install
公式がコンテナを出しており、しかも使いやすくウェブ配信と相性もいい  
  
生でインストールする場合の内容を一応調べておく  
必要なライブラリはPCRE, zlib, OpenSSL  
sourceから導入する場合は先にこれらを入れておかないとmake installしたときに機能が足りなかったりする  

./configure のオプションについて  
```
./configure --prefix=/usr/local/nginx --sbin-path=/usr/sbin/nginx
```
  
多くの機能を持つが、コマンドで指定しないと使用できない機能がある  
  
|モジュール名|解説|指定コマンド|
|---|---|---|
|ngx_http_ssl_module|SSLサポート|--with-http_ssl_module|
|ngx_http_v2_module|HTTP/2のサポート|--with-http_spdy_module|
|ngx_http_realip_module|リクエスト元IPアドレスを利用|--with-http_spdy_module|
|ngx_http_addition_module|HTTP/2のサポート|--with-http_addition_module|
|ngx_http_geoip_module|HTTP/2のサポート|--with-http_geoip_module|
|ngx_http_sub_module|HTTP/2のサポート|--with-http_sub_module|
  
◆実行プロセス管理  
configのテストコマンド  
```
nginx -t 
```
  
nginx -s (signal)  
```  
stop   TERM, INT
quit   QUIT
reload HUP

```
  
## config設定
log_format main [log項目]
```
time: $time_local           # 時刻
status: $status             # ステータス
request_time: $request_time # リクエスト時間
remote_addr: $remote_addr   # リモートアドレス
request_method: $request_method # メソッド
request_url:$request_url    # リクエストURL
protocol:$server_protocol   # プロトコル
http_referer:$http_referer  # リファラ
http_user_agent:$http_user_agent; # ユーザーエージェント
```
  
## リバースプロキシ

バックエンドにアップロード機能があったりする場合、エラーがでることがある
```
413 Request Entity Too Large
```

デフォルトでは1MBまでなので、それを超えるコンテンツサイズだと上記のエラーがでる  
対策としてnginx.confに追記を行う必要がある  
```
client_max_body_size 10M;
```

とりあえずリバースプロキシするには設定  
```
worker_processes  auto;
# worker_processがnginxのプロセス数。
# autoだとnginxが勝手に決める

worker_rlimit_nofile 100000;
# openfile数

worker_cpu_affinity 1000 0100 0010 0001;
# cpuコア数の指定。上記だと4コア

events {
    worker_connections  65535;
}
# worker_process数と乗算することで同時クライアント数となる

http {
    include       mime.types;
    # 拡張子とMIMEタイプの割り当て
    default_type  application/octet-stream;
    # 上記割り当てにないもののタイプ指定
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    # 出力ログのフォーマット
    access_log /var/log/nginx/access.log  main;
    error_log  /var/log/nginx/error.log info;
    sendfile        on;
    #クライアントへのレスポンス処理sendfileシステムコールというカーネルモジュールをつかうか
    # 評判がよかったりよくなかったり
    #tcp_nopush     on;
    # sendfileが有効な時に機能する、TCP_CORKソケットオプションを使うかどうか
    keepalive_timeout  65;
    # サーバー側でのTCPタイムアウト時間。default75sec
    gzip  on;
　　# gzipによるコンテンツ圧縮
    # ほかにいろいろオプションがつくはず
    server_tokens   off;
    # nginxのversionを表記するか
    server {
        listen       80 default_server;
        server_name  _;
    }
    server {
        client_max_body_size 20M;
        listen       80;
        server_name  www.origin-server.com;

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        proxy_redirect          off;
        # onにするとproxy先がredirectした場合hostヘッダをserver_nameに書き換える
        proxy_cache             off;
        # キャッシュ機能の 有効/無効
        proxy_buffering         off;
        #

        #proxy_connect_timeout   600;
        #proxy_read_timeout      600;
        #proxy_send_timeout      600;

        proxy_set_header Host   $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Real-IP $remote_addr;
        # 送信元のIPをこのヘッダー名でhttp通信に乗せる

        #location ~ .*\.(htm|html|jpg|JPG|gif|GIF|png|PNG|swf|SWF|css|CSS|js|JS|inc|INC|ico|ICO) {
        #   root    /opt/nginx/html;
        #   index   index.html;
        #   break;
        #}

        location / {
            #sub_filter_once off;
            #sub_filter_types text/html;
            
            proxy_pass http://www.origin-server.com/;
        }
    }
}

```

## nginxでfastcgiでPHPを動かす

◆phpとphp-fpmをインストールしておく  
◆php-fpmの設定  
/etc/php-fpm.d/www.conf
```
;defaultのユーザーグループがapacheになってるのでnginxに変更
;user = apache
user = nginx
;group = apache
group = nginx
```
systemctl restart php-fpm
  
◆nginxの大元の設定(nginx.conf)は特に触るところなし。  
◆nginxのサイト側の設定  
```  
conf.d/site01.conf
server {
  listen 80;
  server_name dev-wp52.mysite.info;
  index index.php;
  root /home/efs/wp51;
  #root /home/www/wp51;
  access_log  /var/log/nginx/wp51.access.log  main;
  error_log  /var/log/nginx/wp51.error.log  warn;
  location ~* /wp-config.php {
    deny all;
  }
  location ~ \.php$ {
    root /home/efs/wp51;
    #root /home/www/wp51;
    fastcgi_pass unix:/run/php-fpm/www.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
}
```
/sbin/nginx -t  
systemctl reload nginx  
  
### fastcgiでキャッシュもする

◆nginx.conf
```
#httpディレクティブに記載する
#使うVirtualhostの数だけpathは必要。keys_zoneの名前をユニークにする
fastcgi_cache_path /var/cache/nginx/wp51 levels=1:2 keys_zone=wp99:30m max_size=256M inactive=600m;
#cache_keyはキャッシュオブジェクトの命名規則なので、一個書けばよい
fastcgi_cache_key "$scheme$request_method$host$request_uri";
# cache 404 or other
fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
# return old cache if PHP crashes
fastcgi_cache_use_stale error timeout invalid_header http_500;
```
fastcgi_cache_path /var/cache/nginx/wp51  
　キャッシュの保存先。バーチャルホスト毎に異なるパスを指定  
levels=1:2  
　キャッシュの構造レベルを指定  
keys_zone=testdom:30m  
　任意のキャッシュ名をつける。バーチャルホスト毎にユニークにする  
　キーのメモリサイズが30MBを保持するように設定  
max_size=512M  
　キャッシュの最大サイズ。超過した場合、利用されていないものから削除される。  
inactive=600m  
　キャッシュのTTL  
  
◆conf.d/site51.conf

<details>
<summary>古いsite51.conf</summary>

一応うごいたはず
```
server {
  listen 80;
  server_name dev-wp51.mysite.info;
  index index.php;
  root /home/efs/wp51;
  access_log  /var/log/nginx/wp51.access.log  main;
  error_log  /var/log/nginx/wp51.error.log  warn;
  location / {
    index index.html index.htm index.php;
    try_files $uri $uri/ /index.php?$args;
  }
  location ~ \.php$ {
    fastcgi_pass unix:/run/php-fpm/www.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    # phpの中でキャッシュしたくないものにフラグをつける
    set $do_not_cache 0;
    # phpの中でキャッシュしたくないものにフラグをつける
    if ($request_method = POST) {
      set $do_not_cache 1;
    }
    if ($query_string != "") {
      set $do_not_cache 1;
    }
    if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in") {
      set $do_not_cache 1;
    }
    if ($request_uri ~* "(/wp-admin/|/xmlrpc.php|/wp-(app|cron|login|register|mail).php|wp-.*.php|/feed/|index.php|wp-comments-popup.php|wp-links-opml.php|wp-locations.php|sitemap(_index)?.xml|[a-z0-9_-]+-sitemap([0-9]+)?.xml)") {
      set $do_not_cache 1;
    }
    # nginx.confで作ったkeys_zoneの名前
    fastcgi_cache wp51;
    fastcgi_cache_valid 200 3m;
    fastcgi_no_cache $do_not_cache;
    fastcgi_cache_bypass $do_not_cache;
    # キャッシュの状態を表示できるヘッダをいれる
    # 必須じゃないが、検証中だけでも入れるべき
    add_header X-F-Cache $upstream_cache_status;
    #fastcgi_cache_key "$scheme://$host$request_uri";
  }
```

</details>

site51.conf
```
server {
  listen 80;
  server_name dev-dms-wp51.mysite.info;
  index index.php;
  root /home/efs/wp51;
  access_log  /var/log/nginx/wp51.access.log  main;
  error_log  /var/log/nginx/wp51.error.log  warn;
  # 後述するコンフィグファイルを読み込んでいる。中身はリクエストに応じたmax-age付与設定
  include wordpress.conf;
  # wordpressのphpファイルの中でキャッシュすべきでないもがあるのでifで選別している
  location ~ \.php$ {
    # phpを処理するためにphp-fpmのsocketファイルを指定
    fastcgi_pass unix:/run/php-fpm/www.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    try_files $uri =404;
    # キャッシュ処理するかバイパスするか判定するフラグ
    # 0=Off 1=On
    set $do_not_cache 0;
    if ($request_method = POST) {
      set $do_not_cache 1;
    }
    if ($query_string != "") {
      set $do_not_cache 1;
    }
    if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in") {
      set $do_not_cache 1;
    }
    if ($request_uri ~* "(/wp-admin/|/xmlrpc.php|/wp-(app|cron|login|register|mail).php|wp-.*.php|/feed/|index.php|wp-comments-popup.php|wp-links-opml.php|wp-locations.php|sitemap(_index)?.xml|[a-z0-9_-]+-sitemap([0-9]+)?.xml)") {
      set $do_not_cache 1;
    }
    # nginx.confで作成したキャッシュ領域の名前。Vhost毎に変えないと
    fastcgi_cache wp51;
    # status 200 であれば3分キャッシュする
    fastcgi_cache_valid 200 3m;
    # ifに該当してdo_not_cacheが1なら有効なので、キャッシュしないし、バイパスする
    fastcgi_no_cache $do_not_cache;
    fastcgi_cache_bypass $do_not_cache;
    # レスポンスヘッダにキャッシュの動作が表示できるので、検証に役立つ
    add_header X-F-Cache $upstream_cache_status;
    #fastcgi_cache_key "$scheme://$host$request_uri";
  }
}
```

wordpress.conf(/etc/nginx/wordpress.conf)
```
location / {
  index index.html index.htm index.php;
  try_files $uri $uri/ /index.php?$args;
}
# staticになりえる拡張子はmax-ageつけてキャッシュしてもらう
location ~ \.(jpg|png|gif|swf|jpeg)$ {
  log_not_found off; # 404の時にerror_logに書き込まないようにする設定
  access_log off;
  expires 3m;
}
location ~ \.ico$ {
  log_not_found off;
  access_log off;
  expires 3m;
}
location ~ \.(css|js)$ {
  charset  UTF-8;
  access_log off;
  expires 3m;
}
# htaccessとかあるのでブロック
location ~ /\. {
  deny all;
  log_not_found off;
  access_log off;
}
# 設定書かれているのでブロック
location ~* /wp-config.php {
  deny all;
}
```
1Vhost1ファイルなので、wordpress.confの内容を何度も書かなくていいように分ける
サイトが追加される場合、nginx.confとwpXX.confが変更される。

### 新しいヘッダを足したい
既に同じヘッダがある場合、二重に追加されるので注意
```
add_header 'Content-Type' 'application/json';
```

### subfilter
コンテンツへの干渉ができる

