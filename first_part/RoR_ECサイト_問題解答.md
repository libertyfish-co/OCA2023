## Ruby on Rails：ECサイト　問題_解答

### (1) 

`config/application.rb` の中に次の1行を追加することで設定できます。  
日本国内では `Tokyo` の他に `Osaka` と `Sapporo` も設定できます。

```rb
module EcSite
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 7.1
    
    config.time_zone = `Tokyo` # 追加

    # Configuration for the application, engines, and railties goes here.
    #
    # These settings can be overridden in specific environments using the files
    # in config/environments, which are processed later.
    #
    # config.time_zone = "Central Time (US & Canada)"
    # config.eager_load_paths << Rails.root.join("extras")
  end
end
```

<br>  
  
### (2) 

`app/controllers/tags_controller.rb`
```rb
class TagsController < ApplicationController

  # 省略
 
  # GET /tags/1 or /tags/1.json #削除
  def show # 削除
  end # 削除

  # POST /tags or /tags.json
  def create
    @tag = Tag.new(tag_params)

    respond_to do |format|
      if @tag.save
        format.html { redirect_to tags_url, notice: "Tag was successfully created." } # 変更
        format.json { render :show, status: :created, location: @tag }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @tag.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /tags/1 or /tags/1.json
  def update
    respond_to do |format|
      if @tag.update(tag_params)
        format.html { redirect_to tags_url, notice: "Tag was successfully updated." } # 変更
        format.json { render :show, status: :ok, location: @tag }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @tag.errors, status: :unprocessable_entity }
      end
    end
  end

  # 省略

end
```

`/app/views/tags/show.html.erb `ファイルを削除  

`/app/views/tags/index.html.erb `
```html
<p style="color: green"><%= notice %></p>

<h1>Tags</h1>

<div id="tags">
  <% @tags.each do |tag| %>
    <%= render tag %>
    <p>
      <%= link_to "Show this tag", tag %>  # 削除
      
      # 追加 ここから
      <%= link_to "Edit this tag", edit_tag_path(tag) %>
      <%= button_to "Destroy this tag", tag, method: :delete %>
      # 追加 ここまで

    </p>
  <% end %>
</div>

<%= link_to "New tag", new_tag_path %>
```

`config/routes.rb`
```rb
Rails.application.routes.draw do
  resources :tags, except: :show # 変更
  resources :books
  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
  
  # 省略

end
```

`/test/controllers/tags_controller_test.rb`
```rb
require "test_helper"

class TagsControllerTest < ActionDispatch::IntegrationTest

  # 省略

　# 削除　ここから
  test "should show tag" do
	  get tag_url(@tag)
	  assert_response :success
  end 
  # 削除　ここまで

  # 省略

end
```

<br>

### (3)

ターミナル
```sh
rails new article_sample
cd article_sample
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

<table class="table w-800">
  <thead>
    <tr>
      <th class="border ta-center w-300">タイトル</th>
      <th class="border ta-center w-500">投稿内容</th>
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

 .w-300 {
    width: 300px;
 }

 .w-500 {
    width: 500px;
 }

 .w-800 {
    width: 800px;
 }

 .table {
    border-collapse: collapse;
 }

 .border {
    border: 1px solid #000;
 }

 .ta-center {
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

### (4)

ターミナル
```sh
rails g scaffold Tagging book:references tag:references
rails db:migrate
```

`/app/controllers/books_controller.rb`
```rb
class BooksController < ApplicationController

  省略

  # Only allow a list of trusted parameters through.
  def book_params
    params.require(:book).permit(:book_name, :author_name, :issue_date, :product_display, :price, :status, tag_ids: []) # 追加
  end
end
```

`app/views/books/_form.html.erb`
```html
<%= form_with(model: book) do |form| %>

  省略

  <div>
    <%= form.label :Status, style: "display: block" %>
    <%= form.text_field :status %>
  </div>

  <!-- 追加　ここから -->
  <div>
	  <%= form.collection_check_boxes :tag_ids, Tag.all, :id, :tag_name %>
  </div>
  <!-- 追加　ここまで -->
 
  <div class="actions">
	  <%= form.submit %>
  </div>
