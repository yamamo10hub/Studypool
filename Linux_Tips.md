[[_TOC_]]

## Linuxでよく使ったコマンドなど
---
### HDD系
* VMで仮想HDDの容量が増やせるが、OS側で拡張分を利用可能にするには手順が必要

fdisk  
d,n,p,Enter,Enter,t,8e,w などするとパーティションを現在のDiskの最大値に拡張できる  
最後のwを押すまで実行されないので多少気楽。また、反映にはrebootが必要  
XenserverのVMなどで使っていたが、クラウドでやるとうまくいかないことがある  
  
parted  
使用中のディスクには使えないので割愛  
  
growpart  
クラウドでVMのディスク拡張手段としてよく紹介される
```
lsblk                 # 現状と実行対象確認
growpart /dev/sda 1   # 実行
reboot                # 反映
```

df -T  
ファイルタイプがわかる

---
### ファイル検索

ファイル名でさがす
```
find PATH -name hogelog*.gz
```

ファイルのパーミッションでさがす
```
700でownerしかアクセスできないディレクトリをさがす
find /etc/ -perm 700 -type d

実行属性のついたファイルをさがす
find /etc/httpd/ -executable -type f
```

権限の変更
```
指定したpathのディレクトリ以下にあるディレクトリだけに実行属性をつける
chmod -R go+rX /etc/
```
  
ディレクトリ内を探して何かする
```
find ./ -type f -maxdepth 1 | xargs -i mv {} {}_YYYYMMDD
```
カレントディレクトリのファイルのみを探して、ヒットしたものに大してmvで名前を変更している  
  

### grepで表示操作
ファイルの中身をさがす、フィルターする  
```
grep -v "word"  
wordは除外する
grep -e "word" "kotoba"
wordとkotobaに該当するものを表示する
grep -E "^(w|W)ord$"
拡張正規表現を有効にされて先頭が大文字または小文字で末尾がdで終わるwordを表示する
grep -w "/"
指定した単語に完全一致する行がある場合、その行を表示する
```

### awkで表示操作

```
cat hoge.txt | awk '{print $1}'
テキストでスペース区切りの一列目だけを表示

cat hoge.txt |grep  -e " 200 " -e " 304 " |awk '{if(match($7, /html$/))print $7}' |wc -l
apachecombinedログでステータスが200or304で７要素目がhtmlで終わるログを表示し、該当行数を表示する

cat hoge.txt |grep -v -e "iPhone" -e "iPad" -e "iPod" |awk '{SUM += $10}END{print SUM}'
apacheのcombineログでApple端末以外のファイルサイズを加算計測して、結果だけを表示する
```

### プロセス毎のファイル操作数上限

ulimitという値で制限がされている  
OSごとに制限値が異なる（centos5 -> 無限, centos65 -> 1024）
よく使ったcentos6に乗せていたミドルウェアで影響が出る場合が多かった  
(hadoop, glusterfs, lsyncd etc... では1024を超えることがあった)  
  
ulimitの変更には厄介な部分がある
* 実行ユーザーでパラメータがことなる  
rootで実行するとき、特定ユーザーで実行するとき、などそれぞれ値が違う  
コンソールで`ulimit -n 8192`としてもログアウトしたら1024に戻る  
自動起動では適用されない、など  

ulimitの確認方法
```
ulimit -Sn
ulimit -Hn
ps -aef |grep xinetd
# プロセス番号が欲しい
cat /proc/プロセス番号/limits
# そのプロセスのulimit設定が表示される
```

yumでインストールしたinitによるサービスのプロセスでは適用がむつかしい
/etc/init.d/httpd に直でulimit -n 8192で追加できるが、メンテナンス性が悪すぎる  
そこでchkconfigから外して、rc.localで起動するよう設定した

/etc/rc.local  
```
# start xietd with ulmit
ulimit -n 8192
/etc/init.d/xinetd start

```

## パフォーマンス計測

htop  
見やすいtop 検索とかしやすい  
```
yum install htop
```
  
vmstat  
vmstat 1 で１秒ごとのリソースが表示される  
トラブル中にコンソールで確認するときに使うか    
  
sar  
一ヵ月分のサーバーリソースが /var/log/sa にあったりする  
Centos7からは標準  


