## Ruby on Rails：ECサイト　解答例

### `config/application.rb`
```rb
# 省略

module EcSite
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 7.1

    config.time_zone = 'Tokyo'

    # 省略
  end
end
```

---

### Migrationファイル
  
#### `db/migrate/xxxxxxxxxxxxxx_create_books.rb`
```rb
class CreateBooks < ActiveRecord::Migration[7.1]
  def change
    create_table :books do |t|
      t.string :book_name, null: false
      t.string :author_name, null: false
      t.date :issue_date
      t.boolean :product_display, null: false, default: true
      t.integer :price, null: false
      t.integer :status, null: false, default: 1

      t.timestamps
    end
  end
end
```
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_create_tags.rb`
```rb
class CreateTags < ActiveRecord::Migration[7.1]
  def change
    create_table :tags do |t|
      t.integer :tagging_id
      t.string :tag_name

      t.timestamps
    end
  end
end
```
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_create_taggings.rb`
```rb
class CreateTaggings < ActiveRecord::Migration[7.1]
  def change
    create_table :taggings do |t|
      t.references :book, null: false, foreign_key: true
      t.references :tag, null: false, foreign_key: true

      t.timestamps
    end
  end
end
```
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_devise_create_users.rb`
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
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.string   :current_sign_in_ip
      t.string   :last_sign_in_ip

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
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_create_admins.rb`
```rb
# frozen_string_literal: true

class DeviseCreateAdmins < ActiveRecord::Migration[7.1]
  def change
    create_table :admins do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      # 省略

      t.timestamps null: false
    end

    add_index :admins, :email,                unique: true
    add_index :admins, :reset_password_token, unique: true
    # add_index :admins, :confirmation_token,   unique: true
    # add_index :admins, :unlock_token,         unique: true
  end
end
```
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_create_orders.rb`
```rb
class CreateOrders < ActiveRecord::Migration[7.1]
  def change
    create_table :orders do |t|
      t.integer :quantity, null: false
      t.string :delivery_address, null: false
      t.integer :total, null: false 

      t.timestamps
    end
  end
end
```
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_create_order_details.rb`
```rb
class CreateOrderDetails < ActiveRecord::Migration[7.1]
  def change
    create_table :order_details do |t|
      t.references :book, null: false, foreign_key: true
      t.references :order, null: false, foreign_key: true
      t.integer :count, null: false
      t.integer :price, null: false

      t.timestamps
    end
  end
end
```
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_create_order_carts.rb`
```rb
class CreateCarts < ActiveRecord::Migration[7.1]
  def change
    create_table :carts do |t|
      t.string :session_id, null: false
      t.timestamps
    end
    add_index :carts, :session_id, unique: true
  end
end
```
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_create_line_items.rb`
```rb
class CreateLineItems < ActiveRecord::Migration[7.1]
  def change
    create_table :line_items do |t|
      t.references :book, null: false, foreign_key: true
      t.references :cart, null: false, foreign_key: true
      t.integer :quantity, default: 0
      t.integer :price, default: 0

      t.timestamps
    end
  end
end
```
  
  
  
#### `db/migrate/xxxxxxxxxxxxxx_create_active_storage_tables.active_storage.rb`
```rb
# This migration comes from active_storage (originally 20170806125915)
class CreateActiveStorageTables < ActiveRecord::Migration[7.0]
  def change
    # Use Active Record's configured type for primary and foreign keys
    primary_key_type, foreign_key_type = primary_and_foreign_key_types

    create_table :active_storage_blobs, id: primary_key_type do |t|
      t.string   :key,          null: false
      t.string   :filename,     null: false
      t.string   :content_type
      t.text     :metadata
      t.string   :service_name, null: false
      t.bigint   :byte_size,    null: false
      t.string   :checksum

      if connection.supports_datetime_with_precision?
        t.datetime :created_at, precision: 6, null: false
      else
        t.datetime :created_at, null: false
      end

      t.index [ :key ], unique: true
    end

    create_table :active_storage_attachments, id: primary_key_type do |t|
      t.string     :name,     null: false
      t.references :record,   null: false, polymorphic: true, index: false, type: foreign_key_type
      t.references :blob,     null: false, type: foreign_key_type

      if connection.supports_datetime_with_precision?
        t.datetime :created_at, precision: 6, null: false
      else
        t.datetime :created_at, null: false
      end

      t.index [ :record_type, :record_id, :name, :blob_id ], name: :index_active_storage_attachments_uniqueness, unique: true
      t.foreign_key :active_storage_blobs, column: :blob_id
    end

    create_table :active_storage_variant_records, id: primary_key_type do |t|
      t.belongs_to :blob, null: false, index: false, type: foreign_key_type
      t.string :variation_digest, null: false

      t.index [ :blob_id, :variation_digest ], name: :index_active_storage_variant_records_uniqueness, unique: true
      t.foreign_key :active_storage_blobs, column: :blob_id
    end
  end

  private
    def primary_and_foreign_key_types
      config = Rails.configuration.generators
      setting = config.options[config.orm][:primary_key_type]
      primary_key_type = setting || :primary_key
      foreign_key_type = setting || :bigint
      [primary_key_type, foreign_key_type]
    end
end
```

