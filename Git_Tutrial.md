[[_TOC_]]

<h2>Git使う</h2>

### Gitセットアップ
web側でrepositoryをつくる。  
作る時にパーミッションを聞かれるので、グループ内公開とする      
ここでレポジトリ名が決定する。(下記例ではvsftp-dock.git)  

```
社内向けgitホスト情報
http://ops-git.mydomain.net
ssh://git@10.1.1.200:10022/yamamo10/vsftp-dock.git
```

localで先に中身を作っていた場合、作業ディレクトリに移動して以下を実行
```
git init
git 
git remote add origin ssh://git@10.1.1.200:1022/yamamo10/vsftp-dock.git
git add .
git status
git commit -m "1st"
git remote add origin ssh://git@10.1.1.200:10022/yamamo10/vsftp-dock.git
git push -u origin master
```
以上でリモートにgitリポジトリの作成が終了する  

### 一度登録したgitの設定を確認する
git config --global -l  
git config --system -l  
  

### 編集
一度作成したremoteリポジトリをlocalの内容で更新する場合の手順  
```
git status
git add .
git commit -m "add readme"
git push -u origin master
```

### remoteからlocalにリポジトリを持ってくる
```
cd  置きたいDir
git clone http://10.1.1.200/yamamo10/vsftp-dock.git
```
git名のディレクトリが作成され、配下にファイルが置かれる

### リモートからコンテンツを引っ張ってくる
```
git clone https://gitlab.com/YYYYYY/XXXXX.git
git clone ssh://user@hostname/path/to/repos

https://gitlab.com/yamamo10/ubu-apache2.git
```

### remoteの更新を反映
remoteからcloneしたが、remoteが更新されたので、localにも適用したい
```
git pull origin master
```

上記を実施したのに、ローカルの変更が残ってしまう場合
```
git fetch origin
git reset --hard origin/master
```

### remoteとlocalの差分を確認したい

```
git fetch origin
git diff origin/master
```
fetchでremoteから最新のリポジトリを取得する  
diffで最新と現状のリポジトリの差分を取ることができる  
ファイルの変更箇所まで表示される  

<h3>remoteとlocalの差分を確認したい２</h3>
```
git remote show origin
```
Out of Dateとでたら、今のリポジトリは古いということ  
fetchよりは詳細がでない

### 複数のリモートリポジトリ
社内向けと公式など複数のremoteリポジトリにpushすることもできる
```
# git remote -v
origin  ssh://git@10.1.1.200:10022/yamamo10/vsftp-dock.git (fetch)
origin  ssh://git@10.1.1.200:10022/yamamo10/vsftp-dock.git (push)
# git remote add pub-back git@gitlab.com:yamamo10/vsftp-dock.git
# git remote -v
origin  ssh://git@10.1.1.200:10022/yamamo10/vsftp-dock.git (fetch)
origin  ssh://git@10.1.1.200:10022/yamamo10/vsftp-dock.git (push)
pub-back        git@gitlab.com:yamamo10/vsftp-dock.git (fetch)
pub-back        git@gitlab.com:yamamo10/vsftp-dock.git (push)
# git push pub-back master
```

新しいremote add先を作り、pushするときに新しい方も実行する。

### 後からgitignoreで除外したい
```
git rm -r --cached .
git add .
git commit -m 
git push origin master
```
キャッシュを消さないとignoreにいれても一度上げたファイルは消えない

### branchをローカルで切る
現在のgitソースに上書きしたくないけど、gitにコードは上げたい
機能の追加が終わってから現在のgitソースに反映したい、等

```
git checkout -b [branch名]
git branch
* [branch名]
  master
```
masterは現在のgitソース。checkout -b することでソース置き場がもう一つ増える。  
これでaddやcommit,pushしてもmasterではない方のソースが更新される。  
  
<h3>ローカルでブランチをmasterへmargeする</h3>
ブランチの中身をmasterへ上書きすることをmargeという  

ブランチをcommitして差分の無い状態にしておく  
masterブランチを選択する(実行するとLocalファイルの内容はmasterになる)
```
git checkout master
```
[ブランチ名] の中身をLocalファイルを融合する(差分上書き)  
pushするとremoteのmasterも変更される  
```
git merge [branch名]
git push origin master
```

### branchをリモートでも切る
実装予定のコードを誰かに見てもらうとき、リモートでもブランチを作成する必要がある  

```
git checkout -b [branch名]
git add -A ./
git commit -m 'new branch init'
git push origin [branch名]
```
これで作成される  
git branch -a でリモートのブランチも確認できる  
  
mergeの方法はlocalと一緒  
  
### branchの削除
ローカル、リモートともにマージが終わったら消すものらしい
```
ローカル
git branch --delete [branch名]
リモート
git push --delete origin [branch名]
```

## トラブルシュート

### windowsとgitの認証管理
VScodeとGitを紐づけていて、cloneやpushにBasic認証がある場合、WindowsOS経由で認証情報が管理されてしまう。  
```
コントロール パネル\ユーザー アカウント\資格情報マネージャー
```
ここで管理されるので、入力をミスしたりパスワードを変えた場合、編集する必要がある  