<% end %>
```

`app/views/books/show.html.erb`
```html
<p style="color: green"><%= notice %></p>

<%= render @book %>

<!-- 追加　ここから -->
<div>
  <p>
    <strong>Tag:</strong>
    <%= @book.tags.map(&:tag_name).join(', ') %>
  </p>
</div>
<!-- 追加　ここまで -->

<div>
  <%= link_to "Edit this book", edit_book_path(@book) %> |
  <%= link_to "Back to books", books_path %>

  <%= button_to "Destroy this book", @book, method: :delete %>
</div>
```

<br>

### (5)

ターミナル
```sh
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

### (6)


<br>

### (7)


<br>

### (8)

`Gemfile`
```rb
gem devise
```

```
$ rails g devise:install
$ rails g devise User
```

`user.rb`
```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :recoverable, and :omniauthable
  devise :database_authenticatable, :registerable,
         :rememberable, :trackable, :validatable
end
```

`db/migrate/xxxxxxxxxxxxxx_devise_create_users`
```rb
create_table :users do |t|
  ## Database authenticatable
  t.string :email,          	null: false, default: ""
  t.string :encrypted_password, null: false, default: ""

  ## Recoverable
  t.string   :reset_password_token
  t.datetime :reset_password_sent_at

  ## Rememberable
  t.datetime :remember_created_at

  ## Trackable
  t.integer  :sign_in_count, default: 0, null: false # 変更
  t.datetime :current_sign_in_at # 変更
  t.datetime :last_sign_in_at # 変更
  t.string   :current_sign_in_ip # 変更
  t.string   :last_sign_in_ip # 変更

  ## Confirmable
  # t.string   :confirmation_token
  # t.datetime :confirmed_at
  # t.datetime :confirmation_sent_at
  # t.string   :unconfirmed_email # Only if using reconfirmable
end
```

```
$ rails db:migrate
```

`config/routes.rb`
```rb
get :mypage, to: 'mypage#index' # 追加
```

`app/controllers/mypage_controller.rb`
```rb
class MypageController < ApplicationController
  def index # 追加
  end # 追加
end
```

`app/controllers/mypage_controller.rb`

```rb
class MypageController < ApplicationController
  before_action :authenticate_user! # 追加

  def index
  end
end
```

`app/views/mypage/index.html.erb` 最終形態
```erb
<h1>マイページ</h1>
<p>マイページ<p>
<p>あなたのメールアドレスは<%= current_user.email %>です</p>
<p><%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete } %></p>
```
```
$ rails generate devise:views
```

`app/views/devise/sessions/new.html.erb`
```erb
<h2>ログイン</h2> # 変更

<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
  <div class="field">
  	<%= f.label :email %><br />
	  <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>
<% end %>
```

### (9)


<br>

### (10)
#### ～コントローラーの作成～

```
$ rails g controller products
$ rails g controller orders
```

#### ～モデルの作成～
```
$ rails g model order book_id:integer count:integer address:text
```

#### ～管理者以外の一覧と詳細作成～
`config/routes.rb`
```rb
resources :products, only: [:index, :show]
```

`app/controllers/products_controller.rb`
```rb
def index
  @books = Book.all
end

def show
  @book = Book.find(params[:id])
end
```

`app/views/products/index.html.erb → 新規作成`
```erb
<p id="notice"><%= notice %></p>

<h1>Products</h1>

<table>
  <thead>
    <tr>
      <th>Title</th>
      <th>Author</th>
      <th>Published on</th>
      <th>Showing</th>
      <th>Price</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @books.each do |book| %>
      <tr>
        <td><%= book.title %></td>
        <td><%= book.author %></td>
        <td><%= book.published_on %></td>
        <td><%= book.showing %></td>
        <td><%= book.price %></td>
        <td><%= link_to 'Show', product_path(book) %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

`app/views/products/show.html.erb → 新規作成`
```erb
<p id="notice"><%= notice %></p>

<p>
  <strong>Title:</strong>
  <%= @book.title %>
</p>

<p>
  <strong>Author:</strong>
  <%= @book.author %>
