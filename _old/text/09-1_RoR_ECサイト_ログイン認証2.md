## 9.1 Ruby on Rails：ECサイトの開発 ログイン認証とユーザー管理2

### 9.1.1 例題

認証が必要なマイページを作成します。

ログインするユーザのモデルを作成します。  

`Devise`を使ってモデルを作成するときは独自のルールに従って作成します。  
今までは`rails g model`や`rails g scaffold`で作成していましたが、`devise`に変更するだけです。

```sh
$ rails g devise User
```
まだ、`rails db:migrate`はしないでください。  
作成ができたら、出来上がったモデルを見てみましょう。

```rb
# app/models/user.rb

class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
end
```

これは、RailsアプリケーションでDeviseを使用する際に、Deviseが提供するいくつかのモジュールを有効にするためのコードです。  
先ほど説明したDeviseを構成しているモジュールが記載されています。  
今回利用するものは下記の5つです。

- :database_authenticatable：データベースを使用してユーザーの認証を行います。ユーザーが登録したメールアドレスとパスワードを使用してログインできます。

- :registerable: ユーザーの登録機能を提供します。新しいユーザーがアプリケーションに登録できます。

- :trackable：ユーザーのログイン情報を追跡する機能を提供します。アカウントのセキュリティを向上させたり、アクティビティの追跡を行ったりすることが可能になります。

- :rememberable: ユーザーを記憶するための機能を提供します。ユーザーがログインした状態を維持し、次回の訪問時に自動的にログインできるようになります。

- :validatable：ユーザーの入力データのバリデーションを行います。例えば、メールアドレスの形式やパスワードの複雑さなどを検証します。

上記のものはコメントアウトを外し、
上記のもの以外はコメントアウトしましょう。

```rb
# app/models/user.rb

class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :recoverable and :omniauthable
  devise :database_authenticatable, :registerable,
         :trackable, :rememberable, :validatable
end
```

コメントアウトしたものは以下のようなモジュールを有効にします。  

- :recoverable: パスワードのリセット機能を提供します。ユーザーがパスワードを忘れた場合に、新しいパスワードを設定できます。

次にマイグレーションファイルを見てみましょう。


```rb
# db/migrate/xxxxxxxxxxxxxx_devise_create_users(xのところにはマイグレーションファイルを作成した日時が入ります。)

# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      # t.integer  :sign_in_count, default: 0, null: false
      # t.datetime :current_sign_in_at
      # t.datetime :last_sign_in_at
      # t.string   :current_sign_in_ip
      # t.string   :last_sign_in_ip

      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at


      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end

```

利用するモジュールに必要なカラムが記載されています。<br>
今回利用する4つのモジュールに必要なカラムはコメントアウトを外し、
それ以外のカラムはコメントアウトしましょう。

`db/migrate/xxxxxxxxxxxxxx_devise_create_users`

```rb
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      t.integer  :sign_in_count, default: 0, null: false # 編集
      t.datetime :current_sign_in_at # 編集
      t.datetime :last_sign_in_at # 編集
      t.string   :current_sign_in_ip # 編集
      t.string   :last_sign_in_ip # 編集

      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at


      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```

そのあと、マイグレーションを実行しましょう

```sh
$ rails db:migrate
```


#### マイページの作成

まずはマイページを作って表示できるようにしてみましょう。
マイページ用のルーティングを追加します。

```rb
# config/routes.rb

Rails.application.routes.draw do
  devise_for :users
  get :mypage, to: 'mypage#index' # 追加
  ・
  ・
  ・
end
```

次にマイページ用のコントローラーを作成します。

```sh
$ rails g controller mypage
```

作成されたコントローラーに下記を追記します。

```rb
# app/controllers/mypage_controller.rb 

class MypageController < ApplicationController
  def index
  end
end
```

app/views/mypageフォルダの中にindex.html.erbファイルを作成し、下記を追記します。

```sh
$ touch app/views/mypage/index.html.erb
```
```html
<!-- app/views/mypage/index.html.erb -->

<h1>マイページ</h1>
<p>ここはマイページです</p>
```

ここまでできたらサーバを起動して確認してみましょう。

