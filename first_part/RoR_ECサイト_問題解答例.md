## Ruby on Rails：ECサイト　解答例

```bash
rails new ec_site
cd ec_site

# gemfile を編集してから
bundle install
rails g devise:install

# Deviseでモデルの作成
rails g devise User
rails g scaffold Book
rails g scaffold Tag
rails g scaffold Tagging

# マイグレーションファイルを編集してから
rails db:migrate

# コントローラーの作成
rails g devise:controllers users
rails g devise:controllers admins
rails g controllers mypage
rails g controllers

# ビューの作成
rails g devise:views
rails g devise:admins
touch app/views/mypage/show.html.erb

```

---

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
  
### `db/migrate/xxxxxxxxxxxxxx_create_books.rb`
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

---

### `db/migrate/xxxxxxxxxxxxxx_create_tags.rb`
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

---

### `db/migrate/xxxxxxxxxxxxxx_create_taggings.rb`
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

---

### `db/migrate/xxxxxxxxxxxxxx_devise_create_users.rb`
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

---

### `db/migrate/xxxxxxxxxxxxxx_create_admins.rb`
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

---

