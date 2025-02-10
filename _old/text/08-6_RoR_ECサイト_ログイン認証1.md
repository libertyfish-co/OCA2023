## 8.6 Ruby on Rails：ECサイトの開発 ログイン認証とユーザー管理1

### ユーザー管理
ウェブ上にあるサービスを利用しているとアカウント登録やログイン機能を使っていると思います。ECサイトをはじめ様々なウェブアプリでは現在ユーザーの管理は必須になっています。  
ユーザーアカウントとカートを紐づけることによって今誰がどのようなものを購入しようとしているのかなど、ユーザーに関連するデータを紐づけることができます。  
ユーザーを管理することのメリットは多岐にわたり、サービスによって変わります。その中の一部をECサイトを例にして挙げると以下の通りになります。  

1. 顧客情報の集約：顧客の基本情報や購買履歴などのデータを集約し、効果的に管理することができます。これにより、顧客の好みや行動パターンを把握し、ターゲティングされたマーケティング戦略を展開することができます。

2. パーソナライズされたサービス提供：顧客ごとに異なるサービスや特典を提供することができます。例えば、過去の購買履歴や興味関心に基づいて商品のレコメンデーションを行ったり、特定のユーザーに割引クーポンを提供することが可能です。

3. セキュリティの向上：顧客の個人情報や支払い情報を安全に保管することができます。また、アカウントごとにアクセス権を管理することで、不正なアクセスや情報漏洩を防止することができます。

4. 顧客満足度の向上：顧客が容易にアカウントを作成し、注文履歴を追跡したり、配送状況を確認したりすることができます。これにより、顧客の利便性が向上し、購買体験が向上します。

5. 効率的なマーケティング：ユーザー管理システムを活用することで、顧客の購買履歴や行動データを分析し、ターゲティングされたマーケティングキャンペーンを展開することができます。顧客にとって関心のある商品やサービスを提供することで、売上や顧客ロイヤルティを向上させることができます。


### 8.6.1 ログイン認証
商品一覧ページには特定の人しかアクセスできないようにするために認証機能を追加します。  
認証を実現するために`Devise`というgemを使って実装します。  

### 8.6.2 Deviseとは
`Devise`とはRailsでも利用できる認証機能を提供してくれるgemです。  
`Devise`を使うことによってユーザー承認やパスワードのリセット、セッション管理などの機能を迅速かつ安全に実装することができます。  
いくつかのモジュールで構成されています。

|モジュール名|説明|
|---|---|
|Database Authenticatable|ログインのためのパスワードをデータベースに暗号化して保存する。|
|Oauthable|OAuthを利用する。|
|Confirmable|登録したメールアドレスにメールを送信してメールアドレスの存在を確認|
|Recoverble|パスワードを忘れた際に、パスワードリセットして再登録用のURLをメールで送信する。|
|Registerable|ログインするためのアカウントを登録する。|
|Rememberable|ログイン状態を保持する|
|Trackable|ログインした回数や日時、IPアドレスなどを保存する。|
|Timeoutable|一定期間操作がない場合に、自動的にログアウトする。|
|Validatable|メールアドレスとパスワードのバリデーションを設定する。|
|Lockable|一定回数ログインに失敗した場合、そのアカウントをロックする。|

上記の機能全てを説明できないので、今回説明できなかった機能については下記を参照してください。

https://github.com/plataformatec/devise/wiki

### 8.6.3 Deviseのインストール

まずは新しくアプリを作成します。

```sh
$ rails new devise_practice
$ cd devise_practice
```

アプリが作成出来たら`Gemfile`に`devise`を追記しましょう。
追記したら忘れずに `bundle install`を行いましょう。


`Gemfile`

```rb
・
・
・
gem 'devise' # 追加
```

`bundle install`が完了したら下記のコマンドで`Devise`をインストールします。

```sh
$ rails g devise:install
```

`devise:install`を実行すると以下のような実行結果が表示されます。  
```sh
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Depending on your application's configuration some manual setup may be required:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

     * Required for all applications. *

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

     * Not required for API-only Applications *

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

     * Not required for API-only Applications *

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

     * Not required *

===============================================================================
```

1. デフォルトのURLオプションの設定：開発環境や本番環境などに応じて、アプリケーションのURLオプションを設定します。開発環境の例が記載されており、ポート番号やホスト名などが含まれます。

2. root_urlの定義: アプリケーションのルートURLを設定します。これは、ルーティングの設定に関連しています。

3. Flashメッセージの設定: ユーザーに通知を表示するためのFlashメッセージをレイアウトに追加します。

4. Deviseのビューのコピー: カスタマイズや変更が必要な場合に、Deviseのビューをアプリケーションにコピーします。

これで`Devise`を使ったログイン認証の準備が整いました。  

### 8.6.4 ログイン処理の作成の流れ

まず、ログインに必要な情報を保存するためのモデルと、画面が必要です。

一般的に、アプリケーションやシステムにログインするためには、アカウント、パスワードが必要です。

ここでは、その情報を管理するためにモデルを作成します。

また、ログインを受け付ける画面や、ログインの状態に応じてアクセスできる情報の制御も必要ですから実装します。

ここでは、ログインの受付画面、ログインしてる場合だけアクセスできるマイページの作成を作成します。また、ログインのアカウントにはメールアドレスを利用します。最初にアクセスした場合はログインの認証に必要な情報がないので、アカウントの作成機能も必要です。

アカウント作成をする場合は、一般的に同じパスワード２回入力することが多いです。これは、入力画面のパスワード欄に文字を入力しても、一般的にはそのまま表示はしないため、同じパスワードを入力することで意図したパスワードが入力できているか確認するためです。

ログインして、画面が変わってもログインの状態を維持できるようにセッション管理機能も必要です。

ログインができたら、ログアウトも必要です。ログイン、マイページができたら、ログアウトの画面と処理を実装します。