---

### Model
  
#### `app/models/user.rb`
```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :recoverable and :omniauthable
  devise :database_authenticatable, :registerable,
         :trackable, :rememberable, :validatable
end
```
  
  
  
#### `app/models/admin.rb`
```rb
class Admin < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
end
```
  
  
  
#### `app/models/application_record.rb`
```rb
class ApplicationRecord < ActiveRecord::Base
  primary_abstract_class
end
```
  
  
  
#### `app/models/book.rb`
```rb
class Book < ApplicationRecord
  has_many :tagging, dependent: :destroy
  has_many :tags, through: :tagging
    
  has_one_attached :image

  has_many :order_details
  has_many :orders, through: :order_details

  has_many :line_items, dependent: :destroy
  has_many :carts, through: :line_items

  enum status: {販売中: 1, 品切れ: 2}

  def self.ransackable_attributes(auth_object = nil)
    ["book_name"]
  end 
end
```
  
  
  
#### `app/models/cart.rb`
```rb
class Cart < ApplicationRecord
 
  has_many :line_items
  has_many :books, through: :line_items

  # カートに商品を追加するメソッド
  def add_product(book, quantity)
    line_item = line_items.find_or_initialize_by(book: book)
    line_item.quantity += quantity
    line_item.price = book.price
    line_item.save
  end

  # カートの合計金額
  def total_price
    line_items.sum { |item| item.quantity * item.price }
  end

  # カート内の合計個数
  def total_number
    line_items.sum(:quantity)
  end

end
```
  
  
  
#### `app/models/line_item.rb`
```rb
class LineItem < ApplicationRecord
  belongs_to :book
  belongs_to :cart

  # 小計金額を計算
  def total_price
    quantity * price
  end
end
```
  
  
  
#### `app/models/order_details.rb`
```rb
class OrderDetail < ApplicationRecord
  belongs_to :book
  belongs_to :order
end
```
  
  
  
#### `app/models/order.rb`
```rb
class Order < ApplicationRecord
  has_many :order_details, dependent: :destroy
  has_many :books, through: :order_details
end
```
  
  
  
#### `app/models/tags.rb`
```rb
class Tag < ApplicationRecord
  has_many :tagging, dependent: :destroy
  has_many :books, through: :tagging
end
```
  
  
  
#### `app/models/tagging.rb`
```rb
class Tagging < ApplicationRecord
  belongs_to :book
  belongs_to :tag
end
```

---

### Seed

#### `db/seeds.rb`
```rb
# This file should ensure the existence of records required to run the application in every environment (production,
# development, test). The code here should be idempotent so that it can be executed at any point in every environment.
# The data can then be loaded with the bin/rails db:seed command (or created alongside the database with db:setup).
#
# Example:
#
#   ["Action", "Comedy", "Drama", "Horror"].each do |genre_name|
#     MovieGenre.find_or_create_by!(name: genre_name)
#   end

admins = Admin.create!(
  { email: 'admin@admin100.com', password: 'Admin100' }
)
```
  
  
  
