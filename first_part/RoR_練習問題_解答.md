## Ruby on Rails：練習問題_解答

### 目次
<details>

 - [問題1](#問題1)
 - [問題2](#問題2)
 - [問題3](#問題3)
 - [問題4](#問題4)
 - [問題5](#問題5)
 - [問題6](#問題6)
 - [問題7](#問題7)

</details>

### 問題1
<details>

<br>

ターミナル
```sh
rails new article_practice
cd article_practice
rails g model Article title:string content:text
rails db:migrate
rails g controller articles index new show edit update destroy
```

---

`/config/routes.rb`
```rb
Rails.application.routes.draw do
  resources :articles, only: [:index, :new, :create, :show, :edit, :update,:destroy] do
    collection do
      post :confirm
    end
  end

  root to: 'articles#index'
  # 省略

end
```

---

`/app/controllers/articles_controller.rb`
```rb
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id]) 
  end

  def edit
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def create
    @article = Article.new(article_params)

    if params[:back].present?
      render :new
    else
      if @article.save
        redirect_to articles_path, notice: '新規投稿しました。'
      else
        render :new
      end
    end
  end

  def update
    if params[:back].present?
      render :edit
    else
      @article = Article.find(params[:id])
      if @article.update(article_params)
        redirect_to articles_path, notice: '投稿を更新しました。'
      else
        render :edit
      end
    end

  end

  def destroy
    @article = Article.find(params[:id])
    @article.destroy
    redirect_to articles_path, alert: '投稿を削除しました。'
  end

  private

  def article_params
    params.require(:article).permit(:title, :content)
  end

end
```

---

`/app/views/articles/index.html.erb`
```html
<p class="color-green" id="notice"><%= notice %></p> # 登録・更新
<p class="color-red" id="alert"><%= alert %></p>     # 削除

<h1>投稿一覧</h1>

<%= link_to '新規投稿', new_article_path %>

<table class="table width-800">
  <thead>
    <tr>
      <th class="border textalign-c width-300">タイトル</th>
      <th class="border textalign-c width-500">投稿内容</th>
    </tr>
  </thead>

  <tbody>
    <% @articles.each do |article| %>
      <tr>
        <td class="border">
          <%= link_to article.title, article_path(article) %> 
        </td>
        <td class="border">
          <%= article.content %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>
```

---

`/app/views/articles/new.html.erb`
```html
<h1>新規投稿</h1>
<%= form_with(model: @article, local: true, data: { turbo: false }) do |form| %>
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@article.errors.count, "error") %> prohibited this article from being saved:</h2>
      <ul>
        <% @article.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= form.label :タイトル %>
    <%= form.text_field :title, id: :article_title %>
  </div>

  <div class="field">
    <%= form.label :投稿内容 %>
    <%= form.text_area :content, id: :article_content %>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>

<%= link_to 'Back', articles_path %>
```

---

`/app/views/articles/show.html.erb`
```html
<h1>投稿詳細</h1>

<p>
  <strong>タイトル：</strong>
  <%= @article.title %>
</p>

<p>
  <strong>投稿内容：</strong>
  <%= @article.content %>
</p>

<div>
  <%= link_to "投稿編集へ遷移", edit_article_path(@article) %> |
  <%= link_to "一覧画面へ戻る", articles_path %>

  <%= button_to "削除ボタン", @article, method: :delete %>
</div>
```

---

`/app/views/artcles/edit.html.erb`
```html
<h1>投稿編集</h1>

<%= form_with(model: @article, local: true, url: article_path, data: { turbo: false }) do |form| %>
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@article.errors.count, "error") %> prohibited this article from being saved:</h2>
      <ul>
        <% @article.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :タイトル %>
    <%= form.text_field :title, id: :article_title %>
  </div>

  <div>
    <%= form.label :投稿内容 %>
    <%= form.text_area :content, id: :article_content %>
  </div>

  <div class="actions">
    <%= form.submit '投稿' %>
  </div>
<% end %>

<div>
  <%= link_to "投稿詳細へ戻る", article_path(@article) %> |
</div>
```

---

`/app/assets/stylesheets/application.css`
```css

 # 省略

 .width-300 {
    width: 300px;
 }

 .width-500 {
    width: 500px;
 }

 .width-800 {
    width: 800px;
 }

 .table {
    border-collapse: collapse;
 }

 .border {
    border: 1px solid #000;
 }

 .textalign-c {
    text-align: center;;
 }

 .color-green {
    color: #008000;
 }

 .color-red {
   color: #ff0000;
 }
```
</details>

<br>

### 問題2
<details>

<br>

ターミナル
```sh
rails new people_lists
rails g scaffold Member name:string height:float weight:float
rails db:migrate
```

---

`/app/models/member.rb`
```rb
class Member < ApplicationRecord
    validates :name, presence: true, length: { in: 1..20 }, uniqueness: true
    validate :must_be_1_height
    validate :must_be_1_weight

    private

    def must_be_1_height
      if height.present? && height < 1
        errors.add(:height, "Must be 1 or greater")
      end
    end

    def must_be_1_weight
      if weight.present? && weight < 1
        errors.add(:weight, "Must be 1 or greater")
      end
    end
end
```
</details>

<br>

### 問題3
<details>

<br>

ターミナル
```sh
rails new bcrypt_practice
rails cd bcrypt_practice

# gemfile を編集してから
rails bundle install

# コントローラー作成
rails g controller Users top
rails g controller Sessions new 
rails g controller Pages my_page

# Userモデル作成
rails g User name:string email:string password_digest:string
rails db:migrate

# ビューの作成
touch app/views/users/top.html.erb
touch app/views/users/signup.html.erb
```

---

`/gemfile`
```rb
# 省略

# Use Active Model has_secure_password [https://guides.rubyonrails.org/active_model_basics.html#securepassword]
gem "bcrypt", "~> 3.1.7" #を外す

# 省略
```

---

`/config/routes.rb`
```rb
Rails.application.routes.draw do
  # トップ画面　初期画面設定
  root 'users#top'

  # サインアップ
  get 'signup' => 'users#signup'
  post 'create' => 'users#create'

  # ログイン/ログアウト
  resource :session, only: [:new, :create, :destroy]

  # マイページ
  get 'my_page', to: 'pages#my_page'

  # 省略
end
```

---

`/app/models/user.rb`
```rb
class User < ApplicationRecord
  has_secure_password # 追加
end
```

---

`/app/controllers/sessions_controller.rb`
```rb
class SessionsController < ApplicationController
  def new
  end

  # 追加　ここから
  def create # ログイン処理
    user = User.find_by(email: params[:email])
    if user && user.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to my_page_path
    else
    　# 一致しなかった場合、ログイン画面に戻ってフラッシュメッセージを表示
      flash[:notice] = 'メールアドレスまたはパスワードが一致しません。'
      redirect_to session_path
    end
  end

  def destroy
    session.delete(:user_id)
    redirect_to root_path
  end
  # 追加　ここまで

end
```

---

`/app/controllers/pages_controller.rb`
```rb
class PagesController < ApplicationController
  before_action :authenticate_user! # 追加

  def my_page
  end

　# 追加　ここから
  def destroy
    session.delete(:user_id)
    redirect_to root_path
  end

  private

  def authenticate_user!
    redirect_to root_path unless session[:user_id]
  end
  # 追加　ここまで

end
```

---

`/app/helpers/application_helper.rb`
```rb
module ApplicationHelper
  # 追加　ここから
  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end
  # 追加　ここまで
end
```

---

`/app/views/users/top.html.erb`
```html
<h1>Top画面</h1>

<%= link_to 'サインアップ', signup_path %> |
<%= link_to 'ログイン', new_session_path %>
```

---

`/app/views/users/signup.html.erb`
```html
<h1>サインアップ</h1>

<%= form_with model: @user, url: create_path do |form| %>
  <% if @user.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@user.errors.count, "error") %> prohibited this user from being saved:</h2>
      <ul>
        <% @user.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :名前, style: "display: block" %>
    <%= form.text_field :name, id: :user_name %>
  </div>

  <div>
    <%= form.label :メールアドレス, style: "display: block" %>
    <%= form.text_area :email, id: :user_email %>
  </div>

  <div>
    <%= form.label :パスワード, style: "display: block" %>
    <%= form.password_field :password, id: :user_password %>
  </div>

  <div>
    <%= form.label :パスワードの確認, style: "display: block" %>
    <%= form.password_field :password_confirmation, id: :user_password_confirmation %>
  </div>
  
  <br>

  <div class="actions">
    <%= form.submit 'サインアップ' %>
  </div>
<% end %>

<br>

<%= link_to 'Top画面に戻る', root_path %>
```

---

`/app/views/sessions/new.html.erb`
```html
<div>
  <%= flash[:notice] %>
</div>

<h1>ログイン</h1>

<%= form_with url: session_path, method: :post do |form| %>
  <% if flash[:alert] %>
    <div class="alert"><%= flash[:alert] %></div>
  <% end %>

  <div>
    <%= form.label :メールアドレス, style: "display: block" %>
    <%= form.text_area :email %>
  </div>

  <div>
    <%= form.label :パスワード, style: "display: block" %>
    <%= form.password_field :password %>
  </div>

  <br>

  <div>
    <%= form.submit "ログイン" %>
  </div>
<% end %>

<br>

<%= link_to 'Top画面に戻る', root_path %>
```

---

`/app/views/pages/my_page.html.erb`
```html
<h1>My_Page画面</h1>

<p>
  <strong>名前：</strong>
  <%= current_user.name %>
</p>

<p>
  <strong>メールアドレス：</strong>
  <%= current_user.email %>
</p>

<p>
  <strong>ハッシュ化パスワード：</strong>
  <%= current_user.password_digest %>
</p>

<%= link_to 'ログアウト', session_path, data: { turbo_method: :delete } %>
```
</details>

<br>

### 問題4
<details>

<br>

ターミナル
```sh
rails new devise_practice
rails cd devise_practice

# gemfile を編集してから
bundle install
rails g devise:install

# Deviseでモデルの作成
rails g devise User

# マイグレーションファイルを編集してから
rails db:migrate

# コントローラーの作成
rails g devise:controllers users
rails g controller Mypage mypage
rails g controller Top top

# ビューの作成
rails g devise:views
touch app/views/mypage/index.html.erb
touch app/views/top/top.html.erb
```

---

`/gemfile`
```rb
# 省略

# Use Active Model has_secure_password [https://guides.rubyonrails.org/active_model_basics.html#securepassword]
gem 'devise' # deviseの追加

# 省略
```

---

`/config/initializers/devise.rb`
```rb
  
  # 省略

  # ==> Configuration for any authentication mechanism
  # Configure which keys are used when authenticating a user. The default is
  # just :email. You can configure it to use [:username, :subdomain], so for
  # authenticating a user, both parameters are required. Remember that those
  # parameters are used only when authenticating and not when retrieving from
  # session. If you need permissions, you should implement that in a before filter.
  # You can also supply a hash where the value is a boolean determining whether
  # or not authentication should be aborted when the value is not present.
  config.authentication_keys = [:email] #を外す
  # 50行目あたり

  # 省略

```

---

`/db/migrate/202XXXXXXXXXXX_devise_create_users.rb`
```rb
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.1]
  def change
　# 省略

      ## Trackable
      # #を外す ここから
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.string   :current_sign_in_ip
      t.string   :last_sign_in_ip
      # #を外す ここまで

  # 省略

end
```

---

`/config/routes.rb`
```rb
Rails.application.routes.draw do
  root to: 'top#top' # 追加
  
  devise_for :users
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  get :mypage, to: 'mypage#index' # 追加

　# 省略
end
```

---

`/app/controllers/application_controller.rb`
```rb
class ApplicationController < ActionController::Base
  # 追加　ここから
  before_action :configure_permitted_parameters, if: :devise_controller?

  def after_sign_in_path_for(resource)
      mypage_path
  end

  def after_sign_out_path_for(resource)
      root_path
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
  end
  # 追加　ここまで
end
```

---

`/app/views/controllers/users/registrations_controller.rb`
```rb
# frozen_string_literal: true

class Users::RegistrationsController < Devise::RegistrationsController
  # 省略

  # 追加　ここから
  def after_sign_up_path_for(resource)
    mypage_path
  end
  # 追加　ここまで

end
```

---

`/app/controllers/mypage_controller.rb`
```rb
class MypageController < ApplicationController
  before_action :authenticate_user! # 追加

  def index
  end
end
```

---

`/app/assets/stylesheets/application.css`
```css

# 省略

 .width-200 {
    width: 200px;
 }

 .width-500 {
    width: 500px;
 }

 .border {
    border: 1px solid #000;
 }

 .text-align-c {
    text-align: center;
 }

 .table {
    border-collapse: collapse;
 }

```

---

`/app/views/top/top.html.erb`
```html
<h1>トップ画面</h1>

<%= link_to "ログイン", new_user_session_path %> | 
<%= link_to "サインアップ", new_user_registration_path %>
```

---

`/app/views/divese/registrations/new.html.erb`
```html
<h2>サインアップ画面</h2>

<%= form_with model: @user, url: registration_path(resource_name) do |f| %>
  <%= render "devise/shared/error_messages", resource: resource %>

  <div class="field">
    <%= f.label :名前 %><br />
    <%= f.text_field :name, autofocus: true, autocomplete: "name" %>
  </div>

  <div class="field">
    <%= f.label :メールアドレス %><br />
    <%= f.email_field :email, autocomplete: "email" %>
  </div>

  <div class="field">
    <%= f.label :パスワード %>
    <% if @minimum_password_length %>
    <em>(<%= @minimum_password_length %> 文字以上)</em>
    <% end %><br />
    <%= f.password_field :password, autocomplete: "new-password" %>
  </div>

  <div class="field">
    <%= f.label :パスワードの確認 %><br />
    <%= f.password_field :password_confirmation, autocomplete: "new-password" %>
  </div>

  <p></p>

  <div class="actions">
    <%= f.submit "サインアップ" %>
  </div>
<% end %>

<br />

<%= render "devise/shared/links" %>

<%= link_to "トップ画面に戻る", root_path %>
```

---

`/app/views/devise/sessions/new.html.erb`
```html
<h2>ログイン画面</h2>

<%= form_with model: @user, url: session_path(resource_name) do |f| %>
  <div class="field">
    <%= f.label :メールアドレス %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>

  <div class="field">
    <%= f.label :パスワード %><br />
    <%= f.password_field :password, autocomplete: "current-password" %>
  </div>

  <% if devise_mapping.rememberable? %>
    <div class="field">
      <%= f.check_box :remember_me %>
      <%= f.label :ログイン状態を保持 %>
    </div>
  <% end %>

  <p></p>

  <div class="actions">
    <%= f.submit "ログイン" %>
  </div>
<% end %>

<br />

<%= render "devise/shared/links" %>

<%= link_to "トップ画面に戻る", root_path %>
```

---

`/app/views/devise/shared/_links.html.erb`
```html
<%- if controller_name != 'sessions' %>
  <%= link_to "ログイン", new_session_path(resource_name) %><br /> # 編集
<% end %>

<%- if devise_mapping.registerable? && controller_name != 'registrations' %>
  <%= link_to "サインアップ", new_registration_path(resource_name) %><br /> # 編集
<% end %>

<%- if devise_mapping.recoverable? && controller_name != 'passwords' && controller_name != 'registrations' %>
  <%= link_to "パスワードをお忘れですか?", new_password_path(resource_name) %><br /> # 編集
<% end %>

# 省略
```

---

`/app/views/mypage/index`
```html
<h1>マイページ画面</h1>

<table class="table">
  <tr>
    <th class="width-200 border">名前</th>
    <td class="width-500 border">
      <%= current_user.name %>
    </td>
  </tr>

  <tr>
    <th class="width-200 border">メールアドレス</th>
    <td class="width-500 border">
      <%= current_user.email %>
    </td>
  </tr>
</table>

<p><%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete } %></p>
```
</details>

<br>

### 問題5
<details>

<br>

ターミナル
```sh
rails new sns_site
rails cd sns_site

# gemfile を編集してから
rails g devise:install

# Deviseでモデルの作成
rails g devise User
rails g scaffold Post title:string comment:text user_id:integer
rails g scaffold Favorite user:references post:references

# マイグレーションファイルを編集してから
rails db:migrate

# コントローラーの作成
rails g devise:controllers Users
rails g controller Top top
rails g controller Mypages

# ビューの作成
rails g devise:views
touch app/views/top/top.html.erb
touch app/views/mypage/show.html.erb
touch app/views/mypage/edit.html.erb
```

---

`/Gemfile`
```gemfile
# 省略

gem 'devise' # 追加
```

---

`/db/migrate/xxxxxxxxxxxxxx_devise_create_users.rb`
```rb
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      ## Database authenticatable

      # 追加　ここから
      t.string :name, null: false                             # 名前
      t.text :content                                         # 自己紹介
      # 追加　ここまで
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      # 省略

      ## Trackable
      t.integer  :sign_in_count, default: 0, null: false    #を外す
      t.datetime :current_sign_in_at                        #を外す
      t.datetime :last_sign_in_at                           #を外す
      t.string   :current_sign_in_ip                        #を外す
      t.string   :last_sign_in_ip                           #を外す

      # 省略
    end
  end
end

```

---

`/config/routes.rb`
```rb
Rails.application.routes.draw do
  get 'top/top'                                                         # 追加
  root to: 'top#top'                                                    # 追加
  resources :mypages, only: [:show, :edit, :update]                     # 追加

  resources :posts, only: [:new, :create, :index, :show, :destroy] do
    resource :favorite, only: [:create, :destroy]                       # 追加
  end

  devise_for :users

  # 省略
end
```

---

`/config/initializers/devise.rb`
```rb

# 省略

# ==> Configuration for any authentication mechanism
  # Configure which keys are used when authenticating a user. The default is
  # just :email. You can configure it to use [:username, :subdomain], so for
  # authenticating a user, both parameters are required. Remember that those
  # parameters are used only when authenticating and not when retrieving from
  # session. If you need permissions, you should implement that in a before filter.
  # You can also supply a hash where the value is a boolean determining whether
  # or not authentication should be aborted when the value is not present.
  config.authentication_keys = [:name]

# 省略
```

---

`/app/models/user.rb`
```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  # 追加　ここから
  has_many :posts, dependent: :destroy
  has_many :favorites, dependent: :destroy
  # 追加　ここまで
end
```

---

`/app/models/post.rb`
```rb
class Post < ApplicationRecord
    # 追加　ここから
    belongs_to :user
    has_many :favorites, dependent: :destroy

    def favorited_by?(user)
        favorites.exists?(user_id: user.id)
    end
    # 追加　ここから
end
```

---

`/app/models/favorite.rb`
```rb
class Favorite < ApplicationRecord
  belongs_to :user
  belongs_to :post

  validates :user_id, uniqueness: {scope: :post_id} # 追加
end
```

---

`/app/controllers/application_controller.rb`
```rb
class ApplicationController < ActionController::Base
 
  # 追加　ここから
  before_action :configure_permitted_parameters, if: :devise_controller?

  def after_sign_in_path_for(resource)
      mypage_path(resource)
  end

  def after_sign_out_path_for(resource)
      root_path
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name, :email])
  end
  # 追加　ここまで

end
```

---

`/app/controllers/users/registrations_controller.rb`
```rb
# frozen_string_literal: true

class Users::RegistrationsController < Devise::RegistrationsController
 
 # 省略

   # The path used after sign up.
  def after_sign_up_path_for(resource)
    mypage_path(resource) # 編集
  end

  # The path used after sign up for inactive accounts.
  # def after_inactive_sign_up_path_for(resource)
  #   super(resource)
  # end
end
```

---

`/app/controllers/posts_controller.rb`
```rb
class PostsController < ApplicationController
  before_action :authenticate_user! # 追加
  before_action :set_post, only: %i[ show edit update destroy ]

  # GET /posts or /posts.json
  def index
    @posts = Post.all
  end

  # GET /posts/1 or /posts/1.json
  def show
  end

  # GET /posts/new
  def new
    @post = Post.new
  end

  # GET /posts/1/edit
  def edit
  end

  # POST /posts or /posts.json
  def create
    @post = Post.new(post_params)
    @post.user_id = current_user.id  # 追加

    respond_to do |format|
      if @post.save
        format.html { redirect_to @post, notice: "投稿しました" } # 編集
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /posts/1 or /posts/1.json
  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to @post, notice: "投稿を更新しました" } # 編集
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /posts/1 or /posts/1.json
  def destroy
    @post.destroy!

    respond_to do |format|
      format.html { redirect_to posts_path, status: :see_other, notice: "投稿を削除しました" } # 編集
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def post_params
      params.require(:post).permit(:title, :comment)
    end
end
```

---

`/app/controllers/favorites_controller.rb`
```rb
class FavoritesController < ApplicationController
  # before_action :set_favorite, only: %i[ show edit update destroy ]
  before_action :authenticate_user! # 追加

  # POST /favorites or /favorites.json
  def create
    # コメントアウト ここから
    # @favorite = Favorite.new(favorite_params)

    # respond_to do |format|
    #   if @favorite.save
    #     format.html { redirect_to @favorite, notice: "Favorite was successfully created." }
    #     format.json { render :show, status: :created, location: @favorite }
    #   else
    #     format.html { render :new, status: :unprocessable_entity }
    #     format.json { render json: @favorite.errors, status: :unprocessable_entity }
    #   end
    # end
    # コメントアウト　ここまで

    # 追加　ここから
    post = Post.find(params[:post_id])
    favorite = current_user.favorites.new(post_id: post.id)
    favorite.save
    redirect_to post_path(post)
    # 追加　ここまで
  end

  # DELETE /favorites/1 or /favorites/1.json
  def destroy
    # コメントアウト　ここから
    # @favorite.destroy!

    # respond_to do |format|
    #   format.html { redirect_to favorites_path, status: :see_other, notice: "Favorite was successfully destroyed." }
    #   format.json { head :no_content }
    # end
    # コメントアウト　ここまで
  
    # 追加　ここから
    post = Post.find(params[:post_id])
    favorite = current_user.favorites.find_by(post_id: post.id)
    favorite.destroy
    redirect_to post_path(post)
    # 追加　ここまで

  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_favorite
      @favorite = Favorite.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def favorite_params
      params.require(:favorite).permit(:user_id, :post_id)
    end
end
```

---

`/app/controllers/mypages_controller.rb`
```rb
class MypagesController < ApplicationController
  before_action :authenticate_user! # 追加

  def show
    @user = User.find(params[:id]) # 追加
  end

  def edit
    # 追加　ここから
    @user = current_user
    if @user.id != current_user.id
      redirect_to mypage_path(current_user)
    end
    # 追加　ここまで
  end

  def update
    # 追加　ここから
    user = User.find(params[:id])
    user.update(user_params)
    redirect_to mypage_path(user.id)
    # 追加　ここまで
  end

  # 追加　ここから
  private

  def user_params
    params.require(:user).permit(:name, :email, :content)
  end
  # 追加　ここまで
    
end
```

---

`/app/views/devise/registrations/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->
  <h2>サインアップ</h2>

  <%= form_with model: @user, url: registration_path(resource_name) do |f| %> <!-- 編集 -->
    <%= render "devise/shared/error_messages", resource: resource %>

    <!-- 追加　ここから -->
    <div class="field">
      <%= f.label :名前 %><br />
      <%= f.text_field :name, autofocus: true, autocomplete: "name" %>
    </div>
    <!-- 追加　ここまで -->

    <div class="field">
      <%= f.label :メールアドレス %><br /> <!-- 編集 -->
      <%= f.email_field :email, autocomplete: "email" %>
    </div>

    <div class="field">
      <%= f.label :パスワード %> <!-- 編集 -->
      <% if @minimum_password_length %>
      <em>(<%= @minimum_password_length %> 文字以上)</em> <!-- 編集 -->
      <% end %><br />
      <%= f.password_field :password, autocomplete: "new-password" %>
    </div>

    <div class="field">
      <%= f.label :パスワードの確認 %><br /> <!-- 編集 -->
      <%= f.password_field :password_confirmation, autocomplete: "new-password" %>
    </div>

    <div class="actions">
      <%= f.submit "サインアップ" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>

  <%= link_to "トップ画面に戻る", root_path %> <!-- 追加 -->

</div> <!-- 追加 -->
```

---

`/app/views/devise/sessions/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->
  <h2>ログイン</h2> <!-- 編集 -->

  <%= form_with model: @user, url: session_path(resource_name) do |f| %>
    <div class="field">
      <%= f.label :名前 %><br /> <!-- 編集 -->
      <%= f.text_field :name, autofocus: true, autocomplete: "name" %> <!-- 編集 -->
    </div>

    <div class="field">
      <%= f.label :パスワード %><br /> <!-- 編集 -->
      <%= f.password_field :password, autocomplete: "current-password" %>
    </div>

    <% if devise_mapping.rememberable? %>
      <div class="field">
        <%= f.check_box :remember_me %>
        <%= f.label :ログイン状態を保持 %> <!-- 編集 -->
      </div>
    <% end %>

    <div class="actions">
      <%= f.submit "ログイン" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>

  <%= link_to "トップ画面に戻る", root_path %> <!-- 追加 -->

</div> <!-- 追加 -->
```

---

`/app/views/devise/registrations/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->
  <h2>サインアップ</h2>

  <%= form_with model: @user, url: registration_path(resource_name) do |f| %> <!-- 編集 -->
    <%= render "devise/shared/error_messages", resource: resource %>

    <!-- 追加　ここから -->
    <div class="field">
      <%= f.label :名前 %><br />
      <%= f.text_field :name, autofocus: true, autocomplete: "name" %>
    </div>
    <!-- 追加　ここまで -->

    <div class="field">
      <%= f.label :メールアドレス %><br /> <!-- 編集 -->
      <%= f.email_field :email, autocomplete: "email" %>
    </div>

    <div class="field">
      <%= f.label :パスワード %> <!-- 編集 -->
      <% if @minimum_password_length %>
      <em>(<%= @minimum_password_length %> 文字以上)</em> <!-- 編集 -->
      <% end %><br />
      <%= f.password_field :password, autocomplete: "new-password" %>
    </div>

    <div class="field">
      <%= f.label :パスワードの確認 %><br /> <!-- 編集 -->
      <%= f.password_field :password_confirmation, autocomplete: "new-password" %>
    </div>

    <div class="actions">
      <%= f.submit "サインアップ" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>

  <%= link_to "トップ画面に戻る", root_path %> <!-- 追加 -->

</div> <!-- 追加 -->
```

---

`/app/views/devise/shared/_links.html.erb`
```html
<%- if controller_name != 'sessions' %>
  <%= link_to "ログイン", new_session_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<%- if devise_mapping.registerable? && controller_name != 'registrations' %>
  <%= link_to "サインアップ", new_registration_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<%- if devise_mapping.recoverable? && controller_name != 'passwords' && controller_name != 'registrations' %>
  <%= link_to "パスワードをお忘れですか?", new_password_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<!-- 省略 -->
```

---

`/app/views/layouts/application.html.erb`
```html
<!DOCTYPE html>
<html>
  <!-- 省略 -->

  <body>
    <!-- 追加 ここから -->
    <header class="header display-f">
      <% if user_signed_in? %>
        <div class="width-200 text-align-c padding-l-15p">
          <%= link_to "新規投稿", new_post_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "投稿一覧", posts_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "マイページ", mypage_path(current_user.id), class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete }, class: "color-white" %>
        </div>
      <% else %>
        <div class="width-200 text-align-c padding-l-15p">
          <%= link_to "サインアップ", new_user_registration_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "ログイン", new_user_session_path, class: "color-white" %>
        </div>
      <% end %>
    </header>
    <!-- 追加　ここまで -->

    <%= yield %>
  </body>
</html>
```

---

`/app/views/top/top.html.erb`
```html
<!-- 追加　ここから -->
<h1 class="text-align-c">SNSサイト</h1>

<div class="text-align-c"> 
  <%= link_to "サインアップ", new_user_registration_path %> |
  <%= link_to "ログイン", new_user_session_path %>
</div>
<!-- 追加　ここまで -->
```

---

`/app/views/mypages/show.html.erb`
```html
<!-- 追加　ここから -->
<div class="padding-l-15p">

  <p style="color: green"><%= notice %></p>

  <h2>
    <%= @user.name %>
  </h2>

  <% if @user == current_user %>
    <div class="width-120 border-green border-radius-3 text-align-c background-color-green">
      <%= link_to "マイページ編集", edit_mypage_path, class: "color-white" %>
    </div>
  <% end %>

  <p>
    <strong>メールアドレス:</strong>
    <%= @user.email %>
  </p>

  <p>
    <strong>自己紹介:</strong>
    <%= @user.content %>
  </p>

</div>
<!-- 追加　ここまで -->
```

---

`/app/views/mypages/edit.html.erb`
```html
<!-- 追加　ここから -->
<div class="padding-l-15p">

  <h1>編集</h1>

  <%= form_with model: @user, url: mypage_path do |form| %>
    <% if @user.errors.any? %>
      <div style="color: red">
        <h2><%= pluralize(@user.errors.count, "error") %> prohibited this user from being saved:</h2>

        <ul>
          <% @user.errors.each do |error| %>
            <li><%= error.full_message %></li>
          <% end %>
        </ul>
      </div>
    <% end %>

    <div>
      <%= form.label :名前, style: "display: block" %>
      <%= form.text_field :name %>
    </div>

    <div>
      <%= form.label :自己紹介, style: "display: block" %>
      <%= form.text_area :content %>
    </div>

    <div>
      <%= form.submit '編集' %>
    </div>
  <% end %>

</div>
<!-- 追加　ここまで -->
```

---

`/app/views/posts/_post.html.erb`
```html
<div id="<%= dom_id post %>">
  <p>
    <strong>タイトル:</strong> <!-- 編集 -->
    <%= post.title %>
  </p>

  <p>
    <strong>投稿内容:</strong> <!-- 編集 -->
    <%= post.comment %>
  </p>

</div>
```

---

`/app/views/posts/index.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <p style="color: green"><%= notice %></p>

  <h1>投稿一覧</h1> <!-- 編集 -->
  
  <!-- 下から移動 ここから -->
  <p>
    <%= link_to "新規投稿", new_post_path %> <!-- 編集 -->
  </p>
  <!-- 下から移動 ここまで -->

  <!-- 追加　ここから -->
  <div id="posts">
    <table>
      <!-- ヘッダー -->
      <tr>
        <th class="width-200">タイトル</th>
        <th class="width-600">投稿内容</th>
        <th class="width-120">お気に入り数</th>
        <th class="width-120"></th>
      </tr>

      <!-- ボディ -->
      <% @posts.each do |post| %>
        <tr id="<%= post.id %>">
          <td class="">
            <%= post.title %>
          </td>

          <td class="">
            <%= post.comment %>
          </td>

          <td class="text-align-c">
            ★<%= post.favorites.count %>
          </td>          

          <td>
            <%= link_to "投稿詳細", post %> <!-- 編集 -->
          </td>
      <% end %>
    </table>
  </div>
  <!-- 追加　ここから -->

</div>
```

---

`/app/views/posts/_form.html.erb`
```html
<%= form_with(model: post) do |form| %>
  <% if post.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>

      <ul>
        <% post.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :タイトル, style: "display: block" %> <!-- 編集 -->
    <%= form.text_field :title %>
  </div>

  <div>
    <%= form.label :投稿内容, style: "display: block" %> <!-- 編集 -->
    <%= form.text_area :comment %>
  </div>

  <div>
    <%= form.submit '投稿' %> <!-- 編集 -->
  </div>
<% end %>
```

---

`/app/views/posts/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <h1>新規投稿</h1> <!-- 編集 -->

  <%= render "form", post: @post %>

  <br>

  <!-- 削除　ここから -->
  <div>
    <%= link_to "Back to posts", posts_path %>
  </div>
  <!-- 削除　ここまで -->

</div> <!-- 追加 -->
```

---

`/app/views/posts/show.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <p style="color: green"><%= notice %></p>

  <%= render @post %>

  <div>
    <!-- 追加　ここから -->
    <% if @post.favorited_by?(current_user) %>
      <p>
        <%= link_to post_favorite_path(@post),  data: { "turbo-method": :delete } do %>
          ★<%= @post.favorites.count %> お気に入り
        <% end %>
      </p>
    <% else %>
      <p>
        <%= link_to post_favorite_path(@post), data: { "turbo-method": :post } do %>
          ☆<%= @post.favorites.count %> お気に入り
        <% end %>
      </p>
    <% end %>

    <!-- 投稿者のみ削除できるようにする -->
    <% if @post.user == current_user %>
      <%= button_to "投稿を削除", @post, method: :delete %>
    <% end %>
  　<!-- 追加　ここまで -->

　　<!-- 削除　ここから -->
    <%= link_to "Edit this favorite", edit_favorite_path(@favorite) %> |
    <%= link_to "Back to favorites", favorites_path %>

    <%= button_to "Destroy this favorite", @favorite, method: :delete %>
　　<!-- 削除　ここまで -->

  </div>

</div>
```

---

`/app/assets/stylesheets/application.css`
```css
/*
 省略
 */

 /* リンク下線消し */
 a {
    text-decoration: none;
 }

 /* ヘッダー */
 .header {
    width: 100%;
    height: 50px;
    line-height: 50px;
    background: #004ba7;
 }

 /* 横幅 */
 .width-120 {
    width: 120px;
 }

 .width-200 {
    width: 200px;
 }

 .width-600 {
    width: 600px;
 }

 /* 間隔 */
 .padding-5 {
    padding: 5px;
 }

 .padding-l-15p {
    padding-left: 15%;
 }

 /* display */
 .display-f {
    display: flex;
 }

 /* 文字位置　横 */
 .text-align-c {
    text-align: center;
 }

 /* 文字色 */
 .color-white {
    color: #ffffff;
 }

 /* 背景色 */
 .background-color-green {
    background-color: #01bf33;
 }

 /* 線 */
 .border {
    border: 1px solid #000000;
 }

 .border-green {
    border: 1px solid #01bf33;
 }

 /* 線の丸み */
 .border-radius-3 {
    border-radius: 3px;
 }
```

</details>

<br>

### 問題6

<details>

<br>

ターミナル
```bash
rails new photo_post
cd photo_post

# active_storageのインストール
rails active_storage:install

# gemfileに必要なgemを記入後
bundle install

# deviseの導入
rails g devise:install
rails g devise User

# postモデルの作成
rails g scaffold Post tilte:string content:text image:string user_id:integer

# migrateファイルを編集後
rails db:migrate

# コントローラーの作成
rails g devise:controllers Users
rails g controller Mypages show edit update

# ビューの作成
rails g devise:views

# 不必要なビューの削除
rm app/views/mypages/update.html.erb # マイページ更新画面
rm app/views/posts/edit.html.erb     # 投稿編集画面
```

---

`Gemfile`
```rb
# 省略

# 追加
gem 'devise'            # ユーザー認証
gem 'image_processing'  # 画像のリサイズ
```

---

`/config/routes.rb`
```rb
Rails.application.routes.draw do
  # 削除　ここから
  get 'mypages/show'
  get 'mypages/edit'
  get 'mypages/update'
  # 削除　ここまで
  resources :mypages, only: [:show, :edit, :update]                     # 追加
  resources :posts
  devise_for :users
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Reveal health status on /up that returns 200 if the app boots with no exceptions, otherwise 500.
  # Can be used by load balancers and uptime monitors to verify that the app is live.
  get "up" => "rails/health#show", as: :rails_health_check

  # Defines the root path route ("/")
  # root "posts#index"
end


```


---

`/db/migrate/xxxxxxxxxxxxxx_devise_create_users.rb`
```rb
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""
      # 追加　ここから
      t.string :name
      t.string :avatar
      # 追加ここまで

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      # #を外す　ここから
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.string   :current_sign_in_ip
      t.string   :last_sign_in_ip
      # #を外す　ここまで

      # 省略
  end
end

```

---

`/app/models/user.rb`
```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  # 追加　ここから
  has_many :posts, dependent: :destroy
  has_one_attached :avatar
  validates :name, presence: true,  length: { maximum: 20 }, uniqueness: true
  # 追加　ここまで
  
end
```

---

`/app/models/post.rb`
```rb
class Post < ApplicationRecord

    # 追加　ここから
    belongs_to :user
    has_one_attached :image
    validates :tilte, presence: true, length: { maximum: 20 }
    validates :content, length: { maximum: 200 }
    # 追加　ここまで

end
```

---

`/config/initializers/devise.rb`
```rb
  # 省略

  # ==> Configuration for any authentication mechanism
  # Configure which keys are used when authenticating a user. The default is
  # just :email. You can configure it to use [:username, :subdomain], so for
  # authenticating a user, both parameters are required. Remember that those
  # parameters are used only when authenticating and not when retrieving from
  # session. If you need permissions, you should implement that in a before filter.
  # You can also supply a hash where the value is a boolean determining whether
  # or not authentication should be aborted when the value is not present.
  config.authentication_keys = [:name] # emailから変更

  # 省略

```

---


`/app/controllers/application_controller.rb`
```rb
class ApplicationController < ActionController::Base
 
  # 追加　ここから
  before_action :configure_permitted_parameters, if: :devise_controller?

  def after_sign_in_path_for(resource)
    posts_path
  end

  def after_sign_out_path_for(resource)
    user_session_path
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name, :email])
  end
  # 追加　ここまで

end
```

---

`/app/controllers/users/registrations_controller.rb`
```rb
# frozen_string_literal: true

class Users::RegistrationsController < Devise::RegistrationsController
 
 # 省略

  # The path used after sign up.
  # #を外す　ここから
  def after_sign_up_path_for(resource)
    posts_path # 編集
  end
  # #を外す　ここまで

  # The path used after sign up for inactive accounts.
  # def after_inactive_sign_up_path_for(resource)
  #   super(resource)
  # end
end
```

---

`/app/controllers/posts_controller.rb`
```rb
class PostsController < ApplicationController
  before_action :authenticate_user! # 追加
  before_action :set_post, only: %i[ show edit update destroy ]

  # GET /posts or /posts.json
  def index
    @posts = Post.all
  end

  # GET /posts/1 or /posts/1.json
  def show
  end

  # GET /posts/new
  def new
    @post = Post.new
  end

　# 削除　ここから
  # GET /posts/1/edit
  def edit
  end
  # 削除　ここまで

  # POST /posts or /posts.json
  def create
    @post = Post.new(post_params)
    @post.user_id = current_user.id  # 追加

    respond_to do |format|
      if @post.save
        format.html { redirect_to @post, notice: "投稿しました" } # 編集
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # 削除　ここから
  # PATCH/PUT /posts/1 or /posts/1.json
  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to @post, notice: "" }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end
  # 削除　ここまで

  # DELETE /posts/1 or /posts/1.json
  def destroy
    @post.destroy!

    respond_to do |format|
      format.html { redirect_to posts_path, status: :see_other, notice: "投稿を削除しました" } # 編集
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def post_params
      params.require(:post).permit(:title, :comment, :image)
    end
end
```


---

`/app/controllers/mypage_controller.rb`
```rb
class MypagesController < ApplicationController
  before_action :authenticate_user! # 追加

  def show
    @user = User.find(params[:id]) # 追加
  end

  def edit
    # 追加　ここから
    @user = current_user
    if @user.id != current_user.id
      redirect_to mypage_path(current_user)
    end
    # 追加　ここまで
  end

  def update
    # 追加　ここから
    user = User.find(params[:id])
    user.update(user_params)
    redirect_to mypage_path(user.id)
    # 追加　ここまで
  end

  # 追加　ここから
  private

    def user_params
      params.require(:user).permit(:name, :email, :avatar)
    end
  # 追加　ここまで
end
```

---

`/app/views/layouts/application.html.erb`
```html
<!DOCTYPE html>
<html>
  <!-- 省略 -->

  <body>
    <!-- 追加　ここから -->
    <header class="header display-f">
      <% if user_signed_in? %>
        <div class="width-200 text-align-c padding-l-15p">
          <%= link_to "新規投稿", new_post_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "投稿一覧", posts_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "マイページ", mypage_path(current_user.id), class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete }, class: "color-white" %>
        </div>
      <% else %>
        <div class="width-200 text-align-c padding-l-15p">
          <%= link_to "サインアップ", new_user_registration_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "ログイン", new_user_session_path, class: "color-white" %>
        </div>
      <% end %>
    </header>
    <!-- 追加　ここまで -->

    <%= yield %>
  </body>
</html>
```

---

`/app/views/devise/registrations/new.html.erb`

```html
<div class="padding-l-15p"> <!-- 追加 -->
  <h2>サインアップ</h2> <!-- 編集 -->

  <%= form_with model: @user, url: registration_path(resource_name) do |f| %> <!-- 編集 -->
    <%= render "devise/shared/error_messages", resource: resource %>
    
    <!-- 追加　ここから -->
    <div class="field">
      <%= f.label :名前 %><br />
      <%= f.text_field :name, autofocus: true, autocomplete: "name" %>
    </div>
    <!-- 追加　ここまで -->

    <div class="field">
      <%= f.label :メールアドレス %><br /> <!-- 編集 -->
      <%= f.email_field :email, autocomplete: "email" %> <!-- 編集 -->
    </div>

    <div class="field">
      <%= f.label :アイコン画像 %><br /> <!-- 編集 -->
      <%= f.file_field :avatar, autocomplete: "avatar" %> <!-- 編集 -->
    </div>

    <div class="field">
      <%= f.label :パスワード %> <!-- 編集 -->
      <% if @minimum_password_length %>
        <em>(<%= @minimum_password_length %> characters minimum)</em>
      <% end %><br />
      <%= f.password_field :password, autocomplete: "new-password" %>
    </div>

    <div class="field">
      <%= f.label :パスワードの確認 %><br /> <!-- 編集 -->
      <%= f.password_field :password_confirmation, autocomplete: "new-password" %>
    </div>

    <div class="actions">
      <%= f.submit "サインアップ" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>
</div> <!-- 追加 -->
```

---

`/app/views/devise/sessions/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <h2>ログイン</h2> <!-- 編集 -->

  <%= form_with model: @user, url: session_path(resource_name) do |f| %>  <!-- 編集 -->
    <div class="field">
      <%= f.label :名前 %><br /> <!-- 編集 -->
      <%= f.text_field :name, autofocus: true, autocomplete: "name" %> <!-- 編集 -->
    </div>

    <div class="field">
      <%= f.label :パスワード %><br /> <!-- 編集 -->
      <%= f.password_field :password, autocomplete: "current-password" %>
    </div>

    <% if devise_mapping.rememberable? %>
      <div class="field">
        <%= f.check_box :remember_me %>
        <%= f.label :ログイン状態を保持 %> <!-- 編集 -->
      </div>
    <% end %>

    <div class="actions">
      <%= f.submit "ログイン" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>

</div> <!-- 追加 -->
```


---

`/app/views/devise/shared/_links.html.erb`
```html
<%- if controller_name != 'sessions' %>
  <%= link_to "ログイン", new_session_path(resource_name) %><br /> <!-- 編集 --> 
<% end %>

<%- if devise_mapping.registerable? && controller_name != 'registrations' %>
  <%= link_to "サインアップ", new_registration_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<%- if devise_mapping.recoverable? && controller_name != 'passwords' && controller_name != 'registrations' %>
  <%= link_to "パスワードをお忘れですか?", new_password_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<!-- 省略 --> 
```

---

`/app/views/mypages/show.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <h1>マイページ</h1> <!-- 編集 -->

<!-- 追加 ここから -->
  <div>
    
    <p>
      <strong>名前</strong>： <!-- 編集 -->
      <%= @user.name %>
    </p>

    <p>
      <strong>メールアドレス</strong>： <!-- 編集 -->
      <%= @user.email %>
    </p>

    <p>
      <strong class="vertical-align-t">アイコン画像</strong> <!-- 編集 -->
      <!-- 追加　ここから -->
      <% if @user.avatar.attached? %>
        <%= image_tag @user.avatar, size: "300x300" %>
      <% else %>
        <%= image_tag 'NoImage.png', size: "300x300" %>
      <% end %>
      <!-- 追加　ここまで -->
    </p>

  </div>

  <%= link_to "編集", edit_mypage_path(@user) %> <!-- 編集 -->

</div>
<!-- 追加 ここまで -->

```

---

`/app/views/mypages/edit.html.erb`
```html
<!-- 追加　ここから -->
<div class="padding-l-15p">

  <h1>ユーザー情報編集</h1>
  <%= form_with model: @user, url: mypage_path do |form| %>
    <% if @user.errors.any? %>
      <div style="color: red">
        <h2><%= pluralize(@user.errors.count, "error") %> prohibited this user from being saved:</h2>
  
        <ul>
          <% @user.errors.each do |error| %>
            <li><%= error.full_message %></li>
          <% end %>
        </ul>
      </div>
    <% end %>
    
    <div>
      <%= form.label :名前, style: "display: block" %>
      <%= form.text_field :name %>
    </div>
  
    <div>
      <%= form.label :メールアドレス, style: "display: block" %>
      <%= form.email_field :email %>
    </div>
    
    <div>
      <%= form.label :アイコン画像, style: "display: block" %>
      <%= form.file_field :avatar %>
    </div>
    
    <br />

    <div>
      <%= form.submit '更新' %>
    </div>

    <br />

    <div>
      <%= link_to "マイページへ戻る", mypage_path(@user) %>
    </div>
  <% end %>
  
</div>
<!-- 追加　ここまで -->

```

---

`/app/views/posts/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <h1>新規投稿</h1> <!-- 編集 -->

  <%= render "form", post: @post %>

  <br>

  <div>
    <%= link_to "投稿一覧へ戻る", posts_path %> <!-- 編集 -->
  </div>

</div> <!-- 追加 -->
```

---

`/app/views/posts/_form.html.erb`
```html
<%= form_with(model: post) do |form| %>
  <% if post.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>

      <ul>
        <% post.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :タイトル, style: "display: block" %> <!-- 編集 -->
    <%= form.text_field :tilte %>
  </div>

  <div>
    <%= form.label :投稿内容, style: "display: block" %> <!-- 編集 -->
    <%= form.text_area :content %>
  </div>

  <div>
    <%= form.label :画像, style: "display: block" %> <!-- 編集 -->
    <%= form.file_field :image %> <!-- 編集 -->
  </div>

  <br /> <!-- 追加 -->

  <div>
    <%= form.submit '投稿' %> <!-- 編集 -->
  </div>
<% end %>
```

---

`/app/views/posts/index.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <p style="color: green"><%= notice %></p>

  <h1>投稿一覧</h1> <!-- 編集 -->

  <div id="posts">
    <% @posts.each do |post| %>
      <%= render post %> <!-- 削除 -->

      <div id="<%= dom_id post %>">
        <p>
          <strong>タイトル:</strong> <!-- 編集 -->
          <%= post.tilte %>
        </p>

        <p>
          <strong>投稿内容:</strong> <!-- 編集 -->
          <%= post.content %>
        </p>

        <p>
          <strong class="vertical-align-t">画像</strong> <!-- 編集 -->
          <!-- 追加　ここから -->
          <% if post.image.attached? %>
            <%= image_tag post.image, size: "300x300" %>
          <% else %>
            <%= image_tag 'NoImage.png', size: "300x300" %>
          <% end %>
          <!-- 追加　ここまで -->
        </p>

      </div>
      <p>
        <%= link_to "投稿詳細", post %> <!-- 編集 -->
      </p>
    <% end %>
  </div>

  <%= link_to "新規投稿", new_post_path %> <!-- 編集 -->

</div>
```

---

`/app/views/posts/show.html.erb`
```html
<div class="padding-l-15p"> <!-- 編集 -->

  <h1>投稿詳細</h1> <!-- 追加 -->

  <p style="color: green"><%= notice %></p>

  <%= render @post %>

  <div>
    <%= link_to "投稿の編集", edit_post_path(@post) %> | <!-- 編集 -->
    <%= link_to "投稿一覧へ戻る", posts_path %> <!-- 編集 -->

    <%= button_to "投稿削除", @post, method: :delete %> <!-- 編集 -->
  </div>

</div>
```

---

---

`/app/views/posts/_post.html.erb`
```html
<div id="<%= dom_id post %>">
  <p>
    <strong>タイトル:</strong> <!-- 編集 -->
    <%= post.tilte %>
  </p>

  <p>
    <strong>投稿内容:</strong> <!-- 編集 -->
    <%= post.content %>
  </p>

  <p>
    <strong class="vertical-align-t">画像</strong> <!-- 編集 -->
    <!-- 追加　ここから -->
    <% if post.image.attached? %>
      <%= image_tag post.image, size: "600x600" %>
    <% else %>
      <%= image_tag 'NoImage.png', size: "600x600" %>
    <% end %>
    <!-- 追加　ここまで -->
  </p>

</div>
```

---

``
```css
/*
 省略
 */

 /* リンク下線消し */
 a {
    text-decoration: none;
 }

 /* ヘッダー */
 .header {
    width: 100%;
    height: 50px;
    line-height: 50px;
    background: #ffc06e;
 }

 /* 横幅 */
 .width-200 {
    width: 200px;
 }

 .width-600 {
    width: 600px;
 }

 /* 間隔 */
 .padding-l-15p {
    padding-left: 15%;
 }

 /* display */
 .display-f {
    display: flex;
 }

 /* 文字位置　横 */
 .text-align-c {
    text-align: center;
 }

 /* 文字色 */
 .color-white {
    color: #ffffff;
 }

 /* 線 */
 .border {
    border: 1px solid #000000;
 }

 /* 文字位置　高さ */
 .vertical-align-t {
    vertical-align: top;
 }
```

</details>

<br>

### 問題7
<details>

<br>

ターミナル
```sh
rails new search_practice

# gemfileを編集後
rails g devise:install

# Deviseでモデルの作成
rails g devise User

# Postモデルの作成
rails g scaffold Post title:string content:text 

# マイグレーションファイルを編集してから
rails db:migrate

# コントローラーの作成
rails g devise:controllers Users
rails g controller Mypages index show

# ビューの作成
rails g devise:views

```

---

`/Gemfile`
```gemfile
# 省略

gem 'devise'    # 追加
gem 'ransack'   # 追加
```

---

`/db/migrate/xxxxxxxxxxxxxx_devise_create_users.rb`
```rb
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      t.string :name # 追加

      # 省略

      ## Trackable
      # #を外す　ここから
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.string   :current_sign_in_ip
      t.string   :last_sign_in_ip
      # #を外す　ここまで

      # 省略
  end
end
```

---

`/config/routes.rb`
```rb
Rails.application.routes.draw do
  get 'mypages/index' # 削除
  get 'mypages/show'  # 削除
  resources :posts, only: [:index, :new, :create, :show]   # 追加
  devise_for :users

  resources :mypages, only: [:index, show] # 追加

  root to: "posts#index" # 追加

  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # 省略
end
```

---

`/config/initializers/devise.rb`
```rb
  # 省略

  # ==> Configuration for any authentication mechanism
  # Configure which keys are used when authenticating a user. The default is
  # just :email. You can configure it to use [:username, :subdomain], so for
  # authenticating a user, both parameters are required. Remember that those
  # parameters are used only when authenticating and not when retrieving from
  # session. If you need permissions, you should implement that in a before filter.
  # You can also supply a hash where the value is a boolean determining whether
  # or not authentication should be aborted when the value is not present.
  config.authentication_keys = [:name] # emailから変更

  # 省略

```

---

`/app/models/user.rb`
```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  # 追加　ここから
  has_many :posts, dependent: :destroy

  def self.ransackable_attributes(auth_object = nil)
    %w[name email]
  end
  # 追加　ここまで
end
```

---

`/app/models/post.rb`
```rb
class Post < ApplicationRecord
    # 追加　ここから
    belongs_to :user 

    def self.ransackable_attributes(auth_object = nil)
        %w[title content]
    end
    # 追加　ここまで
end
```

---

`/app/controllers/application_controller.rb`
```rb
class ApplicationController < ActionController::Base
  # 追加　ここから
  before_action :configure_permitted_parameters, if: :devise_controller?

  def after_sign_in_path_for(resource)
      mypage_path(resource)
  end

  def after_sign_out_path_for(resource)
      root_path
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name, :email])
  end
  # 追加　ここまで
end
```

---

`/app/controllers/Users/registrations_controllers`
```rb
class Users::RegistrationsController < Devise::RegistrationsController
  # 省略

  # The path used after sign up.
  # #を外す　ここから
  def after_sign_up_path_for(resource)
    super(resource)
  end
  # #を外す　ここから

  # The path used after sign up for inactive accounts.
  # def after_inactive_sign_up_path_for(resource)
  #   super(resource)
  # end
end
```

---

`/app/controllers/posts_controller.rb`
```rb
class PostsController < ApplicationController
　before_action :authenticate_user! # 追加
  before_action :set_post, only: %i[ show destroy ] # 編集

  # GET /posts or /posts.json
  def index
    @q = Post.ransack(q_params) # 追加
    @posts = @q.result          # 編集
  end

  # GET /posts/1 or /posts/1.json
  def show
  end

  # GET /posts/new
  def new
    @post = Post.new
  end

　# 削除　ここから
  # GET /posts/1/edit
  def edit
  end
  # 削除　ここまで

  # POST /posts or /posts.json
  def create
    @post = Post.new(post_params)
    @post.user_id = current_user.id  # 追加

    respond_to do |format|
      if @post.save
        format.html { redirect_to @post, notice: notice: "投稿しました" } # 編集
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

　# 削除　ここから
  # PATCH/PUT /posts/1 or /posts/1.json
  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to @post, notice: "Post was successfully updated." }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end
  # 削除　ここまで

  # DELETE /posts/1 or /posts/1.json
  def destroy
    @post.destroy!

    respond_to do |format|
      format.html { redirect_to posts_path, status: :see_other, notice: "投稿を削除しました" } # 編集
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def post_params
      params.require(:post).permit(:title, :content, :user_id)
    end

    # 追加　ここから
    def q_params
      params.require(:q).permit(:title, :content, :user_id)
    end
    # 追加　ここまで
end
```

---

`/app/controllers/mypages_controller.rb`
```rb
class MypagesController < ApplicationController
  before_action :authenticate_user! # 追加

  def index
    @q = User.ransack params[:q]    # 追加
    @users = @q.result              # 追加
  end

  def show
    @user = User.find(params[:id])  # 追加
  end

  # 追加　ここから
  private

    def user_params
      params.require(:user).permit(:name, :email)
    end

    def q_params
      params.require(:q).permit(:name, :email)
    end
  # 追加　ここまで
end
```

---

`/app/views/layouts/application.html.erb`
```html
<!DOCTYPE html>
<html>
  <!-- 省略 -->

  <body>
    <!-- 追加　ここから -->
    <header class="header display-f">
      <% if user_signed_in? %>
        <div class="width-200 text-align-c padding-l-15p">
          <%= link_to "新規投稿", new_post_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "ユーザー一覧", mypages_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "投稿一覧", posts_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "マイページ", mypage_path(current_user.id), class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete }, class: "color-white" %>
        </div>
      <% else %>
        <div class="width-200 text-align-c padding-l-15p">
          <%= link_to "サインアップ", new_user_registration_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "ログイン", new_user_session_path, class: "color-white" %>
        </div>
      <% end %>
    </header>
    <!-- 追加　ここまで -->

    <%= yield %>
  </body>
</html>
```

---

`/app/views/devise/registrations/new.html.erb`

```html
<div class="padding-l-15p"> <!-- 追加 -->
  <h2>サインアップ</h2> <!-- 編集 -->

  <%= form_with model: @user, url: registration_path(resource_name) do |f| %> <!-- 編集 -->
    <%= render "devise/shared/error_messages", resource: resource %>
    
    <!-- 追加　ここから -->
    <div class="field">
      <%= f.label :名前 %><br />
      <%= f.text_field :name, autofocus: true, autocomplete: "name" %>
    </div>
    <!-- 追加　ここまで -->

    <div class="field">
      <%= f.label :メールアドレス %><br /> <!-- 編集 -->
      <%= f.email_field :email, autocomplete: "email" %> <!-- 編集 -->
    </div>

    <div class="field">
      <%= f.label :パスワード %> <!-- 編集 -->
      <% if @minimum_password_length %>
        <em>(<%= @minimum_password_length %> characters minimum)</em>
      <% end %><br />
      <%= f.password_field :password, autocomplete: "new-password" %>
    </div>

    <div class="field">
      <%= f.label :パスワードの確認 %><br /> <!-- 編集 -->
      <%= f.password_field :password_confirmation, autocomplete: "new-password" %>
    </div>

    <div class="actions">
      <%= f.submit "サインアップ" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>
</div> <!-- 追加 -->
```

---

`/app/views/devise/sessions/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <h2>ログイン</h2> <!-- 編集 -->

  <%= form_with model: @user, url: session_path(resource_name) do |f| %>  <!-- 編集 -->
    <div class="field">
      <%= f.label :名前 %><br /> <!-- 編集 -->
      <%= f.text_field :name, autofocus: true, autocomplete: "name" %> <!-- 編集 -->
    </div>

    <div class="field">
      <%= f.label :パスワード %><br /> <!-- 編集 -->
      <%= f.password_field :password, autocomplete: "current-password" %>
    </div>

    <% if devise_mapping.rememberable? %>
      <div class="field">
        <%= f.check_box :remember_me %>
        <%= f.label :ログイン状態を保持 %> <!-- 編集 -->
      </div>
    <% end %>

    <div class="actions">
      <%= f.submit "ログイン" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>

</div> <!-- 追加 -->
```

---

`/app/views/devise/shared/_links.html.erb`
```html
<%- if controller_name != 'sessions' %>
  <%= link_to "ログイン", new_session_path(resource_name) %><br /> <!-- 編集 --> 
<% end %>

<%- if devise_mapping.registerable? && controller_name != 'registrations' %>
  <%= link_to "サインアップ", new_registration_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<%- if devise_mapping.recoverable? && controller_name != 'passwords' && controller_name != 'registrations' %>
  <%= link_to "パスワードをお忘れですか?", new_password_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<!-- 省略 --> 
```

---

`/app/views/mypages/index.html.erb`
```html
<h1>Mypages#index</h1> <!-- 削除 -->
<p>Find me in app/views/mypages/index.html.erb</p> <!-- 削除 -->

<!-- 追加　ここから -->
<div class="padding-l-15p">
  <p style="color: green"><%= notice %></p>

  <h1>ユーザー一覧</h1>

  <h2>ユーザー検索</h2>
  <%= search_form_for @q, url: mypages_path do |f| %>
    <table>
      <tr>
        <th>名前</th>
        <td><%= f.text_field :name_cont %></td>
      </tr>
    </table>
    <%= f.submit '検索' %>
  <% end %>

  <br>

  <h2>一覧</h2>
  <div id="posts">
    <table class="table">
      <thead>
        <tr>
          <th class="border width-200">名前</th>
          <th class="width-200"></th>
        </tr>
      </thead>

      <tbody>
        <% @users.each do |user| %>         
          <tr>
            <td class="border">
              <%= user.name %>
            </td>
            <td>
              <%= link_to "投稿詳細", mypage_path(user.id) %>
            </td>
          </tr>
        <% end %>
      </tbody>

    </table>
  </div>

</div>
<!-- 追加　ここまで -->
```


`/app/views/mypages/show.html.erb`
```html
<h1>Mypages#show</h1> <!-- 削除 -->
<p>Find me in app/views/mypages/show.html.erb</p> <!-- 削除 -->

<!-- 追加　ここから -->
<div class="padding-l-15p"> 
  <p style="color: green"><%= notice %></p>

  <h2>
    <%= @user.name %>
  </h2>

</div>
<!-- 追加　ここまで -->
```

---

`/app/views/posts/index.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->
  <p style="color: green"><%= notice %></p>

  <h1>投稿一覧</h1> <!-- 編集 -->

　<!-- 追加　ここから -->
  <h2>投稿検索</h2>
  <%= search_form_for @q do |f| %>
    <table>
      <tr>
        <th>タイトル</th>
        <td><%= f.text_field :title_cont %></td>
      </tr>
      <tr>
        <th>投稿内容</th>
        <td><%= f.text_field :content_cont %></td>
      </tr>
    </table>
    <%= f.submit '検索' %>
  <% end %>

  <h2>一覧</h2>
  <!-- 追加　ここまで -->

  <div id="posts">
    <% @posts.each do |post| %>
      <%= render post %>
      <p>
        <%= link_to "投稿詳細", post %> <!-- 編集 -->
      </p>
    <% end %>
  </div>

  <%= link_to "New post", new_post_path %> <!-- 削除 -->
</div>
```

---

`/app/views/posts/show.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <p style="color: green"><%= notice %></p>

  <%= render @post %>

  <div>
    <!-- 追加　ここから -->
    <% if @post.user == current_user %>
      <%= button_to "投稿を削除", @post, method: :delete %>
    <% end %>
    <!-- 追加　ここまで -->

    <!-- 削除　ここから -->
    <%= link_to "Edit this favorite", edit_favorite_path(@favorite) %> |
    <%= link_to "Back to favorites", favorites_path %>

    <%= button_to "Destroy this favorite", @favorite, method: :delete %>
    <!-- 削除　ここまで -->

  </div>

</div>
```

---

`/app/assets/stylesheets/application.css`
```css
/*
 省略
*/

 /* リンク下線消し */
 a {
    text-decoration: none;
 }

 /* ヘッダー */
 .header {
    width: 100%;
    height: 50px;
    line-height: 50px;
    background: #ff9915;
 }

 /* 横幅 */
 .width-200 {
    width: 200px;
 }

 .width-600 {
    width: 600px;
 }

 /* 間隔 */
 .padding-l-15p {
    padding-left: 15%;
 }

  /* display */
 .display-f {
    display: flex;
 }

 /* 文字位置　横 */
 .text-align-c {
    text-align: center;
 }

 /* 文字色 */
 .color-white {
    color: #ffffff;
 }

 /* 線 */
 .border {
    border: 1px solid #000000;
 }

 /* テーブル */
 .table {
    border-collapse: collapse;
 }

```

</details>

<br>

---

### 問題8
<details>

ターミナル
```sh
rails new no_gem_search_practice

# Gemfileを編集後
rails g devise:install

# Deviseでモデルの作成
rails g devise User

# Cookingモデル、Gereモデル、Relationモデルの作成
rails g scaffold Cooking dish_name:string content:text user:references genre_id:integer
rails g scaffold Genre genre_name:string

# マイグレーションファイルを編集してから
rails db:migrate

# コントローラーの作成
rails g devise:controllers Users

# ビューの作成
rails g devise:views
```

---

`/Gemfile`
```gemfile
# 省略

gem 'devise'    # 追加
```

---

`/db/migrate/xxxxxxxxxxxxxx_devise_create_users.rb`
```rb
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      t.string :name # 追加

      # 省略

      ## Trackable
      # #を外す　ここから
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.string   :current_sign_in_ip
      t.string   :last_sign_in_ip
      # #を外す　ここまで

      # 省略
  end
end
```

---

`/app/models/user.rb`
```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  has_many :cookings, dependent: :destroy # 追加
  has_many :genres                        # 追加

end
```

---

`/app/models/cooking.rb`
```rb
class Cooking < ApplicationRecord
    # 追加　ここから
    belongs_to :user
    belongs_to :genre
    # 追加　ここまで
end
```

---

`/app/models/genre.rb`
```rb
class Genre < ApplicationRecord
    has_many :cookings # 追加
end
```

---

`/config/routes.rb`
```rb
Rails.application.routes.draw do
  resources :genres
  resources :cookings
  devise_for :users

  root to: "cookings#index" # 追加
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # 省略
end

```

`/app/controllers/application_controller.rb`
```rb
class ApplicationController < ActionController::Base
  # 追加　ここから
  before_action :configure_permitted_parameters, if: :devise_controller?

  def after_sign_in_path_for(resource)
      cookings_path
  end

  def after_sign_out_path_for(resource)
      root_path
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:email])
  end
  # 追加　ここまで
end
```

---

`/app/controllers/Users/registrations_controllers`
```rb
class Users::RegistrationsController < Devise::RegistrationsController
  # 省略

  # The path used after sign up.
  # #を外す　ここから
  def after_sign_up_path_for(resource)
    cookings_path # 編集
  end
  # #を外す　ここから

  # The path used after sign up for inactive accounts.
  # def after_inactive_sign_up_path_for(resource)
  #   super(resource)
  # end
end
```

---

`/app/controllers/cookings_controller.rb`
```rb
class CookingsController < ApplicationController
  before_action :authenticate_user! # 追加
  before_action :set_cooking, only: %i[ show edit update destroy ]

  # GET /cookings or /cookings.json
  def index
    # @cookings = Cooking.all # 削除

    # 追加　ここから
    return @cookings = Cooking.all if params[:dish_name].blank? && params[:genre].blank?

    if params[:dish_name].present?
      @cookings = Cooking.where(["dish_name like ?", "%#{params[:name]}%"])
    else
      @cookings = @cookings.joins(:genres).where("genres.genre_name = ?", params[:genre]) if params[:genre].present?
    end
    # 追加　ここまで
  end

  # GET /cookings/1 or /cookings/1.json
  def show
  end

  # GET /cookings/new
  def new
    @cooking = Cooking.new
  end

  # GET /cookings/1/edit
  def edit
  end

  # POST /cookings or /cookings.json
  def create
    @cooking = Cooking.new(cooking_params)
    @cooking.user_id = current_user.id # 追加

    respond_to do |format|
      if @cooking.save
        format.html { redirect_to @cooking, notice: "料理を新たに登録しました" } # 編集
        format.json { render :show, status: :created, location: @cooking }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @cooking.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /cookings/1 or /cookings/1.json
  def update
    respond_to do |format|
      if @cooking.update(cooking_params)
        format.html { redirect_to @cooking, notice: "Cooking was successfully updated." }
        format.json { render :show, status: :ok, location: @cooking }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @cooking.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /cookings/1 or /cookings/1.json
  def destroy
    @cooking.destroy!

    respond_to do |format|
      format.html { redirect_to cookings_path, status: :see_other, notice: "Cooking was successfully destroyed." }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_cooking
      @cooking = Cooking.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def cooking_params
      params.require(:cooking).permit(:dish_name, :content, :genre_id) # 追加
    end
end
```

---

`/app/views/application.html.erb`
```html
<!DOCTYPE html>
<html>
  <!-- 省略 -->

  <body>

    <!-- 追加　ここから -->
    <header class="header display-f">
      <% if user_signed_in? %>
        <div class="width-200 text-align-c padding-l-15p">
          <%= link_to "新規料理", new_cooking_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "新規ジャンル", new_genre_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "料理一覧", posts_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete }, class: "color-white" %>
        </div>
      <% else %>
        <div class="width-200 text-align-c padding-l-15p">
          <%= link_to "サインアップ", new_user_registration_path, class: "color-white" %>
        </div>
        <div class="width-200 text-align-c">
          <%= link_to "ログイン", new_user_session_path, class: "color-white" %>
        </div>
      <% end %>
    </header>
    <!-- 追加　ここまで -->

    <%= yield %>
  </body>
</html>
```

---

`/app/views/devise/registrations/new.html.erb`

```html
<div class="padding-l-15p"> <!-- 追加 -->
  <h2>サインアップ</h2> <!-- 編集 -->

  <%= form_with model: @user, url: registration_path(resource_name) do |f| %> <!-- 編集 -->
    <%= render "devise/shared/error_messages", resource: resource %>
    
    <div class="field">
      <%= f.label :メールアドレス %><br /> <!-- 編集 -->
      <%= f.email_field :email, autocomplete: "email" %> <!-- 編集 -->
    </div>

    <div class="field">
      <%= f.label :パスワード %> <!-- 編集 -->
      <% if @minimum_password_length %>
        <em>(<%= @minimum_password_length %> 文字以上)</em>
      <% end %><br />
      <%= f.password_field :password, autocomplete: "new-password" %>
    </div>

    <div class="field">
      <%= f.label :パスワードの確認 %><br /> <!-- 編集 -->
      <%= f.password_field :password_confirmation, autocomplete: "new-password" %>
    </div>

    <div class="actions">
      <%= f.submit "サインアップ" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>
</div> <!-- 追加 -->
```

---

`/app/views/devise/sessions/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->

  <h2>ログイン</h2> <!-- 編集 -->

  <%= form_with model: @user, url: session_path(resource_name) do |f| %>  <!-- 編集 -->
    <div class="field">
      <%= f.label :メールアドレス %><br /> <!-- 編集 -->
      <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
    </div>

    <div class="field">
      <%= f.label :パスワード %><br /> <!-- 編集 -->
      <%= f.password_field :password, autocomplete: "current-password" %>
    </div>

    <% if devise_mapping.rememberable? %>
      <div class="field">
        <%= f.check_box :remember_me %>
        <%= f.label :ログイン状態を保持 %> <!-- 編集 -->
      </div>
    <% end %>

    <div class="actions">
      <%= f.submit "ログイン" %> <!-- 編集 -->
    </div>
  <% end %>

  <%= render "devise/shared/links" %>

</div> <!-- 追加 -->
```

---

`/app/views/devise/shared/_links.html.erb`
```html
<%- if controller_name != 'sessions' %>
  <%= link_to "ログイン", new_session_path(resource_name) %><br /> <!-- 編集 --> 
<% end %>

<%- if devise_mapping.registerable? && controller_name != 'registrations' %>
  <%= link_to "サインアップ", new_registration_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<%- if devise_mapping.recoverable? && controller_name != 'passwords' && controller_name != 'registrations' %>
  <%= link_to "パスワードをお忘れですか?", new_password_path(resource_name) %><br /> <!-- 編集 -->
<% end %>

<!-- 省略 --> 
```

---

`/app/views/genres/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->
  <h1>新規ジャンル</h1> <!-- 編集 -->

  <%= render "form", genre: @genre %>

</div> <!-- 追加 -->

<!-- 削除　ここから -->
<br>

<div>
  <%= link_to "Back to genres", genres_path %>
</div>
<!-- 削除　ここまで -->
```

---

`/app/views/genres/_form.html.erb`
```html
<%= form_with(model: genre) do |form| %>
  <% if genre.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(genre.errors.count, "error") %> prohibited this genre from being saved:</h2>

      <ul>
        <% genre.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :ジャンル名, style: "display: block" %> <!-- 編集 -->
    <%= form.text_field :genre_name %>
  </div>

  <br> <!-- 追加 -->

  <div>
    <%= form.submit '登録' %> <!-- 編集 -->
  </div>
<% end %>
```

---

`/app/views/cookings/index.html.erb`
```html
<p style="color: green"><%= notice %></p>

<h1>料理一覧</h1> <!-- 編集 -->

<!-- 追加　ここから -->
<h2>検索</h2>
<%= form_with url: cookings_path, method: :get do |f| %>
  <%= f.text_field :title, placeholder: "料理名で検索" %>
  <%= f.collection_select(:genre_id, Genre.all, :id, :genre_name, include_blank: "  " ) %>
  <%= f.submit "検索" %>
<% end %>

<br>

<h2>一覧</h2>
<!-- 追加　ここまで -->

<div id="cookings">

<!-- 追加　ここから -->
<table class="table">
  <thead>
    <tr>
      <th class="border width-200">料理名</th>
      <th class="border width-200">ジャンル</th>
      <th class="border width-600">説明</th>
      <th></th>
    </tr>
  </thead>
<!-- 追加　ここまで -->

  <% @cookings.each do |cooking| %>
    <%= render cooking %> <!-- 削除 -->

  　<!-- 追加　ここから -->
    <tr>
      <td class="border">
        <%= cooking.dish_name %>
      </td>
      <td class="border">
        <%= cooking.genre.genre_name %>
      </td>
      <td class="border">
        <%= cooking.content %>
      </td>
      <!-- 追加　ここまで -->
      <td> <!-- 編集 -->
        <%= link_to "料理詳細", cooking %> <!-- 編集 -->
      </td> <!-- 編集 -->
    </tr> <!-- 追加 -->
  <% end %>
</div>

<%= link_to "New cooking", new_cooking_path %> <!-- 削除 -->

```

---

`/app/views/cookings/new.html.erb`
```html
<div class="padding-l-15p"> <!-- 追加 -->
  <h1>新規料理</h1> <!-- 編集 -->

  <%= render "form", cooking: @cooking %>

</div> <!-- 追加 -->

<!-- 削除　ここから -->
<br>

<div>
  <%= link_to "Back to cookings", cookings_path %>
</div>
<!-- 削除　ここから -->
```

---

`/app/views/cookings/_form.html.erb`
```html
<%= form_with(model: cooking) do |form| %>
  <% if cooking.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(cooking.errors.count, "error") %> prohibited this cooking from being saved:</h2>

      <ul>
        <% cooking.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :料理名, style: "display: block" %> <!-- 編集 -->
    <%= form.text_field :dish_name %>
  </div>

  <div>
    <%= form.label :説明, style: "display: block" %> <!-- 編集 -->
    <%= form.text_area :content %>
  </div>

  <!-- 追加　ここから -->
  <div>
    <%= form.label :genre_ids, "ジャンル" %><br>
    <%= form.collection_select(:genre_id, Genre.all, :id, :genre_name, include_blank: "選択して下さい" ) %>
  </div>

  <br>
  <!-- 追加　ここまで -->

  <div>
    <%= form.submit '登録' %> <!-- 編集 -->
  </div>
<% end %>
```

---

`/app/views/cookings/show.html.erb`
```html
<div class="padding-l-15p">
  <p style="color: green"><%= notice %></p>

  <h1>料理詳細</h1> <!-- 追加 -->
  
  <%= render @cooking %>

</div>

<!-- 削除　ここから -->
<div>
  <%= link_to "Edit this cooking", edit_cooking_path(@cooking) %> |
  <%= link_to "Back to cookings", cookings_path %>

  <%= button_to "Destroy this cooking", @cooking, method: :delete %>
</div>
<!-- 削除　ここまで -->
```

---

`/app/views/cookings/_cooking.html.erb`
```html
<div id="<%= dom_id cooking %>">
  <p>
    <strong>料理名:</strong> <!-- 編集 -->
    <%= cooking.dish_name %>
  </p>

  <!-- 追加　ここから -->
  <p>
    <strong>ジャンル:</strong>
    <%= cooking.genre.genre_name %>
  </p>
  <!-- 追加　ここまで -->

  <p>
    <strong>説明:</strong> <!-- 編集 -->
    <%= cooking.content %>
  </p>

</div>

```

---

`/app/assets/stylesheets/application.css`
```css
/*
 省略
 */

 /* リンク下線消し */
 a {
    text-decoration: none;
 }

 /* ヘッダー */
 .header {
    width: 100%;
    height: 50px;
    line-height: 50px;
    background: #ff9915;
 }

 /* 横幅 */
 .width-200 {
    width: 200px;
 }

 .width-600 {
    width: 600px;
 }

 /* 間隔 */
 .padding-l-15p {
    padding-left: 15%;
 }

  /* display */
 .display-f {
    display: flex;
 }

 /* 文字位置　横 */
 .text-align-c {
    text-align: center;
 }

 /* 文字色 */
 .color-white {
    color: #ffffff;
 }

 /* 線 */
 .border {
    border: 1px solid #000000;
 }

 /* テーブル */
 .table {
    border-collapse: collapse;
 }

```

</details>