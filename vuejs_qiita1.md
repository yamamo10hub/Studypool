## vuejsでSPA作成を体験する

### node.jsをセットアップする

そのままインストールせずにバージョン管理ツールを使う  
Windows だと nodist というのが主流らしいので、インストーラー落として入れておく  
  
コマンドラインでnodeを導入していく  
```
# installを確認
nodist -v
# 使えるversionの一覧
nodist dist
# 表示したversionの最新を指定して導入
nodist + 12.3.1
# 導入したnodeから使用するversion一つを指定する
nodist 12.3.1
# 指定が適用されて使えるか確認する
node -v
```

npmは勝手に入っている
```
npm --version
npm update -g npm
```


### セットアップ

```
npm install -g yarn
yarn -v
```

### 構築

```
yarn add -D vue-cli
./node_modules/vue-cli/bin/vue init webpack-simple
# Use sass?へのyes以外はEnterですすめる
yarn
# webpack-simpleで使用するパッケージをyarnでinstallする
yarn add -D browser-sync
```

vim package.json
```
# scriptsの一番上に一行追加を行う
"scripts": {
    "watch:server": "browser-sync init", //追加
    "dev": "cross-env NODE_ENV=development webpack-dev-server --open --hot",
    "build": "cross-env NODE_ENV=production webpack --progress --hide-modules"
}
```

変更を適用する  
$ yarn watch:server  
  
サーバーの設定？を変更  
vim bs-config.js
```
    //"server": false, //元の記載を排除
    "server": {
        "baseDir": "./dist",
        "middleware": function(req, res, next){
            var timestamp = "[" + new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '') + "] ";
            console.log(timestamp + req.method + " " + req.originalUrl + " - " +  req.connection.remoteAddress + " - " + req.headers['user-agent']);
            next();
        }
    },
```

npmを複数使用できるようにするパッケージのinstall  
$ yarn add -D npm-run-all  

vim package.json
```
# before
    "watch:server": "browser-sync init",
# after
    "watch:server": "browser-sync start --config bs-config.js",  // 変更
    "watch:js": "webpack -w",  // 追加
    "start": "run-p watch:*",  // 追加
```

webserverを起動させる  
$ yarn start  
  
起動によって生成されたindex.htmlを移動する  
mv index.html src/  
  
vim src/index.html
```
<script src="/dist/bundle.js"></script> <!-- 変更前 -->
<script src="/js/bundle.js"></script>
```

ファイルをコピーできるpackageをinstallする  
$ yarn add -D cpx  

src/ のhtml, 画像, font をdist/へコピーするコマンドを追加  
vim package.json
```
# watch:server と watch:js の間に一行追加する
"watch:file": "cpx \"./src/**/{*.html,*.jpg,*.png,*.gif,*.svg,*.eot,*.ttf,*.woff,package.json}\" ./dist",
```

vim /src/App.vue
```
<img src="./assets/logo.png">
↓
<img src="./img/logo.png">
```


vim webpack.config.js
```
output: のpablicPathを /dist を / に変更する
# module: rules: 以下の画像指定部部に追記
name: '[name].[ext]?[hash]'
name: 'img/[name].[ext]?[hash]'

# 画像の下に
      { // フォントファイルを使用する場合は以下を追記
        test: /\.(eot|ttf|woff)$/,
        loader: 'file-loader',
        options: {
          name: 'fonts/[name].[ext]?[hash]'
        }
```