#### `db/schema.rb`
```rb
# 省略

ActiveRecord::Schema[7.1].define(version: 2025_02_12_124747) do
  create_table "active_storage_attachments", force: :cascade do |t|
    t.string "name", null: false
    t.string "record_type", null: false
    t.bigint "record_id", null: false
    t.bigint "blob_id", null: false
    t.datetime "created_at", null: false
    t.index ["blob_id"], name: "index_active_storage_attachments_on_blob_id"
    t.index ["record_type", "record_id", "name", "blob_id"], name: "index_active_storage_attachments_uniqueness", unique: true
  end

  create_table "active_storage_blobs", force: :cascade do |t|
    t.string "key", null: false
    t.string "filename", null: false
    t.string "content_type"
    t.text "metadata"
    t.string "service_name", null: false
    t.bigint "byte_size", null: false
    t.string "checksum"
    t.datetime "created_at", null: false
    t.index ["key"], name: "index_active_storage_blobs_on_key", unique: true
  end

  create_table "active_storage_variant_records", force: :cascade do |t|
    t.bigint "blob_id", null: false
    t.string "variation_digest", null: false
    t.index ["blob_id", "variation_digest"], name: "index_active_storage_variant_records_uniqueness", unique: true
  end

  create_table "admins", force: :cascade do |t|
    t.string "email", default: "", null: false
    t.string "encrypted_password", default: "", null: false
    t.string "reset_password_token"
    t.datetime "reset_password_sent_at"
    t.datetime "remember_created_at"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["email"], name: "index_admins_on_email", unique: true
    t.index ["reset_password_token"], name: "index_admins_on_reset_password_token", unique: true
  end

  create_table "books", force: :cascade do |t|
    t.string "book_name", null: false
    t.string "author_name", null: false
    t.date "issue_date"
    t.boolean "product_display", default: true, null: false
    t.integer "price", null: false
    t.integer "status", default: 1, null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "carts", force: :cascade do |t|
    t.string "session_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["session_id"], name: "index_carts_on_session_id", unique: true
  end

  create_table "line_items", force: :cascade do |t|
    t.integer "book_id", null: false
    t.integer "cart_id", null: false
    t.integer "quantity", default: 0
    t.integer "price", default: 0
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["book_id"], name: "index_line_items_on_book_id"
    t.index ["cart_id"], name: "index_line_items_on_cart_id"
  end

  create_table "order_details", force: :cascade do |t|
    t.integer "book_id", null: false
    t.integer "order_id", null: false
    t.integer "count", null: false
    t.integer "price", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["book_id"], name: "index_order_details_on_book_id"
    t.index ["order_id"], name: "index_order_details_on_order_id"
  end

  create_table "orders", force: :cascade do |t|
    t.integer "quantity", null: false
    t.string "delivery_address", null: false
    t.integer "total", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "taggings", force: :cascade do |t|
    t.integer "book_id", null: false
    t.integer "tag_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["book_id"], name: "index_taggings_on_book_id"
    t.index ["tag_id"], name: "index_taggings_on_tag_id"
  end

  create_table "tags", force: :cascade do |t|
    t.integer "tagging_id"
    t.string "tag_name"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "users", force: :cascade do |t|
    t.string "email", default: "", null: false
    t.string "encrypted_password", default: "", null: false
    t.string "reset_password_token"
    t.datetime "reset_password_sent_at"
    t.datetime "remember_created_at"
    t.integer "sign_in_count", default: 0, null: false
    t.datetime "current_sign_in_at"
    t.datetime "last_sign_in_at"
    t.string "current_sign_in_ip"
    t.string "last_sign_in_ip"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["email"], name: "index_users_on_email", unique: true
    t.index ["reset_password_token"], name: "index_users_on_reset_password_token", unique: true
  end

  add_foreign_key "active_storage_attachments", "active_storage_blobs", column: "blob_id"
  add_foreign_key "active_storage_variant_records", "active_storage_blobs", column: "blob_id"
  add_foreign_key "line_items", "books"
  add_foreign_key "line_items", "carts"
  add_foreign_key "order_details", "books"
  add_foreign_key "order_details", "orders"
  add_foreign_key "taggings", "books"
  add_foreign_key "taggings", "tags"
end
```

---

### Routing

#### `config/routes.rb`
```rb
Rails.application.routes.draw do
  devise_for :admins, controllers: {
    sessions: 'admins/sessions'
  }

  resources :tags, only: [:index, :new, :create, :destroy]
  resources :books

  devise_for :users

  root to: "products#index"

  get '/search', to: 'searchs#search'

  resources :mypages, only: [:show]
 
  resources :products, only: [:index, :show]

  resources :carts, only: [:show, :destroy]

  resources :line_items, only: [:create]

  resources :orders, only: [:new, :create] do
    collection do
      get :confirm  # 注文確認画面
    end

    member do
      get :complete
    end
  end
  
  # 省略

end
```

---

### controller
  
#### `app/controllers/application_controller.rb`
```rb
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?
  helper_method :current_cart
  before_action :search

  def search
    @q = Book.ransack(params[:q])
    @book = @q.result(distinct: true)
    @result = params[:q]&.values&.reject(&:blank?)
  end


  def after_sign_in_path_for(resource)
      mypage_path(resource)
  end

  def after_sign_out_path_for(resource)
      root_path
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:email])
  end

  def current_cart
    @current_cart ||= Cart.find_or_create_by(session_id: session.id.to_s)
  end

end
```
  
  
  