</p>

<p>
  <strong>Published on:</strong>
  <%= @book.published_on %>
</p>

<p>
  <strong>Showing:</strong>
  <%= @book.showing %>
</p>

<p>
  <strong>Price:</strong>
  <%= @book.price %>
</p>

  <p>
    <strong>タグ</strong>
    <%= @book.tags.map(&:name).join(', ') %>
  </p>

<%= link_to 'Order', new_order_path(@book) %>
<%= link_to 'Back', products_path %>
```

#### ～商品詳細画面に注文ボタンを作成～

`config/routes.rb`
```rb
#追加
resources :orders, only: [:new, :create] do
  collection do
    get :confirm
    get :complete
  end
end
```

`app/controllers/orders_controller.rb`
```rb
class OrdersController < ApplicationController
  def new
    @order = Order.new
    @book = Book.find(params[:format])
  end

  def confirm
    @order = Order.new(order_params)
    @book = Book.find(order_params[:book_id])
  end

  def create
    @order = Order.new(order_params)
    if @order.save
      redirect_to complete_orders_path
    else
      render "confirm"
    end
  end

  def complete
  end

  private

  def order_params
    params.require(:order).permit(:count, :address, :book_id)
  end  
end
```

`app/views/orders/new.html.erb → 新規作成`

```erb
<%= form_with(model: @order, local: true, url: confirm_orders_path, method: :get) do |form| %>
  <% if @order.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@order.errors.count, "error") %> prohibited this book from being saved:</h2>

      <ul>
        <% @order.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  商品名：<%= @book.title %>

  <div class="field">
    <%= form.label :count %>
    <%= form.text_field :count %>
  </div>

  <div class="field">
    <%= form.label :address %>
    <%= form.text_field :address %>
  </div>

  <div class=”field”>
    <%= form.hidden_field :book_id, value: @book.id %>
  </div>

  <div class="actions">
   <%= form.submit '次へ進む' %>
  </div>
<% end %>
```

`app/views/orders/confirm.html.erb → 新規作成`
```erb
<h1>Confirm New Article</h1>

<%= form_with(model: @order, local: true) do |form| %>
  <% if @order.errors.any? %>
      <div id="error_explanation">
        <h2><%= pluralize(@order.errors.count, "error") %> prohibited this article from being saved:</h2>

        <ul>
          <% @order.errors.full_messages.each do |message| %>
              <li><%= message %></li>
          <% end %>
        </ul>
      </div>
  <% end %>

  <div class="field">
    <p><%= form.label :book_id %></p>
    <%= @book.title %>
    <%= form.hidden_field :book_id, value: @book.id %>
  </div>

  <div class="field">
    <p><%= form.label :count %></p>
    <%= @order.count %>
    <%= form.hidden_field :count %>
  </div>

  <div class="field">
    <p><%= form.label :address %></p>
    <%= @order.address %>
    <%= form.hidden_field :address %>
  </div>

  <div class="actions">
    <%= form.submit '注文' %>
  </div>
<% end %>
```

`app/views/orders/complete.html.erb → 新規作成`
```erb
<p>注文が完了しました</p>
```

<br>

### (11)


<br>

### (12)


<br>

### (13)

#### ～セッション管理テーブルの準備～

～Cartモデルとコントローラーの作成～
```
$ rails g model cart
$ rails g controller carts
```

#### ～LineItemテーブルの作成～
```
$ rails g model LineItem book:references cart:references quantity:integer
```

上記でできたマイグレーションファイルに追記しrails db:migrateを実行
```rb
class CreateLineItems < ActiveRecord::Migration[6.1]
  def change
    create_table :line_items do |t|
      t.references :book, null: false, foreign_key: true
      t.references :cart, null: false, foreign_key: true
      t.integer :quantity, null: false, default: 3 #null以降を追記

      t.timestamps
    end
  end
end
```

#### ～アソシエーションの定義～
`app/model/book.rb`
```rb
has_many :line_items #追記
```

`app/model/cart.rb`
```rb
has_many :line_items
has_many :books,through: :line_items #追記
```

`app/controller/application.rb`
```rb
# 以下全て追記
private

