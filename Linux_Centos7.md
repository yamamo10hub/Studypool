## Centos7

### オンプレでのセットアップ

#### install
インストールは６と相違することも少ない  
HDDの容量が大きいとパーティションが分けられる設定になっているようなので、 / のみに直すなど  

#### Diskについて
centos6と同じくLVMを利用。名前は少し変わっているので、コマンドで確認しておく  
```
df
ls /dev/sd* 
pvdisplay
```
ファイルシステムはExt4からXFSに変更されている  
df -T で表示確認ができる  
  
VMでDiskを増やすことなどがある場合、xfs_growfsで増やすことになす  
LVMであることは変わらないので、fdiskとVolumeへの追加は同じ作業内容となる  
  
#### Hostnameの設定
vim /etc/hostsname  
  

#### Networkの設定<hr>
ifcfg-XXXを直接いじるか、付属のコマンドで設定するか  

nmtui
GUIが出てくるので、必要項目を入力していく  
項目入力後「activate」しないと有効にならない  

直接インターフェースファイルをいじる場合は
```
/etc/sysconfig/network-scripts/ifcfg-eth0
```
"nm_controlled"="no"
を記載しておく必要がある



#### yumのパッケージの更新内容
```
rpm -q --changelog httpd |less
```
CVE-XXXX-XXXX を解決、とか書いてくれてるので、問い合わせ対応しやすい。


#### network確認ツール
net-toolが入っていない(netstatが無い)ので、問題なければ基本的にyumで入れたほうが作業しやすい  
  
標準でssというコマンドが入っている  
ss -antでTCPすべての接続ポートリストがでる  

#### initとsystemd
Centos 6 まではサービス起動管理をinitで行っていた(fedoraとか)  
Centos 7 からはsystemdでサービス起動管理をすることになる(debianとかとおなじ？)  
  
サービス操作コマンド
```
制御  
systemctl (start|stop|restart|status|reload) [サービス名]    
自動起動設定  
systemctl (enable|disable) [サービス名]  
```
  
サービスのログ 
``` 
tailで出し続ける  
jounalctl -f -u [サービス名]   
検索する  
jounalctl --no-pager -u [サービス名] |grep hogefuga  
```