#### `app/cotrollers/admins/sessions.rb`
```rb
# frozen_string_literal: true

class Admins::SessionsController < Devise::SessionsController
  # before_action :configure_sign_in_params, only: [:create]
  before_action :configure_permitted_parameters, if: :devise_controller?

  def after_sign_in_path_for(resource)
    books_path
  end

  def after_sign_out_path_for(resource)
    new_admin_session_path
  end

  # GET /resource/sign_in
  # def new
  #   super
  # end

  # POST /resource/sign_in
  # def create
  #   super
  # end

  # DELETE /resource/sign_out
  # def destroy
  #   super
  # end

  # protected

  # If you have extra params to permit, append them to the sanitizer.
  # def configure_sign_in_params
  #   devise_parameter_sanitizer.permit(:sign_in, keys: [:attribute])
  # end
end
```
  
  
  
#### `app/controllers/books_controller.rb`
```rb
class BooksController < ApplicationController
  before_action :set_book, only: %i[ show edit update destroy ]
  before_action :admin_scan, only: %i[ show edit update destroy ]

  # GET /books or /books.json
  def index
    @books = Book.all
  end

  # GET /books/1 or /books/1.json
  def show
  end

  # GET /books/new
  def new
    @book = Book.new
  end

  # GET /books/1/edit
  def edit
  end

  # POST /books or /books.json
  def create
    @book = Book.new(book_params)

    respond_to do |format|
      if @book.save
        format.html { redirect_to @book, notice: "新しい書籍を登録しました" }
        format.json { render :show, status: :created, location: @book }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @book.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /books/1 or /books/1.json
  def update
    respond_to do |format|
      if @book.update(book_params)
        format.html { redirect_to @book, notice: "書籍情報を編集しました" }
        format.json { render :show, status: :ok, location: @book }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @book.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /books/1 or /books/1.json
  def destroy
    @book.destroy!

    respond_to do |format|
      format.html { redirect_to books_path, status: :see_other, notice: "書籍を削除しました" }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_book
      @book = Book.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def book_params
      params.require(:book).permit(:book_name, :author_name, :issue_date, :product_display, :price, :status, :image, tag_ids: []) # 追加
    end

    def admin_scan
      unless admin_signed_in?
        redirect_to products_path   #管理者以外が管理者のみ実行できるアクションを実行しようとしたときのリダイレクト先
      end
    end
end
```
  
  
  
#### `app/controllers/mypages_controller.rb`
```rb
class LineItemsController < ApplicationController
  def create
    @book = Book.find(params[:book_id])
    quantity = params[:quantity].to_i
    current_cart.add_product(@book, quantity)
    redirect_to cart_path(@book.id), notice: "#{@book.book_name}がカートに追加されました"
  end
end
```
  
  
  
#### `app/controllers/tags_controller.rb`
```rb
class TagsController < ApplicationController
  before_action :set_tag, only: %i[ show destroy ]

  # GET /tags or /tags.json
  def index
    @tags = Tag.all
  end

  # GET /tags/new
  def new
    @tag = Tag.new
  end

  # POST /tags or /tags.json
  def create
    @tag = Tag.new(tag_params)

    respond_to do |format|
      if @tag.save
        format.html { redirect_to tags_url, notice: "新しいタグを登録しました" }
        format.json { render :show, status: :created, location: @tag }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @tag.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /tags/1 or /tags/1.json
  def destroy
    @tag.destroy!

    respond_to do |format|
      format.html { redirect_to tags_path, status: :see_other, notice: "タグを削除しました" }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_tag
      @tag = Tag.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def tag_params
      params.require(:tag).permit(:tag_name)
    end
end

```
  
  
  
#### `app/controllers/products_controller.rb`
```rb
class ProductsController < ApplicationController
  before_action :authenticate_user!
  before_action :set_book, only: %i[ show ]

  def index
    @products = Book.all
  end

  def show
    @product = Book.find(params[:id])
    @line_item = LineItem.new
  end

  private

    def set_book
      @product = Book.find(params[:id])
    end

    def book_params
      params.require(:book).permit(:book_name, :author_name, :issue_date, :product_display, :price, :status, :image, tag_ids: []) # 追加
    end

end
```
  
  
  