def current_cart
  @current_cart = Cart.find_by(id: session[:cart_id])
  @current_cart = Cart.create unless @current_cart
  session[:cart_id] = @current_cart.id
  @current_cart
end
```


`app/model/cart.rb`
```rb
# 以下追記
def add_product(book_id)
  line_items.find_or_initialize_by(book_id: book_id)
end
```

```
$ rails g controller line_items
```

`app/controller/line_items_controller.rb`
```rb
# 以下追記
def create
  cart = current_cart
  book  = Book.find(params[:book_id])
  @line_item = cart.add_product(book.id)

  respond_to do |format|
    if @line_item.save
      format.html { redirect_to products_path }
    else
      format.html { redirect_to products_index_url, notice: 'Unprocessable entity.' }
    end
  end
end
```

`config/routes.rb`
```rb
resources :line_items, only: :create
```

`app/view/product/show.html.erb`
```diff
-- <%= link_to 'Order', new_order_path(@book) %>
++ <%= link_to '買い物かごに入れる', line_items_path(book_id: @book.id), method: :post %>
```

ここまででline_iltesに保存ができているはず

`app/model/line_item.rb`
```rb
# 以下追記
def add_product(book_id)
  line_items.find_or_initialize_by(book_id: book_id)
end

def add_product(book_id)
  line_items.find_or_initialize_by(book_id: book_id)
end
```

`app/model/cart.rb`
```rb
def total_price
  line_items.to_a.sum {|item| item.total_price}
end

def total_number
  line_items.to_a.sum {|item| item.quantity}
end
```

`config/routes.rb`
```rb
 resources :carts, only: :show
```

`app/controller/carts_controller.rb`
```rb
  # 以下追記
 before_action :current_cart
  def show
    @line_items = @current_cart.line_items
  end
```
`app\views\products\index.html.erb` → 新規作成
```erb
# 一番下に追記
# 画面に遷移するためのリンクを作成しておく
<% if session[:cart_id] %>
  <%= link_to '買い物かごを見る', cart_path(session[:cart_id]) %>
<% end %>
```

`app\views\carts\show.html.erb` → 新規作成
```erb
<% @line_items.each do |line_item| %>
  <% book = line_item.book %>
  <div>
    <p><%= book.title %></p>
    <p><%= line_item.quantity %>冊</p>
    <p>
      <%= line_item.total_price %>
      <span>円 (税込)</span>
    </p>
  </div>
<% end %>

<div>
  <p>注文数: <%= @current_cart.total_number %><span>冊</span></p>
  <p>
    合計金額:<%= @current_cart.total_price %>
    <span>円 (税込)</span>
  </p>
</div>

<% if @line_items.count == 0 %>
  買い物かごには何も入っていません。
<% end %>
```

#### ～買い物かごを空にする実装～
`config/routes.rb`
```rb
 resources :carts, only: [:show, :destroy]
```

`app/controller/carts_controller.rb`
```rb
  # 以下追記
  def destroy
    session[:cart_id] = nil
    redirect_to products_path
  end
```

`app/views/carts/show.html.erb`
``` 
<% if @line_items.count == 0 %>
  買い物かごには何も入っていません。
<% else %> # 追加
  <%= link_to '買い物かごを空にする', @cart, data: { "turbo-method": :delete, "turbo-confirm": '買い物かごの中身をすべて削除してもいいですか？' } %> > # 追加
<% end %>  
```

<br>

### (14)

#### ～モデル作成～
```
$ rails g model OrderDetail book:references order:references quantity:integer
```

上記で作成したモデルのマイグレーションファイルに以下を追記
```rb
class CreateOrderDetails < ActiveRecord::Migration[6.1]
  def change
    create_table :order_details do |t|
      t.references :book, null: false, foreign_key: true
      t.references :order, null: false, foreign_key: true
      t.integer :quantity, null: false, default: 0 #オプションの追加
      t.timestamps
    end
  end
end
```

#### ～アソシエーション設定～  
`app\models\book.rb`
```rb
   has_many :order_details, dependent: :destroy #追記
   has_many :orders, through: :order_details # 追記
