## Python入門
progateのpythonのチュートリアルに取り組む

[[_TOC_]]

### 文法<hr>

#### print
print ("hoge")

printは()で囲む
四則演算のスペースはいらない

#### 型変換
hoge = 300
print("ガチャ一回" + str(hoge) + "円")
int(hoge)

#### 何もしない処理について
```
pass  
```
if文のtrue側で何もせず、else時だけ処理を書く場合空白になるが、  
pythonは空白の処理を作るとエラーになるのだと思われる。  
なので何もしない場合は
  


#### if文

```
x = 40
if x != 35:
    print("x is greater than 35!")
elif x == 40:
    print("x is just 40!")
else:
    print("x is unknown")
```
endが無い。
ブロック要素の始まりは : 一つで記載される
ブロック要素の範囲はインデントの数で判断される  
１インデントあたり4spaceとなる  

#### 論理値 and or not

and or で条件を追加できる && || とかと同じ

```
x = 20
if x >= 10 and x <= 30:
    print("xは10以上30以下です")

y = 60
if y < 10 or y > 30:
    print("yは10未満または30より大きいです")

z = 55
if not z == 77:
    print("zは77ではありません")
```

#### 会話式入力

```
apple_price = 200

input_count = input("購入するりんごの個数を入力してください：")

count = int(input_count)
total_price = apple_price * count

print('購入するりんごの個数は' + str(count) + '個です')
print('支払い金額は' + str(total_price) + '円です')
```
  
#### リスト
配列の呼称  
```
foods = ['niku','kome','yasai']
print(foods)
print('健康にいいのは'+ foods[2] +'です')

foods[1] = 'sushi'
foods.append('pan')
```
#### 辞書
ハッシュのことだと思うがなぜ辞書？  
```
fruits = {'apple':'red','banana':'yellow','grape':'purple'}
# indexにキーを指定して呼び出す
print('appleの色は' + fruits['apple'] + 'です')

# 変更
fruits['apple'] = 'green'
print('appleの色は'+ fruits['apple'] +'です')

# 存在しないキーを指定すれば追加になる
fruits['mikan'] = 'yellow'

# すべての要素を順に取り出す
for fruit_color in fruits:
    print(fruit_name + 'は' + fruits[fruit_color] +'色です')
```
  
#### while文
```
x = 10
while x > 0:
  print(x)
  x -= 1
print('end')
```
  
### 制御文

#### for文
```
footds = ['niku','kome','yasai']
for food in foods:
  print('好きな食べ物は' + food + 'です')

```  
  
#### breakとcontinue
```
numbers = [765, 921, 777, 256]
for number in numbers:
    print(number)
    if number == 777:
        print('777が見つかったので処理を終了します')
        break

numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
for number in numbers:
    # 変数numberの値が3の倍数のとき、繰り返し処理をスキップしてください
    if number % 3 == 0:
        continue    
    print(number)
```

### 条件分岐
サンプルで購入プログラムを作成した
  
```
items = {'apple': 100, 'banana': 200, 'orange': 400}
for item_name in items:
    print('--------------------------------------------------')
    print(item_name + 'は1個' + str(items[item_name]) + '円です')
    
    input_count = input('購入する'+ item_name + 'の個数を入力してください：')
    print('購入する' + item_name +'の個数は' + input_count + '個です')
    
    count = int(input_count)
    total_price = items[item_name] * count
    print('支払い金額は' + str(total_price) + '円です')
```

所持金を追加、所持金によって条件分岐を追加  
```
money = 1000
items = {'apple': 100, 'banana': 200, 'orange': 400}
for item_name in items:
    print('--------------------------------------------------')
    print('財布には' + str(money) + '円入っています')
    print(item_name + 'は1個' + str(items[item_name]) + '円です')
    
    input_count = input('購入する' + item_name + 'の個数を入力してください：')
    print('購入する' + item_name + 'の個数は' + input_count + '個です')
    
    count = int(input_count)
    total_price = items[item_name] * count
    print('支払い金額は' + str(total_price) + '円です')
    
    if money >= total_price:
        print(item_name + 'を' + input_count + '個買いました')
        money -= total_price
        # moneyの値が0のとき条件を分岐してください
        if money == 0:
            print('財布が空になりました')
            break
        
    else:
        print('お金が足りません')
        print(item_name + 'を買えませんでした')
print('残金は' + str(money) + '円です')
```