####  `app/controllers/carts_controller.rb`
```rb
class CartsController < ApplicationController
  before_action :authenticate_user! # 追加
  
  def show
    @cart = current_cart
  end

  def destroy
    current_cart.line_items.destroy_all
    redirect_to products_path, alert: 'カートが空になりました'
  end
  
end
```
  
  
#### `app/controllers/orders_controller.rb`
```rb
class OrdersController < ApplicationController
  before_action :authenticate_user! # 追加

  def new
    @order = Order.new
    @line_items = current_cart.line_items  # 現在のカートのアイテムを取得
  end

  def confirm
      
    @order = Order.new(order_params)
    @line_items = current_cart.line_items # カートの内容を取得
    @order.total = calculate_total # 合計金額を計算

    @cart = current_cart
    # 注文詳細を作成
    @cart.line_items.each do |line_item|
      # フォームから送られてきた数量を反映
      quantity = params[:order]["line_item_quantity_#{line_item.id}"].to_i
        
      # 数量が1以上なら更新する
      if quantity > 0
        line_item.update(quantity: quantity)
      else
        # 数量が0以下ならその行を削除する
        line_item.destroy
      end
  
      # 注文詳細を作成
      @order.order_details.build(
        book: line_item.book,
        count: line_item.quantity,
        price: line_item.price
      )
    end
  
    # 合計金額や数量を計算
    @order.total = calculate_total
    @order.quantity = calculate_quantity

    # 注文詳細を仮にセット（確認画面用）
    @order_details = @line_items.map do |line_item| {
       book_name: line_item.book.book_name,
       quantity: line_item.quantity,
       price: line_item.price,
       total_price: line_item.price * line_item.quantity
     }
    end
  end

  def create
      
    @order = Order.new(order_params)
    @order.total = calculate_total # 合計金額を計算
    @order.quantity = calculate_quantity

    # 注文詳細を作成
    current_cart.line_items.each do |line_item|
      @order.order_details.build(
        book: line_item.book,
        count: line_item.quantity,
        price: line_item.price
      )
    end

    if @order.save
      # 注文後にカートを空にする
      current_cart.line_items.destroy_all

      redirect_to complete_order_path(@order)
    else
      render :new
    end
  end

  def complete
    @order = Order.find(params[:id]) # 注文情報を取得
  end

  private

    def order_params
      params.require(:order).permit(:quantity, :delivery_address, :total, line_items: [])
    end

    def calculate_total
      current_cart.line_items.sum { |line_item| line_item.price * line_item.quantity }
    end

    def calculate_quantity
      current_cart.line_items.sum { |line_item| line_item.quantity }
    end

end
```
  
  
  
#### `app/controllers/line_items_controller.rb`
```rb
class LineItemsController < ApplicationController
  def create
    @book = Book.find(params[:book_id])
    quantity = params[:quantity].to_i
    current_cart.add_product(@book, quantity)
    redirect_to cart_path(@book.id), notice: "#{@book.book_name}がカートに追加されました"
  end
end
```
  
---

### View

#### layouts

#### `app/views/layouts/application.html.erb`
```rb
<!DOCTYPE html>
<html>
  <head>
    <title>EcSite</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <header class="">
      <div class="">
        <ul class="">
          <!-- 管理者 -->
          <% if admin_signed_in? %>

            <%= render "layouts/search" %>
            <li><%= link_to "新規書籍", new_book_path %></li>
            <li><%= link_to "新規タグ", new_tag_path %></li>
            <li><%= link_to '書籍一覧', books_path %></li>
            <li><%= link_to 'タグ一覧', tags_path %></li>
            <li><%= link_to 'ログアウト', destroy_admin_session_path, data: { turbo_method: :delete } %></li>

          <!-- ユーザー -->
          <% elsif user_signed_in? %>

            <%= render "layouts/search" %>
            <li><%= link_to '書籍一覧', products_path %></li>
            <li><%= link_to 'ユーザーページ', mypage_path(current_user.id) %></li>
            <li><%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete } %></li>

          <!-- 未ログイン状態 -->
          <% else %>

            <li><%= link_to 'サインアップ', new_user_registration_path %></li>
            <li><%= link_to 'ログイン', new_user_session_path %></li>

          <% end %>
        </ul>
      </div>
    </header>

    <%= yield %>
  </body>
</html>
```
  
  
  
#### `app/views/layouts/_search.html.erb`
```html
<% if controller_name != 'sessions' %>
  <div>
    <%= search_form_for @q, url: search_path do |f| %>
      <div>
        <%= f.search_field :book_name_cont %>
        <button type="submit", style="width: 100px; height: 25px;">
          検索ボタン
        </button>
      </div>
    <% end %>
  </div>
<% end %>
```