```

`app\models\order.rb`
```rb
   has_many :order_details, dependent: :destroy #追記
   has_many :books, through: :order_details # 追記
```

`app\models\order_detail.rb`
```rb
class OrderDetail < ApplicationRecord
  belongs_to :book
  belongs_to :order

  def self.create_items(order,line_items)
    line_items.each do |item|
      OrderDetail.create!(order_id: order.id, book_id: item.book_id, quantity: item.quantity)
      LineItem.find(item.id).delete
    end
  end
end
```

`app/views/carts/show.html.erb`
```erb
<% if @line_items.count == 0 %> 
  買い物かごには何も入っていません。
<% else %> 
  <%= link_to '購入する', new_order_path %> # 追記
  <%= link_to '買い物かごを空にする', @cart, data: {"turbo-method": :delete, 
    "turbo-confirm": '買い物かごの中身をすべて削除してもいいですか？' } %> 
<% end %> 
```

`app/models/order_detail.rb`  
以下を追記
```rb
  def total_price
    book.price * quantity
  end
```

`app/models/order.rb`
```rb
class Order < ApplicationRecord
  has_many :order_details, dependent: :destroy
  def total_price # 追加
    order_details.sum { |order| order.total_price } # 追加
  end # 追加
  def total_number # 追加
    order_details.sum { |order| order.quantity } # 追加
  end # 追加
end
```

`app\controllers\orders_controller.rb`
```
 def new
  @order = Order.new
  @books = @current_cart.books # 変更
 end
```

`app\views\orders\new.html.erb`
```erb
<%= form_with(model: @order, local: true, url: confirm_orders_path, method: :get) do |form| %>
  <% if @order.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@order.errors.count, "error") %> prohibited this book from being saved:</h2>
      <ul>
        <% @order.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

<% @books.each do |book|%>
  商品名：<%= book.title %>

  <div class="field">
    <%= form.label :count %>
    <%= form.text_field :count, name: "counts[]", value: LineItem.find_by(cart_id: @current_cart, book_id: book).quantity %>
  </div>

<div class="field">
    <%= form.hidden_field :book_id, value: book.id, name: 'book_ids[]' %>
  </div>

<% end %>

  <div class="field">
    <%= form.label :address %>
    <%= form.text_field :address %>
  </div>

  <div class="actions">
    <%= form.submit '次へ' %>
  </div>
<% end %>
```

`app/controllers/orders_controller.rb`
```rb
class OrdersController < ApplicationController
  def confirm
    @order = Order.new(order_params)
    @books = Book.where(order_params[:book_ids]) # 編集
    @line_items = @current_cart.line_items # 追加
  end
end
```

`app\views\orders\confirm.html.erb`
```erb
<%= form_with(model: @order, local: true) do |form| %>
    <% if @order.errors.any? %>
        <div id="error_explanation">
          <h2><%= pluralize(@order.errors.count, "error") %> prohibited this article from being saved:</h2>

          <ul>
            <% @order.errors.full_messages.each do |message| %>
                <li><%= message %></li>
            <% end %>
          </ul>
        </div>
    <% end %>

<%# @books.each do |book| %>
<!--    <div class="field"> -->
      <%#= form.hidden_field :book_id, value: book.id %>
<!--    </div>-->
<%# end %>

    <div class="field">
      <%= form.hidden_field :count, value: @current_cart.total_number %>
    </div>

    <% @line_items.each do |line_item| %>
      <% book = line_item.book %>
      <div>
        <p><%= book.title %></p>
        <p><%= line_item.quantity %>冊</p>
        <p>
          <%= line_item.total_price %>
          <span>円 (税込)</span>
        </p>
      </div>
    <% end %>
    
    <div>
      <p>合計 <%= @current_cart.total_number %><span>冊</span></p>
      <p>
        <%= @current_cart.total_price %>
        <span>円 (税込)</span>
      </p>
    </div>

    <div class="field">
      <p><%= form.label :address %></p>
      <%= @order.address %>
      <%= form.hidden_field :address %>
    </div>
        
    <div class="actions">
      <%= form.submit '注文' %>
    </div>
