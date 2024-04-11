## 10.2 Ruby on Rails：ECサイトの開発 注文1

### 10.2.1 確認画面
通常、入力画面と完了画面の間には、入力した内容を確認する画面があります。
しかし、Railsのscaffoldで作成したものには確認画面は存在しません。

ここでは確認画面の作成方法を説明します。

### 10.2.2 例題
scaffoldを利用せずに記事入力、確認、完了の流れを作成します。

まずは、例題用のRailsアプリケーションを作成します。

```sh
$ rails new article_sample
$ cd article_sample
```


#### ルーティングの作成
下記のようにルーティングを追加します。

今回は登録、確認、完了と一覧、詳細です。

Ruby on Railsのルーティングは、`'resouces'`メソッドを使うことで`index` `show` `new` `create` `edit` `update` `destroy`の７つの基本のルーティングが定義されます。
しかし、今回の確認画面のように、基本の７つ以外のルーティングが必要な場合があります。

その際に使うのが`member`と`collection`です。

1. `member`：ある１つのリソースに対しての操作を行う場合に利用します。つまり、そのリソースのIDが必要な操作を定義します。例えば、特定のユーザーのプロフィールを表示する、特定の投稿を編集する、特定のコメントに返信するなど、個別のリソースに対するアクションを定義します。  


2. `collection`：特定のリソースが決まっていない場合に利用します。リソース全体に対する操作やクエリを定義します。特定のIDが必要ない一般的なアクションを定義します。例えば、全てのユーザーを一覧表示する、検索結果を表示する、人気のある投稿を表示するなど、リソース全体に対するアクションを定義します。

##### memberの例
```rb
resources :user do
  member do
    get :avatar
  end
end
```
このように定義すると`/users/(:id)/avatar`というURL定義され
`UsersController#avatar`アクションが呼びだされます。
その際`params[:id]`にはアクセスしたURLで指定した(:id)の値がセットされます。
```rb
@user = User.find(params[:id])
```
このようにして、特定のリソースに対して処理を行います。

##### collectionの例
```rb
resources :user do
  collection do
    get :search
  end
end
```
このように定義すると`/users/search`というURLが定義され
`UsersController#search`アクションが呼び出されます。

最初は、`member`は`id`が必要、`collection`は`id`が不要と覚えておきましょう。

今回必要な確認画面は、まだ注文は確定していないためidは持っていません。
そのため、`member`ではなく`collection`を使いルーティングを定義します。

```rb
# config/routes.rb

resources :articles, only: [:index, :new, :create, :show] do
  collection do
    post :confirm
  end
end
```

余談ですが、変更の確認画面が必要になった場合は、`member`を利用します。

#### モデルの作成
モデルを作成するのはRailsのジェネレータを利用します。

```sh
$ rails g model Article title:string content:text
```

上記のコマンドを実行すると、モデルに必要なファイルが作成されます。
ここでマイグレーションを実行します。

```sh
$ rails db:migrate
```

これでモデルの作成は完了です。
