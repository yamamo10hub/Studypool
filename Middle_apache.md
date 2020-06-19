## apache

パッケージベースでのインストールがいいのではないか  
パッケージを探す：`yum search mod ssl httpd`  
少なくとも導入する際にsourceかpackageかは決めておくべき  

## OSごとの違い

### CentOS 5

### CentOS 6
apache 2.2 と 2.4 がパッケージに存在する

◆versionでの差異
versionでyumのパッケージ名とコンフィグのファイル配置や設定が違う  

#### 2.2  
インストール
```
yum install apache mod_ssl
```

設定ファイル
/etc/httpd/conf.d/ssl.conf
```
<VirtualHost 10.0.1.101:443>
    ServerAdmin root@localhost
    DocumentRoot /home/www/site2/

    <Directory "/home/www/site2/">
        AllowOverride All
        Options Includes FollowSymLinks
        Order allow,deny
        Allow from all
    </Directory>
    ServerName yamamo10-test.mysite.com
    ErrorLog  "|/usr/sbin/rotatelogs /var/log/httpd/ssl2-error_log.%Y%m%d 86400 540"
    CustomLog "|/usr/sbin/rotatelogs /var/log/httpd/ssl2-access_log.%Y%m%d 86400 540" combined

    ErrorDocument 404 /404.html

SSLEngine on
SSLProtocol all -SSLv2 -SSLv3
SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLCertificateFile /etc/httpd/ssl/hoge2018.crt
SSLCertificateKeyFile /etc/httpd/ssl/hoge2018np.key
SSLCertificateChainFile /etc/httpd/ssl/hoge2018-chain.crt

SetEnvIf User-Agent ".*MSIE.*" \
         nokeepalive ssl-unclean-shutdown \
         downgrade-1.0 force-response-1.0

</VirtualHost>
```


#### 2.4
インストール
```
yum install apache24 mod24_ssl
```

設定ファイル  
vim /etc/httpd/conf/httpd.conf
```
#AddDefaultCharset UTF-8
ServerTokens Prod
ServerSignature Off
```
下二つは明記されなくなったらしい。  
書かないとバージョンが見える  


/etc/httpd/conf.d  
notrace.conf
```
HTTP trace メソッドの有効/無効
デフォルトで無効
一部ブラウザとの組み合わせで脆弱性を持っているとか
別に使わないので、強い要望がなければ、有効にする必要はないと思う
```
mv welcome.conf welcome.conf.org
```
名前変えるとトップページのwelcomeが表示されなくなる
```
userdir.conf  
```
ユーザーディレクトリ機能の設定ファイル。デフォルトoffで無害
```
autoindex.conf  
```
indexoption関連の設定ファイル。いじらなくても機能する
アイコンとか/ アクセス時の挙動の設定とか
```
ssl.conf
```
SSLHonorCipherOrder On
# クライアントが受けれるCipherを優先するか、サーバー側で指定されたCipherを優先するか
# Onだとサーバー側のCipherを優先する。そうするべき

# ほかは2.2と変わらない様子
```
VirtualHostを書く場合  
httpd.confの末尾に追記  
IPでアクセスされて困る場合は、先頭にダミーのvirtualhostを書きましょう
```
<VirtualHost *:80>
    ServerAdmin root@localhost
    DocumentRoot /home/www/gluster/honban/
    ServerName yamamo-to.mysite.info
    #ServerAlias dev-to.mysite.info
    ErrorLog /var/log/httpd/yamamo-to.error
    LogLevel warn
    CustomLog /var/log/httpd/yamamo-to.access combined env=!nolog
    <Directory "/home/www/gluster/honban">
        Options ExecCGI Indexes
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
ディレクティブのIP制限の書き方が変わった
ホワイトリスト方式
```
Require all granted
Require not ip 192.168.10.100
```
ブラックリスト方式
```
Require all denied
Require ip 192.168.10.100
```

### CentOS7

### 2.2と2.4の違い
allow,denyの書き方が変わる

ログの出力とrotate


## apacheで雑にcookieをつける
cookie付与時の挙動を確認するために何でもいいからcookieをつけたかった

httpd.conf
```
# servernameの下あたりに記載
CookieTracking on
CookieExpires "1 months"
```
set-cookieが付与される

