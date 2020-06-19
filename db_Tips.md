## すぐ忘れるコマンド等


### 権限関係
```
create database hoge;

# @ 以降に許可するネットワークのレンジを書く
grant all privileges on wp99.* to 'admin99'@'192.168.1.%' identified by 'hogefuga';

# 与えた権限を消したい場合
revoke all, grant option  from  admin99@'192.168.1.%';
# 間違えて与えてしまった database admin21 の権限をadmin20userから削除する場合
revoke all on admin21.* from admin20@'10.61.%';
show grants for admin99;

# 権限を付与する
grant all privileges on wp99.* to 'admin99'@'192.168.%' identified by 'hogefuga';
```

### ユーザー情報
```
select Host, User, Password from mysql.user;
```