---

#### users

#### `app/devise/registrations/new.html.erb`
```html
<h2>サインアップ</h2>

<%= form_with model: @user, url: user_registration_path do |f| %>
  <%= render "devise/shared/error_messages", resource: resource %>

  <div class="field">
    <%= f.label :メールアドレス %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
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

  <div class="actions">
    <%= f.submit "サインアップ" %>
  </div>
<% end %>

<%= render "devise/shared/links" %>
```
  
  
  
#### `app/devise/sessions/new.html.erb`
```html
<h2>ログイン</h2>

<%= form_with model: @user, url: user_session_path do |f| %>
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

  <div class="actions">
    <%= f.submit "ログイン" %>
  </div>
<% end %>

<%= render "devise/shared/links" %>
```
  
  
  
#### `app/devise/shared/_link.html.erb`
```html
<%- if controller_name != 'sessions' %>
  <%= link_to "ログイン", new_session_path(resource_name) %><br />
<% end %>

<%- if devise_mapping.registerable? && controller_name != 'registrations' %>
  <%= link_to "サインアップ", new_registration_path(resource_name) %><br />
<% end %>
```
  
---  
  
#### mypages

#### `app/views/mypages/show.html.erb`
```html
<p style="color: green"><%= notice %></p>

<h2>マイページ</h2>
あなたのメールアドレスは<br>
<strong><%= @user.email %></strong><br>
です。
```
  
---
  
#### admins

#### `app/views/admins/sessions/new.html.erb`
```html
<h2>管理者ログイン</h2>

<%= form_with model: @admin, url: admin_session_path do |f| %>
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

  <div class="actions">
    <%= f.submit "ログイン" %>
  </div>
<% end %>
```

---

#### books

#### `app/views/books/index.html.erb`
```html
<p style="color: green"><%= notice %></p>

<h2>書籍一覧</h2>

<div id="books">
  <% @books.each do |book| %>
    <%= render book %>
    <p>
      <%= link_to "書籍詳細", book %>
    </p>
    <br>
  <% end %>
</div>
```
  
  
  
#### `app/views/books/show.html.erb`
```html
<p style="color: green"><%= notice %></p>

<%= render @book %>

<div>
  <p>
    <strong>タグ名</strong>
    <%= @book.tags.map(&:tag_name).join(', ') %>
  </p>
</div>

<div>
  <%= link_to "編集", edit_book_path(@book) %> 
  <%= button_to "書籍の削除", @book, method: :delete %>
</div>
```
  
  
  
#### `app/views/books/_book.html.erb`
```html
<div id="<%= dom_id book %>">

  <% if admin_signed_in? %>

    <% if book.image.attached? %>
      <%= image_tag book.image, size: "200x200" %>
    <% end %>
    <p>
      <strong>書籍名：</strong>
      <%= book.book_name %>
    </p>
    <p>
      <strong>著者名：</strong>
      <%= book.author_name %>
    </p>
    <p>
      <strong>発行日：</strong>
      <%= book.issue_date %>
    </p>
    <p>
      <strong>商品表示：</strong>
      <%= book.product_display %>
    </p>
    <p>
      <strong>価格：</strong>
      <%= book.price.to_fs(:delimited) %>
    </p>
    <p>
      <strong>ステータス：</strong>
      <%= book.status %>
    </p>

  <% end %>

</div>
```
  
  
  
#### `app/views/books/new.html.erb`
```html
<h1>書籍新規登録</h1>

<%= render "form", book: @book %>

<br>
```
  
  
  
#### `app/views/books/edit.html.erb`
```rb
<h1>書籍編集</h1>

<%= render "form", book: @book %>

<br>

<div>
  <%= link_to "書籍詳細へ戻る", @book %> |
</div>
```
  
  
  
