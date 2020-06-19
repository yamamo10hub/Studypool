## javascriptのpackage管理

### npm
node.jsをインストールすると使用可能になる。
globalとlocalで２通りの導入方法があるが、windowsだとローカルではpathが通っていないのか
認識されない  
20191008追記

```
パッケのインストール
npm intall -g typescript

パッケージのリスト
npm ls
npm ls --depth=0

パッケージのアップデート
-gで入れたものは-gをつける
npm update XXXX
npm upgrade XXXX


欲しいパッケージをinstallしたりupdateしようとしたとき、npmが古くて怒られることがある。
npm install -g npm

```

<h4>windowsでのlocal installしたnpm使い方</h4>

一般的にnpmはglobalではなく、localに入れろという主張が多い  
application.jsonにinsatallしたpackageが記載される、というのは正しそうに見える  
windowsでgitbash使ってpath設定し、localのパッケージが使えそうなのでメモしておく  

```  
gitbash
cd ~
vim .bashrc
  export PATH=$PATH:./node_modules/.bin
```
一度閉じて開きなおせばPATHが適用される  
初回のみbash設定変更したことでメッセージがでるが、気にする必要はない  


### yarn
npmとは別のレポジトリ  
yarnを使えるようにするにはnpmでinstallする必要がある  
