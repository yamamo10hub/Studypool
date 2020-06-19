### webについてのtips

[[_TOC_]]

## web全般
---
## hostsファイル
Windowsで特定ホストにアクセスするIPを設定するのに使う  
ここに記載した情報はDNSサーバー問い合わせよりも優先される  
```
C:\Windows\System32\drivers\etc\hosts
```
ブラウザの解析を使いたいが、ローカル環境でDNSもない場合、hostsを設定すると参照可能になる

---
## ブラウザの使い方
chrome, firefox, edgeなどは解析ツールがデフォルトで搭載されるようになった
サイト構成、コンテンツ取得速度、デバッグ などができる

---
## SSL
---
### 単一ライセンスの証明書について
FQDNが同じなので、3ライセンス発行したときに区別がつかない場合がある  
　→　他人がセットアップしていて、ログインできないウェブサーバーなど  
自分で発行した場合は、ファイルが手元にあるのでcrtファイルの中身が違いで判別できる  

証明書の情報のSerialNumberが異なるので、firefoxとかだと確認できる


---
### 証明書の整合性チェック
鍵
```
openssl rsa -text -noout -modulus -in site1.key |grep Modulus
```
証明書
```
openssl x509 -text -noout -modulus -in site1.crt |grep Modulus
```
中間証明書
```
openssl x509 -text -noout -modulus -in site1.crt |grep Modulus
```
csr
```
openssl req -text -noout -modulus -in site1.csr |grep Modulus
```
---
### opensslコマンドによる通信チェック
既に設定されているor設定後にSSL通信できることを確認する
```
openssl s_client -connect www.mysite.info:443  -host [ip_addr]
openssl s_client -connect www.mysite.info:443 -showcerts
```

---
### 証明書の期限を確認する
```
openssl s_client -connect www.mysite.info:443  -host xx.xxx.xxx.xx < /dev/null 2> /dev/null | openssl x509 -text | grep Not
```
IPごとに確認できるので、冗長されていてもOKなやつ  

---
#### Lets encryptでssl証明書を作る

OS CLI  
コマンドの取得と設置
```
# wget https://dl.eff.org/certbot-auto
# mv certbot-auto /usr/local/bin/
# chown root /usr/local/bin/certbot-auto
# chmod 755 /usr/local/bin/certbot-auto
```
certbot-autoで証明書を作成する  
```
/usr/bin/certbot-auto certonly --webroot -w /var/www/html --email 自分のメールアドレス@gmail.com --debug -d www.mysite.info
# 証明書生成に必要なパッケージがない場合はyumが起動するので、インストールを行う
(A)gree/(C)ancel: a
# Let's encryptの利用規約について同意せよとのことなので、Agreeする
(Y)es/(N)o: n
# 今後のlet's encryptの運営チームの情報をメールアドレスに送ってよいか
# Noでも証明書は発行された
IMPORTANT NOTES:
# 以下に生成された証明書ファイルのpath等が記載されているので、よく読む
```
conf.d/ssl.conf
```
    SSLCertificateFile /etc/letsencrypt/live/www.mysite.info/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/www.mysite.info/privkey.pem
```

発行された証明書は３か月で切れるが、自動更新できる仕組みが用意されている
```
# /usr/local/bin/certbot-auto renew --post-hook "service httpd restart"

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/www.mysite.info.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not yet due for renewal

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

The following certs are not due for renewal yet:
  /etc/letsencrypt/live/www.mysite.info/fullchain.pem expires on 2019-08-15 (skipped)
No renewals were attempted.
No hooks were run.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
期限前に更新用のコマンドを実行してみると、更新されずトリガーのapache再起動も発生しないことが確認できた  
30日前であれば更新を実施するらしいので、このコマンドをcronに登録する  

/etc/cron.d/letsencrypt
```
0 16 * * 1 root /usr/local/bin/certbot-auto renew --post-hook "service httpd restart"
```

### 手動で証明書を作る

```
openssl genrsa -des3 2048 > test2020.key
penssl req -new -key test2020.key  -sha256 -out test2020.csr
openssl req -noout -text -in takara2020.csr
openssl rsa -in test2020.key -out n-test2020.key
```


## コマンドクライアントについて
### CURL
---
コマンドでウェブアクセスする  
```
curl -v -H "Host: www.google.co.jp" http://13.231.205.210/hoge.html
```
-v は結果の詳細表示。リクエスト/レスポンスヘッダが表示される  
-H でリクエストヘッダーを付与できる  

```
curl -vk -H "Host: www.google.co.jp" https://13.231.205.210/hoge.html
```
-k httpsでアクセスするときは、証明書チェックを無視するオプションをつける  
  
cookieをつけてリクエストする
```
curl -b 'name=1&arg=value' http://example.com/
curl -H   'Cookie: name=var'  http://example.com
```
cookieはヘッダとしてもつけられるし、cookie付与オプションでもつけられる  
  
最近のブラウザではcURLとしてコピーという右クリックメニューがあるので、  
再現したいときは、それを利用してもよい
```
curl "http://www.mysite.info/header.php" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0" -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" -H "Accept-Language: ja,en-US;q=0.7,en;q=0.3" --compressed -H "DNT: 1" -H "Connection: keep-alive" -H "Cookie: cookie01=value01; cookie02=2019-07-08+02"%"3A22"%"3A27" -H "Upgrade-Insecure-Requests: 1" -H "Pragma: no-cache" -H "Cache-Control: no-cache"
```
上記のようになる  

### wget
curlとちがってデフォルトでファイル名保存となる  
--spiderをつけることで、dryrunできる  
  
centos6でインストールできるversionだと1.12が限界だが、  
このバージョンは証明書のsubjectAltNameに対応していない  
ワイルドカード証明書やマルチドメイン証明書など複数FQDNがある場合、ココに格納される  
  
つまりcentos6のwgetはワイルドカード証明書やマルチドメイン証明書を使用してサイトへ
アクセスするとエラーが表示される。  
  
◆解決案  
a. Centos7にOSを上げる  
　　即時的な解決ではない  
b. wgetをソースからインストールする  
　　時間はかかるし、OSの構成が変わってしまう  
c. wgetで--no-check-certificateをつけて証明書をチェックしない  
　　証明書のチェックはできなくなるが、セキュアな情報でなければあり  
d. curlを使う  
　　wgetと同等のことはオプションでできるはずなので、一番有用か  
  
## TCPDUMP
---
すぐ使い方を忘れるので、メモする  

```
tcpdump -A -s 0 -n -i eth0 src host 221.251.194.101 and port 80
tcpdump -A -s 0 -n -i eth0  port 80
tcpdump -n -i eth0 -XXXX port 81 and host 10.15.1.241
tcpdump -n -i eth0 -X -VVVV  port 81 and host 10.15.1.241
tcpdump -n -i eth0 -Xns 100  port 80
```
```
-A : ASCII文字での表示をする
-X : 16進数とASCII両方で表示する
-s : size指定。defaultは65535で、0にすると全部キャプチャする
-vvv : 詳細を出力する

