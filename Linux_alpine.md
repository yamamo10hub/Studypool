## 概要
軽量さに注目されdockerのベースとしてよく使われるので、調べることにする
  
サポートは２年のようだ。また、メジャーバージョンの区切りはなさそう  
オンプレ視点だと短いが、差し替えられるコンテナなら気にならない  
  
シェルが /bin/ash なので、docker exec でコンテナ内に入るときは注意  
後述するパッケージシステムに bash はあるので、入れてもいい  
`/bin/ashはシバンがpathのみであっさりしてて個人的に使いやすい`  
  
## パッケージ
apkコマンドで管理されている  
telnet curl netstat tcpdump あたりが入っていないので、入れたくなると思われる  
  
```
apk add [package]
apk del [package]
```
  
◆導入候補  
  
```
apk --no-cache add busybox-extras net-tools tcpdump curl bash tzdata gettext
```
* busybox-extras  
　telnetを入れるために使う  
* tzdata  
　デフォルトでtimezoneがUTCしか使えない  
  このパッケージを入れた後、dockerfileで以下コマンドを実行すればJSTに設定可能  
```
　cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime  
```
* gettext  
envsubstというコマンドを使用したい場合、このパッケージを指定する  
  
　　例えばapacheやnginxのhostnameなど固有の情報が必要なコンテナを作る時は、  
　　同梱する***.confファイルが異なるので、DockerImageがhostnameの数だけ作成される  
  
　　***.conf 内の可変部を環境変数で埋めておき、docker run 時に環境変数を指定し、  
　　***.conf の可変部を書き換えると同時にプロセスをフォアグラウンドで起動させる  
　　コンテナを作成することで、単一で汎用性のあるコンテナイメージを作ることができる  
  
```
# envsubst で nginx.conf.template 内の環境変数を書き換えたものを
# nginx.conf として出力しつつ openresty を起動する
CMD envsubst '$$SERVICE_FQDN$$WEBP_SERVER$$ORG_SERVER' < /tmp/nginx.conf.template > 
/usr/local/openresty/nginx/conf/nginx.conf && /usr/local/openresty/bin/openresty -g 'daemon off;'
```
