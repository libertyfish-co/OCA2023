## Ruby on Rails：練習問題_解答

#### この解答は編集箇所のみ記述しております。

### (1)

ターミナル
```sh
rails new article_practice
cd article_practice
rails g model Article title:string content:text
rails db:migrate
rails g controller articles index new show edit update destroy
```

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

<br>

### (2)
ターミナル
```sh
rails new people_lists
rails g scaffold Member name:string height:float weight:float
rails db:migrate
```

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

<br>

### (3)
ターミナル
```sh
rails new bcrypt_practice
rails bcrypt_practice

# gemfile を編集してから
rails bundle install

# コントローラー作成
rails g controller Users top
rails g controller Sessions new 
rails g controller Pages my_page

# Userモデル作成
rails g User name:string email:string password_digest:string
rails db:migrate
```

`/gemfile`
```rb
# 省略

# Use Active Model has_secure_password [https://guides.rubyonrails.org/active_model_basics.html#securepassword]
gem "bcrypt", "~> 3.1.7" #を外す

# 省略
```

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

`/app/models/user.rb`
```rb
class User < ApplicationRecord
  has_secure_password # 追加
end
```

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

`/app/views/users/top.html.erb`
```html
<h1>Top画面</h1>

<%= link_to 'サインアップ', signup_path %> |
<%= link_to 'ログイン', new_session_path %>
```

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