```

### apache bench
久しぶりにテストしようとしたら忘れていた  
Cent6で実施  
  
◆インストール  
yum install httpd apr-util  
  
実行コマンド  
```
//google1に10クライアントで100回アクセス
ab -c [同接] -n [数]　[URL]
ab -c 10 -n 100 http://www.google.co.jp   
// -H でヘッダが書けるので、Hosts書かなくてもいい
ab -H 'Host: www.targetsite.info' -c 5  -n 50 http://対象IP/
-A user:password でBasic認証も突破できる
```
  
<details>

<summary>結果のサンプル</summary>

```

[root@ymgmt-k-01 src]# ab -c 5 -n 50 http://www.targetsite.info/
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking www.targetsite.info (be patient).....done


Server Software:        nginx/1.12.2
Server Hostname:        www.targetsite.info
Server Port:            80

Document Path:          /
Document Length:        28708 bytes

// 接続結果
Concurrency Level:      5
Time taken for tests:   16.751 seconds
Complete requests:      50
Failed requests:        0
Write errors:           0
Total transferred:      1448150 bytes
HTML transferred:       1435400 bytes
Requests per second:    2.98 [#/sec] (mean)
Time per request:       1675.141 [ms] (mean)
Time per request:       335.028 [ms] (mean, across all concurrent requests)
Transfer rate:          84.42 [Kbytes/sec] received

// 接続秒数
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        4    5   1.1      5      11
Processing:  1223 1359 342.0   1316    3709
Waiting:     1120 1256 337.6   1213    3576
Total:       1228 1364 342.1   1322    3714

// 250リクエスト中2%が3.7秒かかっていることがわかる
Percentage of the requests served within a certain time (ms)
  50%   1322
  66%   1334
  75%   1355
  80%   1374
  90%   1383
  95%   1385
  98%   3714
  99%   3714
 100%   3714 (longest request)
[root@ymgmt-k-01 src]#
```

</details>

## HTTP2 について
調べたことを書いておく  
apache が nginx でプロキシする/ウェブサーバーで有効にする  
  
アップグレードメカニズム  
クライアントが HTTP/2.0 に対応しているかはウェブサーバーからはわからない  
サーバー/クライアント双方がHTTP/2.0に対応可能か知らせる仕組みが必要  
  
```
GET / HTTP/1.1
Host: server.example.com
Connection: Upgrade, HTTP2-Settings
Upgrade: h2c
HTTP2-Settings: <HTTP/2 SETTINGS ペイロードの base64url エンコード>
```
Upgrade: (h2|h2c)
ヘッダにはh2はTLS、h2cは平文
Connection: Upgrade, HTTP2-Settings
これで1.1通信から2.0通信にグレードを上げよ、というリクエストになる、のかな

## HTTPヘッダ

### vary
```
vary: header名
header名は例えば、User-Agent , Accept-Encoding etcetc...
```
そのヘッダの値ごとにコンテンツが変わる場合に付与すべきヘッダ  
代表的なのは例にある二つで、理由は以下となる  
* User-Agent　　　 ：スマホとPCでコンテンツが変わる  
* Accept-Encoding　：コンテンツのgzip圧縮転送に対応/非対応でコンテンツが変わる  
  
varyヘッダが付与されていることで、ブラウザ, googleのbot, CDN等コンテンツを取り扱う要素は  
varyヘッダ別にコンテンツをキャッシュしたり、改めて取得したり等、より配信側の意図に沿った  
挙動をすることができる  

### x-forwarded-for
```
X-Forwarded-For: IPaddress
経由する一つ前のIPが入ることがおおい
```
クライアント　→　プロキシ　→　オリジン  
上記のように途中にプロキシを挟むと、オリジンにはプロキシがリクエストを行うことになる  
大元のリクエスト者を確認できるよう、殆どのプロキシサーバーは自分にリクエストした  
クライアントのIPを上記のHTTPヘッダに入れてリクエストを行う  
  
問題点として、プロキシが多段構成である場合、このヘッダを上書きする存在があるかもしれない  
基本的に「,」区切りで後ろに追加されるが、設定次第では消したり上書きも可能となってしまう  


