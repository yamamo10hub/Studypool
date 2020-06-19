### goについて
---
googleが作った言語  
誰が作っても同じコードになるとか聞いた、すごい  
  
コンパイルしてバイナリでも、スクリプトでも動くらしい、すごい  
ゆるきゃらがいる  
  
### Centos7に入れる
---

#### パッケージでインストール 
```
yum install epel-release
yum install golang
go version
```

#### GOPATHの設定
環境変数が必要らしい  
パッケージとかは指定したpathに入る  
  
と思ったけどgo1.8からは必要ないらしい。  
  
  
vim ~.bash_profile  
```
# Before
PATH=$PATH:$HOME/.local/bin:$HOME/bin
export PATH
# After
GOPATH=/usr/local/go
PATH=$GOPATH/bin:$PATH:$HOME/.local/bin:$HOME/bin
export PATH
export GOPATH
```
source .bash_profile

### ubuntu16.04に入れる

<a href=https://golang.org/dl/>goの公式のダウンロードページ<a> でバージョンを確認して  
ダウンロードする  
```
curl https://dl.google.com/go/go1.13.5.linux-amd64.tar.gz
```
解凍して/usr/local/以下に展開する  
```
tar -C /usr/local -xzf go1.13.5.linux-amd64.tar.gz
```
pathをはってshell再読み込みしてバージョンを表示し、正常性の確認
```
export PATH=$PATH:/usr/local/go/bin
exec $SHELL -l
go version
```
  
### Windows10に入れる
* https://golang.org/dl/ でwindows向けmsiをダウンロード  
* gitが無ければ入れる(このwikiではvscodeのセットアップ記事で導入済み)  
* 公式ドキュメントが和訳されてる  
https://gin-gonic.com/ja/docs/quickstart/  
  