<% end %>
```

`app/controllers/orders_controller.rb`
```rb
  def create
    @order = Order.new(order_params)
    if @order.save
      OrderDetail.create_items(@order, @current_cart.line_items)
      redirect_to complete_orders_path(@order), notice: '注文が正常に登録されました'
    else
      redirect_to new_order_path, alert: '注文の登録ができませんでした'  
    end
  end
```

#### complete.html.erbでorderの情報を個別に取り出すためにルーティングを設定  
`config / routes.rb`
```rb
   resources :orders, only: [:new, :create] do
    collection do
      get :confirm
    end
    member do # 変更
      get :complete
    end
  end
```

`app/controllers/orders_controller.rb`
```rb
def complete
  @order = Order.find(params[:id])
end
```

`app/views/orders/complete.html.erb`
```erb
<p>注文が完了しました</p>
<p>注文数: <%= @order.total_number %><span>冊</span></p>  # 追加
<p>合計金額: <%= @order.total_price %><span>円（税込）</span></p> # 追加
<%= link_to '商品一覧へ戻る', products_path %> # 追加
```

`app/controllers/orders_controller.rb`
```rb
  def create
    @order = Order.new(order_params)
    if @order.save
      OrderDetail.create_items(@order, @current_cart.line_items)
      redirect_to complete_order_path(@order), notice: '注文が正常に登録されました'
    else
      redirect_to new_order_path, alert: '注文の登録ができませんでした'  
    end
  end
```

<br>

### (15)


<br>

### (16) 


<br>

### (17) 


<br>

### (18) 


<br>

### (19)
```
$ rails generate mailer CompleteMailer complete_mail
```

`app/mailers/complete_mailer.rb`
```rb
class CompleteMailer < ApplicationMailer

  # Subject can be set in your I18n file at config/locales/en.yml
  # with the following lookup:
  #
  #   en.complete_mailer.complete_mail.subject
  #
  def complete_mail
    @greeting = "Hi" # 削除

    mail to: "to@example.org" # 削除

    mail to: '自身のメールアドレス', subject: '注文完了のお知らせ' # 追加
  end
end
```

`app/views/complete_mailer/complete_mail.html.erb`
```erb
<h1>Complete#complete_mail</h1> # 削除

<p> # 削除
  <%= @greeting %>, find me in app/views/complete_mailer/complete_mail.html.erb # 削除
</p> # 削除

<!DOCTYPE html> # 追加
<html> # 追加
  <head> # 追加
	<meta content='text/html; charset=UTF-8' http-equiv='Content-Type' /> # 追加
  </head> # 追加
  <body> # 追加
	<h1>注文完了のお知らせ</h1> # 追加
	<p>ご注文ありがとうございました。</p> # 追加
  </body> # 追加
</html> #追加
```

`app/views/complete_mailer/complete_mail.text.erb`
```erb
Complete#complete_mail # 削除

<%= @greeting %>, find me in app/views/complete_mailer/complete_mail.text.erb # 削除
注文完了のお知らせ # 追加
========================================= # 追加
ご注文ありがとうございました。 # 追加
```
`app/controllers/orders_controller.rb`
```rb
class OrdersController < ApplicationController
  def create
    @order = Order.new(order_params)
    if @order.save
      redirect_to complete_orders_path
    else
      render "confirm"
    end
  end

  def complete
  	CompleteMailer.complete_mail.deliver # 追加
  end

  private
end
```

`config/environments/development.rb`
```rb
require "active_support/core_ext/integer/time"

Rails.application.configure do
  
  # Uncomment if you wish to allow Action Cable access from any origin.
  # config.action_cable.disable_request_forgery_protection = true

  config.action_mailer.delivery_method = :smtp # 追加
  config.action_mailer.smtp_settings = { # 追加
	address: 'smtp.gmail.com', # 追加
	port: 587, # 追加
	domain: 'gmail.com', # 追加
	user_name: ENV['MAIL_USER_NAME'], # 追加
  password: ENV['MAIL_PASSWORD'], # 追加
	authentication: 'plain', # 追加
	enable_starttls_auto: true # 追加
  } # 追加
end
```