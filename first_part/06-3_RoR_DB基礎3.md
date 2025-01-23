# 6.1 Ruby on Rails：データベース基礎

## 目次

  - [ActiveRecordでの効率的なデータ取得](#activerecordでの効率的なデータ取得)
    - [(1) 「N+1問題」とは](#1-n1問題とは)
    - [(2) 「N+1問題」解消方法](#2-n1問題解消方法)
    - [(3) サブクエリの作り方](#3-サブクエリの作り方)
    - [(4) 練習](#4-練習)

## ActiveRecordでの効率的なデータ取得

ここではActiveRecordの応用的な利用方法「N+1問題について」「サブクエリの作り方」について学習します。

### (1) 「N+1問題」とは

一覧画面などに表示するデータを取得するとき、SQLのクエリが「データ量(N) + 1」発行され、データ量が多くなるにつれてパフォーマンスを低下させてしまう問題です。  
1つのクエリでデータを取得する際に、その取得したデータに関連する別のデータを取得するために、複数の追加のクエリが発生することを指します。  
例えばデータの数だけSELECT文を発行すると1回が1msでも、データの数が増えるとその数だけSELECT文を発行することになり、パフォーマンスが悪くなっていきます。

では、N+1問題について具体的なコードを見ながら考えてみましょう。

Modelは `User` と `Favorite` のものがあったとします。

`User: ユーザーテーブル`

| field名 | 名称 | 型 |
|---|---|---|
| id | ID | integer |
| name | 名前 | string |

`Favorite: お気に入りテーブル`

| field名 | 名称 | 型 |
|---|---|---|
| id | ID | integer |
| user_id | ユーザーID | integer |
| title | タイトル | string |

ユーザー(User)はたくさんのお気に入り(Favorite)を登録することができるという「1 対 多」の関連付をします。  
アプリを作成しましょう。  

```sh
$ rails new favorite_app
$ cd favorite_app
$ rails g model User name:string
$ rails g model Favorite user:references title:string
$ rails db:migrate
$ rails g controller Favorites index
```

```rb
# app/models/user.rb

class User < ApplicationRecord
  has_many :favorites
end
```

```rb
# app/models/favorite.rb

class Favorite < ApplicationRecord
  belongs_to :user
end
```

Controllerでは `Favorite` の一覧を取得するよう以下の内容になっているとします。

```rb
# app/controllers/favorites_controller.rb

class FavoritesController < ApplicationController
  def index
    @favorites = Favorite.order(:id)
  end
end

```

`View` では取得したお気に入りに関連する `user` の名前を表示するとします。

```html
<!-- app/views/favorites/index.html.erb -->

<h1>お気に入り一覧</h1>
  <% @favorites.each do |fav| %>
    <div><%= fav.user.name %></div>
  <% end %>
```

「お気に入り一覧を表示する」という機能的には問題なく満たしていますが、レコードを取得する際のデータベースへのアクセスに問題があります。

ログを確認してみましょう。  

今回はseedを使ってデータを作成します。  
初期データやテストデータを作成するときに使われます。  

```rb
# db/seeds.rb

# 1つのユーザーを生成
user = User.create(name: "test")

# お気に入りを生成
100.times do |n|
  user.favorites.create(title: "お気に入り #{n + 1}")
end
```

seeds.rbに記述した内容をDBに反映させます。

```sh
$ rails db:seed
```

```sh
$ rails s
```

**ログ**

`users`テーブルへの `SELECT` が `favorite` の数だけ行われています。
これではレコードが増えれば増えるほどSQLのクエリが発行されてしまいパフォーマンスが非常に悪くなってしまいます。
例えば1回のクエリにかかる時間が0.1ms前後だったとして、取得するレコードが何千何万とあった場合それだけ時間がかかってしまうということになります。これが「N+1問題」の実態です。

```sh
Favorite Load (0.5ms)  SELECT "favorites".* FROM "favorites" ORDER BY "favorites"."id" ASC
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  ・
  ・
  ・
  ```

### (2) 「N+1問題」解消方法

解決方法としては以下の4つのメソッドを使用するのが一般的です。<br>
メソッドは組み合わせて使うこともできます。

#### (a) includesを使用する

引数には、Association先を指定します。

```rb
def index
  @favorites = Favorite.order(:id).includes(:user)
end
```

するとクエリの実行は2回で済みます。
1回目でお気に入りを全件取得し、2回目で `user_id` を指定して紐づく `user` を取得しています。

```sh
Favorite Load (1.0ms)  SELECT "favorites".* FROM "favorites" ORDER BY "favorites"."id" ASC
User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? [["id", 1]]
```

#### (b) preloadを使用する

```rb
def index
  @favorites = Favorite.preload(:user).order(:id)
end
```

```sh
Favorite Load (11.2ms)  SELECT "favorites".* FROM "favorites" ORDER BY "favorites"."id" ASC
User Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? [["id", 1]]
```

#### (c) eager_loadを使用する

```rb
def index
  @favorites = Favorite.eager_load(:user).order(:id)
end
```

引数で指定したAssociationを `LEFT OUTER JOIN`(左外部結合)します。
1回のクエリで済みます。

```sql
SELECT
    "favorites"."id" AS t0_r0, 
    "favorites"."user_id" AS t0_r1, 
    "favorites"."title" AS t0_r2, 
    "favorites"."created_at" AS t0_r3, 
    "favorites"."updated_at" AS t0_r4, 
    "users"."id" AS t1_r0, 
    "users"."name" AS t1_r1, 
    "users"."created_at" AS t1_r2, 
    "users"."updated_at" AS t1_r3
  FROM 
    "favorites" 
    LEFT OUTER JOIN "users" 
      ON "users"."id" = "favorites"."user_id" 
 ORDER BY 
    "favorites"."id" ASC
```

#### (d) joinsを使用する

joinsは、指定したAssociationを `INNER JOIN` します。
これだけでは「N+1問題」の解消はできませんが、結合先のテーブルに対しての絞り込みが可能です。

```rb
def index
  @favorites = Favorite.joins(:user).where(users: {id: 1})
end
```

```sh
Favorite Load (0.2ms)  SELECT "favorites".* FROM "favorites" INNER JOIN "users" ON "users"."id" = "favorites"."user_id" WHERE "users"."id" = ?  [["id", 1]]
```

### (3) サブクエリの作り方

サブクエリとは、SQLクエリの中に含まれるクエリです。つまり、クエリの中で別のクエリが実行され、その結果が親クエリの条件や操作に利用されます。  

サブクエリは以下のような場面で利用されます。

1. 条件の絞り込み：サブクエリを使って特定の条件を満たす行のみを取得します。
2. 集計：サブクエリを使って集計関数を利用し、集計結果を取得します。
3. 結合条件：サブクエリを使って別のテーブルからデータを取得し、それを元に結合条件を指定します。

```sql
SELECT name 
FROM users 
WHERE id IN (
    SELECT user_id 
    FROM orders 
    WHERE total_amount > 1000
);
```

このクエリでは、ordersテーブルからtotal_amountが1000を超える注文のuser_idをサブクエリで取得し、それを使ってusersテーブルから該当するユーザーのnameを取得しています。

サブクエリを使うことで、複雑な条件やデータの取得を行うことができ、柔軟性が向上します。

Userテーブルから抽出する場合

**時間がかかるクエリ(SQL構文)**

userテーブルのデータを複数取得したい場合、id毎にクエリを発行すると、データベースへのアクセス回数が増えパフォーマンスが悪くなってしまいます。

```sql
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE id = 2;
SELECT * FROM users WHERE id = 3;
.
.
.
```

**INを使用したクエリ(SQL構文)**

userテーブルから複数のデータを取得したい場合IN句を使用すれば発行するクエリは1回で済みます。

```sql
SELECT * FROM users WHERE id IN (1, 2, 3);
```

**Railsで書いた場合**

railsでuserテーブルから1度に複数のデータを取得する場合を見てみましょう。

配列で指定する

```rb
User.where(id: [1, 2, 3])
```

`ids`メソッドを使用し同じことができます

```rb
User.where(id: User.ids)
```

どのようなSQLクエリが実行されているのか`to_sql`メソッドを使用して確認してみましょう。
  
```rb
User.where(id: [1, 2, 3]).to_sql

# => 'SELECT "users".* FROM "users" WHERE "users"."id" IN (1, 2, 3)'
```

**複数条件で絞り込みを行う**

`where`メソッドをチェーンしサブクエリを発行します。

```sh
Favorite.where(title: "title1").where(user: User.where(name: "name2"))
```

`to_sql`メソッドで確認してみましょう。<br>
`favorites`テーブルに対してサブクエリを含む2つの条件(AND条件)で絞り込みを行うクエリが発行されているのが確認できると思います。
  
```sql
SELECT "favorites".* 
  FROM "favorites" 
 WHERE "favorites"."title" = 'title1' 
   AND "favorites"."user_id" IN (
       SELECT "users"."id" FROM "users" WHERE "users"."name" = 'name2'
   )
```

`or`メソッドを使用し、OR条件で絞り込みを行う。<br>
Rails5系から`or`メソッドが実装されました。

```sh
User.where(id: 1).or(User.where(name: "user2"))
```

`to_sql`メソッドで確認してみましょう。<br>
`users`テーブルに対して2つの条件(OR条件)で絞り込みを行うクエリが発行されているのが確認できると思います。

```sql
SELECT "users".* 
  FROM "users" 
 WHERE (
   "users"."id" = 1 OR 
   "users"."name" = 'user2'
 )
```

`not`メソッドを使い否定する。<br>
`not`メソッドを使用し`where`メソッドなどで指定した条件を否定することができます。
  
```sh
User.where.not(id: 1)
```

`to_sql`メソッドで確認すると「idが1ではない(`"id" != 1"`)」というクエリが発行されているのがわかります。

```sql
SELECT "users".* 
  FROM "users" 
 WHERE "users"."id" != 1
```

### (4) 練習

railsで以下の要件をもつshop_appという名前のアプリケーションを作成しましょう。
- 以下のような店舗テーブルと商品テーブルを持つ。

`Shop: 店舗テーブル`

| field名 | 名称 | 型 |
|---|---|---|
| id | ID | integer |
| name | 店舗名 | string |

`Products: 商品テーブル`

| field名 | 名称 | 型 |
|---|---|---|
| id | ID | integer |
| shop_id | 店舗ID | integer |
| name | 商品名 | string |

- 初期データとして商品数が10未満、10以上の店舗を持つ。
- /shops/indexにアクセスした際に商品数が10以上の店舗を一覧で表示する。

####  解答(練習)

```sh
$ rails new shop_app
$ cd shop_app
$ rails g model Shop name:string
$ rails g model Product shop:references name:string
$ rails db:migrate
$ rails g controller Shops index
```

```rb
# app/models/shop.rb

Class Shop <ApplicationRecord
  has_many :products
end
```

```rb
# app/models/product.rb

Class Product < ApplicationRecord
  belongs_to :shop
end
```

```rb
# app/views/shop/index.html.erb

<h1>商品数10以上の店舗一覧</h1>
  <% @shops.each do |shop| %>
    <div>・店舗名：<%= shop.name %>　商品数：<%= shop.products.count %></div>
  <% end %>
```

```rb
# app/controller/shops_controller.rb

class ShopsController < ApplicationController
  def index
    @shops = Shop.joins(:products)
                 .group('shops.id')
                 .having('COUNT(products.id) >= 10')
  end
end
```

```rb
# /db/seeds.rb

・
・
# 店舗を作成
5.times do |i|
    shop = Shop.create(name: "店舗#{i + 1}")
  
    # 商品を作成
    (i + 8).times do |j|
      shop.products.create(name: "商品#{j + 1}")
    end
end
```
