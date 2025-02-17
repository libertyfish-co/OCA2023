# Ruby on Rails：Bootstrapの活用

## 目次
+ [はじめに](#はじめに)
+ [Bootstrap](#Bootstrap)
  + [RailsにBootstrapを導入する方法](#RailsにBootstrapを導入する方法)
  + [Bootstrapインストール](#Bootstrapインストール)
  + [デザインの適用](#デザインの適用)
  + [国際化対応（I18n）](#国際化対応i18n)
+ [練習](#練習)

<br>

---

## はじめに

Webアプリケーションの開発では、データの処理を担当する**バックエンド**と、デザインやユーザーの操作を担当する**フロントエンド**に分けて考えます。Railsでは、バックエンドは主にRubyエンジニアが担当し、フロントエンドはデザイナーが担当することが多いです。

今回は、**フロントエンド**に注目し、Webページのデザインを作成するために使われるツールである**Bootstrap**を紹介します。

---

## Bootstrap

**Bootstrap**は、Webデザインを簡単に行うためのライブラリです。多くのデザインテンプレートやUIコンポーネント（ボタン、ナビゲーションバー、カードなど）が用意されており、コードを書かなくても見た目の整ったページを作ることができます。これを使えば、レスポンシブデザイン（モバイルやタブレットに適応したデザイン）も簡単に実現できます。

---

### RailsにBootstrapを導入する方法

Rails 7では、Bootstrapをインストールする方法として、**importmap**という新しい仕組みを使うことができます。**importmap**を使うことで、`Node.js`を使わずにモダンなJavaScriptライブラリを簡単に管理できるようになります。

今回は、**gemを使ってBootstrapをインストールする方法**を紹介します。

---

### Bootstrapインストール

Rails 7では、`importmap` を使うことで `Node.js` に依存せずに `Bootstrap` を導入できます。本テキストでは、**gem**を使った**Bootstrap**のインストール方法を初心者向けに解説します。

[Bootstrap公式サイト](http://getbootstrap.com/)

※**RailsのImportmap(インポートマップ)** は、JavaScriptのモジュールやライブラリを管理し、効率的にブラウザに読み込むための仕組みです。Rails 7から導入された新機能で、従来のSprocketsエンジンに代わって、モダンなJavaScriptモジュールの取り扱いを改善するために導入されました。


1. **新しいRailsアプリケーションを作成する**  
    まず、新しいRailsアプリケーションを作成しましょう。
    ```sh
    $ rails new bootstrap_practice
    $ cd bootstrap_practice
    $ rails g scaffold User name:string phone_number:string
    $ rails db:migrate
    ```

1. **必要なgemを追加する**  
    `Gemfile` に以下の行を追加します。
    ```rb
    # Gemfile

    ・
    ・
    gem 'bootstrap', '~> 5.3.0' # Bootstrapを追加
    gem 'jquery-rails' # jQueryを追加
    gem 'sassc-rails' # Sassを使用するために追加
    ```

    追加後、以下のコマンドを実行してインストールします。

    ```sh
    $ bundle install
    ```

1. **CSSを設定する**  
    BootstrapのCSSを適用するために、`app/assets/stylesheets/application.scss` に以下の行を追加します。

    ```css
    /* app/assets/stylesheets/application.scss */

    @import "bootstrap"; /* BootstrapのCSSを読み込む */
    ```

    もし、`application.scss` が存在しない場合は、以下のコマンドで `application.css` の名前を変更してください。

    ```sh
    $ mv app/assets/stylesheets/application.css app/assets/stylesheets/application.scss
    ```

1. **JavaScriptを設定する**  
    BootstrapのJavaScriptを利用するために、`importmap.rb` に `bootstrap` と `@popperjs/core` を追加します。

    ```sh
    $ bin/importmap pin bootstrap
    ```

    次に、`config/importmap.rb` に以下の2行が追加されているか確認しましょう。

    ```rb
    # config/importmap.rb

    pin "bootstrap", to: "bootstrap.min.js", preload: true
    pin "@popperjs/core", to: "popper.js", preload: true
    ```

    また、`precompile` に `bootstrap.min.js` と `popper.js` を追加します。

    ```rb
    # config/initializers/assets.rb

    Rails.application.config.assets.precompile += %w(bootstrap.min.js popper.js)
    ```

    最後に、`app/javascript/application.js` に以下の3行を追加します。

    ```js
    // app/javascript/application.js

    //= require jquery3
    //= require popper
    //= require bootstrap-sprockets
    ```

    `import "@hotwired/turbo-rails"` よりも上に追加してください。  

以上の設定で、Bootstrapの機能が使えるようになりました。

---

#### 【補足】`Node.js` のエラーが発生した場合
`Node.js` が適切にインストールされていない場合、エラーが発生することがあります。その場合、以下の手順で対応してください。

1. `Node.js`のバージョン確認
    ```sh
    node -v
    ```
    サポートされているバージョンは `10`、`12`、`14` 以上です。それ以外の場合はアップデートが必要です。

1. `Node.js` のインストールまたは更新
    nvm（Node Version Manager）を使用すると、`Node.js` のバージョンを簡単に管理できます。

    1. nvmのインストール
        ```sh
        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
        source ~/.nvm/nvm.sh
        ```

    1. 必要なバージョンの `Node.js` をインストール
        ```sh
        nvm install 14
        ```

    1. デフォルトの `Node.js` バージョンを設定
        ```sh
        nvm alias default 14
        ```

1. プロジェクトの依存関係を再インストール
    ```sh
    $ npm install
    ```
    これで、`Node.js` の問題が解決し、Bootstrapが正しく動作するようになります。

---

### デザインの適用
Bootstrapのデザインを適用して、簡単にデザイン変更を体験してみましょう。今回は利用者のModelを作成し、氏名と電話番号を必須入力とします。

```rb
# app/models/user.rb

class User < ApplicationRecord
  validates :name, presence: true # 追加
  validates :phone_number, presence: true # 追加
end
```

次に、Bootstrapのクラスを使ってデザインを変更してみましょう。

```html
<!-- app/views/layouts/application.html.erb -->

<!DOCTYPE html>
・
・
  <body>
    <!-- ここから変更 -->
    <nav class="navbar navbar-light bg-faded">
      <h1 class="navbar-brand mb-0">Navbar</h1>
    </nav>

    <div class="container">
      <div class="row">
        <div class="col-md-9">
          <%= yield %>
        </div>
        <div class="col-md-3">
          <div>
            <h4>About</h4>
            This system is for user registration.
          </div>
        </div>
      </div>
    </div>
    <!-- ここまで変更 -->
  </body>
</html>
```

以下の3つのファイルも変更します。


- 利用者一覧  
  ![画像](images/05-6-1.png)  

  ```html
  <!-- app/views/users/_form.html.erb -->

  <%= form_with(model: user, local: true) do |form| %>
    <% if user.errors.any? %>
      <div class="card border-danger">
        <div class="card-header bg-danger text-white">
          <%= pluralize(user.errors.count, "error") %>
          prohibited this user from being saved:
        </div>
        <div class="card-body">
          <ul class="mb-0">
            <% user.errors.each do |error| %>
              <li><%= error.full_message %></li>
            <% end %>
          </ul>
        </div>
      </div>
    <% end %>

    <div class="form-group mb-3">
      <%= form.label :name %>
      <%= form.text_field :name, id: :user_name, class: "form-control" %>
    </div>

    <div class="form-group mb-3">
      <%= form.label :phone_number %>
      <%= form.text_field :phone_number, id: :user_phone_number,
      class: "form-control" %>
    </div>

    <div class="btn-group">
      <%= form.submit class: "btn btn-primary" %>
    </div>
  <% end %>
  ```

- 入力画面  
  ![画像](images/05-6-2.png)  

  ```html
  <!-- app/views/users/index.html.erb -->

  <% if notice.present? %>
    <div class="alert alert-success" role="alert">
      <%= notice %>
    </div>
  <% end %>

  <h1>Users</h1>

  <table class="table table-striped">
    <thead>
      <tr>
        <th>Name</th>
        <th>Phone number</th>
        <th colspan="3"></th>
      </tr>
    </thead>

    <tbody>
      <% @users.each do |user| %>
        <tr>
          <td><%= user.name %></td>
          <td><%= user.phone_number %></td>
          <td><%= link_to 'Show', user %></td>
          <td><%= link_to 'Edit', edit_user_path(user) %></td>
          <td><%= link_to 'Destroy', user, data: { "turbo-method": :delete, "turbo-confirm": "本当に削除しますか？" } %></td>
        </tr>
      <% end %>
    </tbody>
  </table>

  <br>

  <%= link_to 'New User', new_user_path %>
  ```

- 詳細画面  
  ![画像](images/05-6-3.png)  

  ```html
  <!-- app/views/users/show.html.erb -->

  <% if notice.present? %>
    <div class="alert alert-success" role="alert">
      <%= notice %>
    </div>
  <% end %>

  <div class="row">
    <div class="col-md-6">
      <span class="text-muted">Name:</span><%= @user.name %>
    </div>
    <div class="col-md-6">
      <span class="text-muted">Phone number:</span><%= @user.phone_number %>
    </div>
  </div>

  <%= link_to 'Edit', edit_user_path(@user) %> |
  <%= link_to 'Back', users_path %>
  ```

デザインが変更されたことを確認できたでしょうか？スクリーンショットも参考にしてください。この他にもBootstrapには多くの機能がありますので、公式のサンプルを活用してみてください。

<http://getbootstrap.com/docs/4.0/examples/>  

---

### 国際化対応（I18n）
Bootstrapを使って開発を進めるのはどうでしたか？Rubyエンジニアでも簡単に見栄えの良いWebアプリケーションを作ることができましたね。実際の現場では、デザインだけでなく、「利用する」や「使用する」といった表現の統一や、海外に展開するための英語化など、文言を変更する必要が出てきます。Railsでは、I18nというgemを使って、これらの文言を簡単に管理できるようになります。

I18nを使うと、`I18n.translate`メソッドで、あらかじめYAMLファイルに定義しておいた文言を簡単に使うことができます。設定方法については後で詳しく説明しますが、まずは次のような結果が得られることをイメージしてください。

```html
<!-- app/views/users/show.html.erb -->

<% if notice.present? %>
  <div class="alert alert-success" role="alert">
    <%= notice %>
  </div>
<% end %>

<div class="row">
  <div class="col-md-6">
    <!-- ここから変更 -->
    <span class="text-muted"><%= User.human_attribute_name('name') %>:</span>
    <%= @user.name %>
  </div>
  <div class="col-md-6">
    <span class="text-muted"><%= User.human_attribute_name('phone_number') %>:
    </span>
    <%= @user.phone_number %>
    <!-- ここまで変更 -->
  </div>
</div>

<%= link_to t('.edit'), edit_user_path(@user) %> | <!-- 変更 -->
<%= link_to t('.back'), users_path %> <!-- 変更 -->
```

`link_to`メソッドの中で「Edit」と書かれている部分は、`t('.edit')`と置き換えることができます。  
このように、メソッドから文言を参照することで、YAMLファイルだけを変更することで文言を簡単に変更できます。  
` t`は`I18n.translate`の短縮形で、同じ機能を持っています。  
さらに、カラム名は`User.human_attribute_name('name')`を使って取得できます。

それでは、I18nの設定を進めていきましょう。  
I18nの設定には、次の2つの作業があります。

1. **Railsアプリケーションの言語を日本語に設定する**  
    まず、Railsアプリケーションの言語設定を日本語に変更するため、`config/initializers/locale.rb`というファイルを作成します。

    ```sh
    $ touch config/initializers/locale.rb
    ```

    次に、作成したファイルに以下のコードを追加します。これで、アプリケーションのデフォルトの言語設定が日本語に変更されます。

    ```rb
    # config/initializers/locale.rb

    Rails.application.config.i18n.default_locale = :ja
    ```

    これで、アプリケーション内のデフォルトの言語が日本語として設定されました。

1. **YAMLファイルをダウンロードする**

    日本語向けのYAMLファイルが提供されていますので、それを利用しましょう。  
    まず、以下のコマンドを実行して、日本語用のYAMLファイルをダウンロードします。

    ```sh
    $ curl -o config/locales/ja.yml -L https://raw.github.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml
    ```

    次に、ダウンロードした`ja.yml`ファイルに、利用者に関する情報を追加していきます。YAMLファイルではインデント（字下げ）が重要ですので、編集するときは階層に注意して作業しましょう。すでに同じ内容がある場合は、重複して追加する必要はありません。  

    ファイルを編集する際は、インデントや階層構造に気をつけてください。

    ```yml
    # config/locales/ja.yml

    ---
    ja:
      activerecord:
        errors:
          messages:
            record_invalid: "バリデーションに失敗しました: %{errors}"
            restrict_dependent_destroy:
              has_one: "%{record}が存在しているので削除できません"
              has_many: "%{record}が存在しているので削除できません"
        # ここから追加
        models:
          user: 利用者
        attributes:
          user:
            name: 名前
            phone_number: 電話番号
        # ここまで追加
        ・
    ~~ 中略 ~~
        ・
        # ここから追加
      users:
        default: &default
          new: 新規作成
          edit: 編集
          show: 詳細
          destroy: 削除
          back: 戻る
          confirm: 本当に削除しますか？
        index:
          <<: *default
          title: 一覧画面
        show:
          <<: *default
        new:
          <<: *default
          title: 新規画面
        edit:
          <<: *default
          title: 編集画面
        create:
          success: 利用者を新規作成しました
        update:
          success: 利用者を更新しました
        destroy:
          success: 利用者を削除しました
        # ここまで追加
    ```

    画面が異なっていても同じ内容であれば、すでに設定した定義を再利用することができます。

    YAMLでは、**アンカー**と**エイリアス**という標準機能があります。これを使うと、同じ設定を繰り返さずに、共通部分を再利用できます。

    - `&名称`でアンカーを定義  
    - `*名称`でエイリアスとして参照

    例えば、次のように設定することができます。
    ```yaml
    edit:
      <<: *default
      title: 編集画面
    ```
    ここでは、`default`という名前でアンカーを設定しておき、`edit`の設定で`<<: *default`としてその定義を引き継いでいます。これにより、`title: 編集画面`だけを個別に変更することができ、共通部分は再利用されています。

    ---

    次に、I18nを使った設定を進めてみましょう。変更する箇所は次を参照してください。

    ```rb
    # app/controllers/users_controller.rb

    class UsersController < ApplicationController
      before_action :set_user, only: %i[ show edit update destroy ]

      # GET /users or /users.json
      def index
        @users = User.all
      end

      # GET /users/1 or /users/1.json
      def show
      end

      # GET /users/new
      def new
        @user = User.new
      end

      # GET /users/1/edit
      def edit
      end

      # POST /users or /users.json
      def create
        @user = User.new(user_params)

        respond_to do |format|
          if @user.save
            format.html { redirect_to user_url(@user), notice: t('.success') } # 変更
            format.json { render :show, status: :created, location: @user }
          else
            format.html { render :new, status: :unprocessable_entity }
            format.json { render json: @user.errors, status: :unprocessable_entity }
          end
        end
      end

      # PATCH/PUT /users/1 or /users/1.json
      def update
        respond_to do |format|
          if @user.update(user_params)
            format.html { redirect_to user_url(@user), notice: t('.success') } # 変更
            format.json { render :show, status: :ok, location: @user }
          else
            format.html { render :edit, status: :unprocessable_entity }
            format.json { render json: @user.errors, status: :unprocessable_entity }
          end
        end
      end

      # DELETE /users/1 or /users/1.json
      def destroy
        @user.destroy!

        respond_to do |format|
          format.html { redirect_to users_url, notice: t('.success') } # 変更
          format.json { head :no_content }
        end
      end

      private
      # Use callbacks to share common setup or constraints between actions.
      def set_user
        @user = User.find(params[:id])
      end

      # Only allow a list of trusted parameters through.
      def user_params
        params.require(:user).permit(:name, :phone_number)
      end
    end
    ```

    ```html
    <!-- app/views/users/index.html.erb -->

    <% if notice.present? %>
      <div class="alert alert-success" role="alert">
        <%= notice %>
      </div>
    <% end %>

    <h1><%= t('.title') %></h1> <!-- 変更 -->

    <table class="table table-striped">
      <thead>
        <tr>
          <th><%= User.human_attribute_name('name') %></th> <!-- 変更 -->
          <th><%= User.human_attribute_name('phone_number') %></th> <!-- 変更 -->
          <th colspan="3"></th>
        </tr>
      </thead>

      <tbody>
        <% @users.each do |user| %>
          <tr>
            <td><%= user.name %></td>
            <td><%= user.phone_number %></td>
            <!-- ここから変更 -->
            <td><%= link_to t('.show'), user %></td>
            <td><%= link_to t('.edit'), edit_user_path(user) %></td>
            <td><%= link_to t('.destroy'), user, data: { "turbo-method": :delete, "turbo-confirm": t('.confirm') }  %></td>
            <!-- ここまで変更 -->
          </tr>
        <% end %>
      </tbody>
    </table>

    <br>

    <%= link_to t('.new'), new_user_path %> <!-- 変更 -->
    ```

    ```html
    <!-- app/views/users/edit.html.erb -->

    <h1><%= t('.title') %></h1>

    <%= render 'form', user: @user %>

    <%= link_to t('.show'), @user %> |
    <%= link_to t('.back'), users_path %>
    ```

    ```html
    <!-- app/views/users/new.html.erb -->

      <h1><%= t('.title') %></h1>

      <%= render 'form', user: @user %>

      <%= link_to t('.back'), users_path %>
    ```

    ```html
    <!-- app/views/users/_form.html.erb -->

    <%= form_with(model: user, local: true) do |form| %>
      <% if user.errors.any? %>
        <div class="card border-danger">
          <div class="card-header bg-danger text-white">
            <%= t('errors.template.header', model: User.model_name.human, count: user.errors.size) %> <!-- 変更-->
          </div>
          <div class="card-body">
            <ul class="mb-0">
              <% user.errors.full_messages.each do |message| %> <!-- 変更 -->
                <li><%= message %></li> <!-- 変更 -->
              <% end %>
            </ul>
          </div>
        </div>
      <% end %>

      <div class="form-group">
        <%= form.label :name %>
        <%= form.text_field :name, id: :user_name, class: "form-control" %>
      </div>

      <div class="form-group">
        <%= form.label :phone_number %>
        <%= form.text_field :phone_number, id: :user_phone_number,
        class: "form-control" %>
      </div>

      <div class="actions">
        <%= form.submit class: "btn btn-primary" %>
      </div>
    <% end %>
    ```

    変更が完了したら、アプリケーションを再起動してみましょう。  
    新しくファイルを追加したり変更したりした場合、再起動することでその内容が反映されます。


## 練習
`bootstrap`を調べて自分で`railsbasic`のアプリを編集しましょう。  
1. bootstrapの導入。  
2. bootstrapの適用。  
    1. 以下を参照にしてユーザー一覧画面をテーブルにしてみましょう。  
      <https://getbootstrap.jp/docs/5.3/content/tables/>  
    1. 以下を参照にしてユーザー詳細画面をカードにしてみましょう。  
      <https://getbootstrap.jp/docs/5.3/components/card/>  
    1. 新規作成画面や編集画面を編集してみましょう。  
1. 国際化対応  
    1. I18nの導入。  
    1. I18nの適用。  
        1. 日本語に編集してみましょう。