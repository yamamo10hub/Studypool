### Centosでgoを使う
インストールはしらべたらこんな感じ
```
yum install epel-release
yum install go-lang
```
  
自分はepelのインストール済で、/etc/yum.repos.d/epel.confを
enable=0にしていたのでこうなる
```
yum --enablerepo=epel install golang
```
  
バージョン確認
```
[root@cent7tmp-01 cdnhttp2-nginx]# go version
go version go1.13.3 linux/amd64
```

### goでパッケージを追加
import "gopkg.in/yaml.v2"  
プログラムの頭に記載してパッケージを使う  
  
パッケージはコマンドでインストールしておく必要がある  
go get gopkg.in/yaml.v2  
  
他にもURL指定する方法がありそう?  
