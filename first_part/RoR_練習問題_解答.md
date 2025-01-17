## Ruby on Rails：練習問題_解答

#### この解答は編集箇所のみ記述しております。

### (1)
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

### (2)
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

### (3)
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

### (4)
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

### (5)
<details>

<br>

ターミナル
```sh
rails new sns_site
rails cd sns_site

# gemfile を編集してから
bundle install
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
rails g controller Mypages show edit update

# ビューの作成
rails g devise:views
touch app/views/top/top.html.erb
touch app/views/mypage/show.html.erb
touch app/views/mypage/edit.html.erb
```

---

`Gemfile`
```gemfile
# 省略

gem 'devise' # 追加
```

---

`xxxxxxxxxxxxxx_devise_create_users.rb`
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