#### `app/views/books/_form.html.erb`
```html
<%= form_with(model: book) do |form| %>
  <% if book.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(book.errors.count, "error") %> prohibited this book from being saved:</h2>

      <ul>
        <% book.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.file_field :image %>
  </div>

  <div>
    <%= form.label :書籍名, style: "display: block" %>
    <%= form.text_field :book_name %>
  </div>

  <div>
    <%= form.label :著者名, style: "display: block" %>
    <%= form.text_field :author_name %>
  </div>

  <div>
    <%= form.label :発行日, style: "display: block" %>
    <%= form.date_field :issue_date %>
  </div>

  <div>
    <%= form.label :商品表示, style: "display: block" %>
    <%= form.check_box :product_display %>
  </div>

  <div>
    <%= form.label :価格, style: "display: block" %>
    <%= form.number_field :price %>
  </div>

  <div>
    <%= form.label :タグ, style: "display: block" %>
    <%= form.collection_check_boxes :tag_ids, Tag.all, :id, :tag_name %>
  </div>

  <div>
    <%= form.label :ステータス, style: "display: block" %>
    <%= form.text_field :status %>
  </div>
  
  <div class="actions">
    <%= form.submit "登録"%>
  </div>
<% end %>
```

---

#### tags

#### `app/views/tags/index.html.erb`
```html
<p style="color: green"><%= notice %></p>

<h1>タグ一覧</h1>

<div id="tags">
  <% @tags.each do |tag| %>
    <%= render tag %>
    <p>

      <%= button_to "タグの削除", tag, method: :delete %>

    </p>
  <% end %>
</div>
```
  
  
  
#### `app/views/tags/_tag.html.erb`
```html
<div id="<%= dom_id tag %>">
  <p>
    <strong>タグ名：</strong>
    <%= tag.tag_name %>
  </p>

</div>
```
  
  
  
#### `app/views/tags/new.html.erb`
```html
<h1>新規タグ</h1>

<%= render "form", tag: @tag %>

<br>

<div>
  <%= link_to "一覧へ戻る", tags_path %>
</div>
```
  
  
  
#### `app/views/tags/_form.html.erb`
```html
<%= form_with(model: tag) do |form| %>
  <% if tag.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(tag.errors.count, "error") %> prohibited this tag from being saved:</h2>

      <ul>
        <% tag.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :タグ名, style: "display: block" %>
    <%= form.text_field :tag_name %>
  </div>

  <div>
    <%= form.submit '登録' %>
  </div>
<% end %>
```

---

#### products

#### `app/views/products/index.html.erb`
```html
<p style="color: green"><%= notice %></p>
<p style="color: red"><%= alert %></p>

<h2>商品一覧</h2>

<div id="books">

  <table>
  
    <thead>
      <tr>
        <th>書籍名</th>
        <th>著者名</th>
        <th>発行日</th>
        <th>価格</th>
        <th>商品ステータス</th>
        <th></th>
      </tr>
    </thead>

    <tbody>
      <% @products.each do |product| %>
        <% if product.product_display == true %>
          <tr>
            <td>
              <%= product.book_name %>
            </td>
            <td>
              <%= product.author_name %>
            </td>
            <td>
              <%= product.issue_date %>
            </td>
            <td>
              <%= product.price.to_fs(:delimited) %> 円
            </td>
            <td>
              <%= product.status %>
            </td>
            <td>
              <%= link_to "商品詳細", product_path(product.id) %>
            </td>
          </tr>
        <% end %>
      <% end %>
    </tbody>

  <table>

  <%= link_to "買い物かごを確認する", cart_path(id: current_cart.id) %>

</div>
```
  
  
    
#### `app/views/products/show.html.erb`
```html
<h2>商品詳細</h2>

<% if @product.product_display == true %>

  <p>
    <strong>書籍名：</strong>
    <%= @product.book_name %>
  </p>
  <p>
    <strong>著者名：</strong>
    <%= @product.author_name %>
  </p>
  <p>
    <strong>発行日：</strong>
    <%= @product.issue_date %>
  </p>
  <p>
    <strong>価格：</strong>
    <%= @product.price.to_fs(:delimited) %> 円
  </p>
  <p>
    <strong>商品ステータス：</strong>
    <%= @product.status %>
  </p>


  <%= form_with url: line_items_path(book_id: @product.id), method: :post, local: true do %>
    <%= label_tag :quantity, '数量' %>
    <%= number_field_tag :quantity, 1, min: 1, class: 'form-control' %>
    <%= submit_tag '買い物かごに入れる' %>
  <% end %>

<% end %>
```
  
---
  
#### carts