`rails s`でサーバを起動して、[マイページ](http://localhost:3000/mypage)にアクセスすると、下記のような画面が表示されます。

![画像](images/09-1-1-1.png)

しかし、この状態だとログインしていない人でもアクセスできてしまいます。  
次にこのページに認証をかけて、ログインした人だけがアクセスできるようにします。  
認証をかけるには`before_action :authenticate_user!`を使います。  

このメソッドを使用することで、特定のアクションがログイン済みのユーザーにのみアクセス可能です。ユーザーがログインしていない場合、このフィルターが自動的にユーザーをログインページにリダイレクトします。  

```rb
# app/controllers/mypage_controller.rb

class MypageController < ApplicationController
  before_action :authenticate_user! # 追加
  
  def index
  end
end
```

これでマイページに認証がかかりました。早速アクセスしてみましょう。先ほど開いたページを更新ボタンや`ctrl + R`でリロードしてみましょう。

![画像](images/09-1-1-2.png)

先ほどとは違って、ログイン画面が表示されたかと思います。


次にログインするためのユーザを作成します。
先ほど表示したログイン画面に`sign up`というリンクがあるのでクリックしてみましょう。

ここでEmail, Password, Password confirmationを入力することでユーザが作成できます。

| フィールド名 |　例　|
|:--|:--|
| Email | test@example.com |
| Password | password |
| Password confirmation | password |

※テストユーザのメールアドレスを決めるときにすでに存在するドメインに注意しましょう。メールの誤送信をしてしまう可能性があります。以下は使用しないほうが良いドメインです。
- aaa.com
- abc.com
- dummy.com
- hoge.com
- sample.com
- test.com

そのため`example.com`を使用しましょう。`example.com`以外を使う場合は調べてからメールアドレスを決めましょう。    


デフォルトではユーザを作成したら、ログインした状態になります。  
早速マイページにアクセスしてみましょう。  
今度はマイページが表示されました。  
Deviseを使えば、簡単にログイン機能が実装できます。

#### ログアウトの実装

まずはルーティングを確認してみましょう

```sh
$ rails routes
```

```sh
・
・
・
destroy_user_session DELETE /users/sign_out(.:format)      devise/sessions#destroy
・
・
・
```

`delete destroy_user_session`というルーティングがあります。  
`Devise`のログインは`session`を作成、ログアウトは`session`を破棄という表現ができます。  
`delete destroy_user_session`のルーティングを使ってログアウトします。  

マイページのviewにログアウト用のリンクを追加してみましょう

```html
<!-- app/views/mypage/index.html.erb` -->

<h1>マイページ</h1>
<p>ここはマイページです</p>

<p><%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete } %></p><!-- 追加 -->
```

マイページにアクセスすると、ログアウトのリンクが表示されます。
クリックしてログアウトしてみましょう。

Railsの初期画面が表示されました。
これでログアウトができています。

試しにもう一度[マイページ](http://localhost:3000/mypage)にアクセスしてみましょう。
ログイン画面が表示されるはずです。

どうでしょうか。ログイン、ログアウトは正しく操作できたでしょうか。  
ですが、サインアップやログイン、ログアウトをしたときにルートパスに遷移してしまいます。次はこちらを変更していきます。  
サインアップ、ログイン後はmy_pageへ、ログアウト後はログイン画面へ遷移できるようにしましょう。  

`devise:controllers`コマンドを実行してControllerを作成しましょう。
```sh
$ rails g devise:controllers users
```

`after_sign_up_path_for`メソッドがコメントアウトになっているのでコメントアウトを解除しましょう。

```rb
# app/controllers/users/registrations_controller.rb

  def after_sign_up_path_for(resource)
    mypage_path
  end
```

次は、`app/controllers/application_controller.rb`を編集していきます。

```rb
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  def after_sign_in_path_for(resource)
    mypage_path
  end

  def after_sign_out_path_for(resource)
    new_user_session_path
  end
end
```

編集出来たら確認してみましょう。  
[サインアップ](http://localhost:3000/users/sign_up)もしくは、[ログイン](http://localhost:3000/users/sign_in)にアクセスしてログインしてください。  
マイページに遷移しましたか？  
次は、ログアウトボタンを押下してみましょう。ログイン画面に遷移しましたか？  
このように変更することで遷移先を変更することができます。  