### 関数
サンプル
```
def print_hand(hand, name='ゲスト'):
    print(name + 'は' + hand + 'を出しました')

print('じゃんけんをはじめます')
player_name = input('名前を入力してください：')

if player_name == '':
  print_hand('グー')
else:
  print_hand('グー',player_name)
```

#### 引数や引数のバリデーションを搭載したサンプル
```
def validate(hand):
    if hand < 0 or hand > 2:
        return False
    return True

def print_hand(hand, name='ゲスト'):
    hands = ['グー', 'チョキ', 'パー']
    print(name + 'は' + hands[hand] + 'を出しました')

# 関数judgeを定義してください
def judge(player, computer):
    # playerとcomputerの比較結果によって条件を分岐してください
    if player == computer:
        return '引き分け'
    elif player == 0 and computer == 1:
        return '勝ち'
    elif player == 1 and computer == 2:
        return '勝ち' 
    elif player == 2 and computer == 0:
        return '勝ち'
    else:
        return '負け'

print('じゃんけんをはじめます')
player_name = input('名前を入力してください：')
print('何を出しますか？（0: グー, 1: チョキ, 2: パー）')
player_hand = int(input('数字で入力してください：'))

if validate(player_hand):
    computer_hand = 1
   
    if player_name == '':
        print_hand(player_hand)
    else:
        print_hand(player_hand, player_name)
    
    print_hand(computer_hand, 'コンピューター')
    
    # 変数resultに関数judgeの戻り値を代入してください
    result = judge(player_hand, computer_hand)
    # 変数resultを出力してください
    print('結果は' + result + 'でした')
else:
    print('正しい数値を入力してください')
```

#### モジュールでコードを分ける
utils.pyに関数を移動し、メインのpyファイルで関数を呼び出す場合  
```
# 関数を移動する。
# 頭に移動先のファイル名を拡張子抜きで記載する
import utils

# import先を指す接頭辞を呼び出す関数の頭につける
utils.print_hand
```

#### ライブラリを使う
Pythonであらかじめ用意されているモジュールをライブラリと呼ぶ  
ライブラリはimportで記載してやればすぐに使える  
math, random ,datetime etc...  

```
import utils
# randomモジュールを読み込んでください
import random

print('じゃんけんをはじめます')
player_name = input('名前を入力してください：')
print('何を出しますか？（0: グー, 1: チョキ, 2: パー）')
player_hand = int(input('数字で入力してください：'))

if utils.validate(player_hand):
    # randintを用いて0から2までの数値を取得し、変数computer_handに代入してください
    computer_hand = random.randint(0,2)
    
    if player_name == '':
        utils.print_hand(player_hand)
    else:
        utils.print_hand(player_hand, player_name)

    utils.print_hand(computer_hand, 'コンピューター')
    
    result = utils.judge(player_hand, computer_hand)
    print('結果は' + result + 'でした')
else:
    print('正しい数値を入力してください')
```

utils.py
```
def validate(hand):
    if hand < 0 or hand > 2:
        return False
    return True

def print_hand(hand, name='ゲスト'):
    hands = ['グー', 'チョキ', 'パー']
    print(name + 'は' + hands[hand] + 'を出しました')

def judge(player, computer):
    if player == computer:
        return '引き分け'
    elif player == 0 and computer == 1:
        return '勝ち'
    elif player == 1 and computer == 2:
        return '勝ち'
    elif player == 2 and computer == 0:
        return '勝ち'
    else:
        return '負け'
```