#### `app/views/carts/show.html.erb`
```html
<p style="color: green"><%= notice %></p>

<h2>買い物かご</h2>

<table class="table">
  <thead>
    <tr>
      <th>書籍名</th>
      <th>数量</th>
      <th>単価</th>
      <th>小計</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <% @cart.line_items.each do |item| %>
      <tr>
        <td>
          <%= item.book.book_name %>
        </td>
        <td>
          <%= item.quantity %>
        </td>
        <td>
          <%= item.price.to_fs(:delimited) %>円
        </td>
        <td>
          <%= item.total_price.to_fs(:delimited) %>円
        </td>
      </tr>
    <% end %>
  </tbody>
</table>

<p><strong>合計金額:</strong> <%= @cart.total_price.to_fs(:delimited) %> 円</p>
<p><strong>合計個数:</strong> <%= @cart.total_number %> 冊</p>

<% if @cart.line_items.present? %>
  <%= link_to '注文する', new_order_path %> | 
  <%= link_to '買い物かごを空にする', cart_path, data: { turbo_method: :delete, turbo_confirm: "カートを空にしますか？" } %>

<% end %>
```

---

#### orders

#### `app/views/orders/new.html.erb`
```html
<h2>注文画面</h2>

<%= form_with(model: @order, url: confirm_orders_path, method: :get) do |f| %>
  <div>
    <h3>カートの内容</h3>

    <table>
      <thead>
        <tr>
          <th>書籍名</th>
          <th>数量</th>
          <th>単価</th>
        </tr>
      </thead>

      <tbody>
        <% @line_items.each do |line_item| %>
          <% if line_item.book.present? %>
            <tr>
              <td>
                <%= line_item.book.book_name %>
              </td>
              <td>
                <%= f.number_field "line_item_quantity_#{line_item.id}", value: line_item.quantity %>
              </td>
              <td>
                <%= line_item.price.to_fs(:delimited) %> 円
              <td>
            </tr>
          <% end %>
        <% end %>
      <tbody>
    </table>
  </div>

  <br>

  
  <h3><%= f.label :送り先, style: "display: block" %></h3>
  <%= f.text_field :delivery_address %>

  <div>
    <%= f.submit '注文確認' %>
  </div>
  <div>
    <%= link_to 'カートに戻る', cart_path(id: current_cart.id) %>
  </div>
<% end %>
```
  
  
  
#### `app/views/orders/confirm.html.erb`
```html
<h2>注文確認画面</h2>

<h3>カートの内容</h3>

<table>
  <thead>
    <tr>
      <th>書籍名</th>
      <th>数量</th>
      <th>単価</th>
      <th>合計</th>
    </tr>
  </thead>

  <tbody>
    <% @order_details.each do |detail| %>
      <tr>
        <td>
          <%= detail[:book_name] %>
        </td>
        <td>
          <%= detail[:quantity] %> 冊
        </td>
        <td>
          <%= detail[:price].to_fs(:delimited) %> 円
        </td>
        <td>
          <%= detail[:total_price].to_fs(:delimited) %> 円
        </td>
      </tr>
    <% end %>
  </tbody>
</table>

<p><strong>合計金額：<%= @order.total.to_fs(:delimited) %> 円</strong></p>
<p><strong>送り先：<%= @order.delivery_address %></strong></p>

<%= form_with(model: @order, url: orders_path(@order), data: { turbo_method: :POST }) do |f| %>
  <%= f.hidden_field :delivery_address, value: @order.delivery_address %>
  <%= f.submit '注文確定' %>
<% end %>

<%= link_to '戻る', new_order_path %>
```
  
  
  
#### `app/views/orders/complete.html.erb`
```html
<h2>注文完了</h2>

<p>ご注文が完了しました。</p>
<p>注文番号: <%= @order.id %></p>
<p>注文量: <%= @order.quantity %> 冊</p>
<p>合計金額: <%= @order.total.to_fs(:delimited) %> 円</p>
```

---

#### searchs

#### `app/views/searchs/search.html.erb`
```html
<div>
  <table>
    <% if @result == [] %>

      <h3>検索結果がありません。</h3>

    <% elsif @book.present? %>

      <% search_word = @q.book_name_cont %>
      <h2>検索結果 "<%= search_word %>"</h2>
      <thead>
        <tr>
          <th>書籍名</th>
          <th>価格</th>
        </tr>
      </thead>

      <tbody>
        <% @book.each do |f| %>
          <tr>
            <td>
              <% if admin_signed_in? %>
                <%= link_to f.book_name, book_path(f.id) %>
              <% elsif user_signed_in? %>
                <%= link_to f.book_name, product_path(f.id) %>
              <% end %>
            </td>
            
            <td>
              <%= f.price.to_fs(:delimited) %> 円
            </td>
          </tr>
        <% end %>
      </tbody>

    <% else %>
      
      <h3>検索結果がありません。</h3>
    
    <% end %>
  </table>
</div>
```