### クラス
クラスの作成
```
class MenuItem:
  pass
```
インスタンス(変数)の作成
```
menu_item1 = MenuItem()
menu_item1.name = 'ホットドッグ'
print(menu_item1.name)
```
クラス内メソッド
```
class MenuItem:
  def info(self):
    return self.name + ': ¥' + str(self.price)

menu_item1 = MenuItem()
menu_item1.name = 'サンドイッチ'
menu_item1.price = 500
print(menu_item1.info())
```
selfは作法。この場合は変数menu_item1自身を指定するための内部処理なのだろう  
jsのthisより見やすい気がする  
  
#### イニシャライズ
\_\_init\_\_ でインスタンス生成時に処理を呼び出せる  

```
class MenuItem:
    # 引数「name」と「price」を受け取るようにしてください
    def __init__(self,name,price):
        self.name = name
        self.price = price

    def info(self):
        return self.name + ': ¥' + str(self.price)

    def get_total_price(self, count):
        total_price = self.price * count
        return total_price

menu_item1 = MenuItem('サンドイッチ', 500)
print(menu_item1.info())

result = menu_item1.get_total_price(4)
print('合計は' + str(result) + '円です')
```
必ず必要な変数割り当てをインスタンス生成時に引数を渡すことでやってくれる  

#### ファイル分割時のクラス利用
* menu_item.pyにクラス記述を移動
* 実行側にfrom [参照先ファイル] import [class名]を記載する
```
from menu_item import MenuItem
```

#### サンプル
menu_item.py
```
class MenuItem:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def info(self):
        return self.name + ': ¥' + str(self.price)

    def get_total_price(self, count):
        total_price = self.price * count
        
        if count >= 3:
            total_price *= 0.9

        return round(total_price)
```
script.py
```
from menu_item import MenuItem

menu_item1 = MenuItem('サンドイッチ', 500)
menu_item2 = MenuItem('チョコケーキ', 400)
menu_item3 = MenuItem('コーヒー', 300)
menu_item4 = MenuItem('オレンジジュース', 200)

menu_items = [menu_item1, menu_item2, menu_item3, menu_item4]

index = 0

for menu_item in menu_items:
    print(str(index) + '. ' + menu_item.info())
    index += 1

print('--------------------')

order = int(input('メニューの番号を入力してください: '))
selected_menu = menu_items[order]
print('選択されたメニュー: ' + selected_menu.name)

count = int(input('個数を入力してください(3つ以上で1割引): '))
result = selected_menu.get_total_price(count)
print('合計は' + str(result) + '円です')
```

### クラスの継承
既に存在するクラスを利用して新しいクラスを作成することを継承という

menu_item.py
```
class MenuItem:
```
menu_item.py上のクラスMenuItemを使用してクラスを継承するには以下のようになる  
  
food.py
```
from menu_item import MenuItem

class Food(MenuItem):
    pass
```
  
script.py
```
from food import Food
from drink import Drink

food1 = Food('サンドイッチ',500)
print(food1.info())
```
特別な指定をしなくてもMenuItemで定義されてるクラスを使用できる  
  
#### クラスメソッドの上書き
親子クラスで同名のメソッドが定義されている場合、このメソッドの定義が優先される  
子クラスで同名のメソッドで定義を書くと上書きしたことになる  

menu_item.py
```
class MenuItem:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def info(self):
        return self.name + ': ¥' + str(self.price)
```
  
food.py
```
class Food(MenuItem):
    def info(self):
        return self.name + ': ¥' + str(self.price) + ' (' + str(self.calorie) + 'kcal)'
```

script.py  
```
from food import Food
from drink import Drink

food1 = Food('サンドイッチ', 500)
food1.calorie = 330
print(food1.info())
```
script.pyで定義しているcalorieはクラス上に存在しない変数でも定義できる  
定義されてないため何にも使用されないが、エラーにはならない  
  
food1.py
```
class Food(self):
    def __init__(self,name,price,calorie):
        super().__init__(name,price)
        self.calorie = calorie
```
上記の様にコンストラクタも上書きで使う変数を増やしておいたほうがよい  
  



