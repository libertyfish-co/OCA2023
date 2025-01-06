## 9.3 Ruby on Rails：ECサイトの開発 ログイン認証とユーザー管理4

### 9.3.1 例題(つづき)

#### テストの追加

##### rspecのインストール

rspecがインストールされていない場合はインストールしましょう

`Gemfile`

```rb
・
・
group :development, :test do
  ・
  ・
  gem 'rspec-rails' # 追加
end
```

```sh
$ bundle install
$ rails g rspec:install
```

Deviseのテストを行うにはRequest specを利用します。

##### ログアウトした状態でのテスト

ログアウトした状態でのテストは特に何も必要ありません。
いつも通りrequest specを書けば良いです。

```sh
$ mkdir spec/requests
$ touch spec/requests/mypage_spec.rb
```

```rb
# spec/requests/mypage_spec.rb

require 'rails_helper'

RSpec.describe 'Mypage', type: :request do
  describe 'GET /mypage' do
    context 'ログインしていない場合' do
      it 'ログイン画面へリダイレクトすること' do
        get mypage_path
        expect(response).to redirect_to(:new_user_session)
      end
    end
  end
end
```


##### ログインした状態でのテスト

ログインしたい状態のテストを行うにはログインした状態を作ってからrequestを投げる必要があります。

ログイン状態を作るための設定を行います。

```rb
# spec/rails_helper.rb
・
・
RSpec.configure do |config|
  ・
  ・
  # Devise
  config.include Warden::Test::Helpers
  config.before :suite do
    Warden.test_mode!
  end
end
```

specファイルを修正します。

```rb
# spec/request/mypage_spec.rb

require 'rails_helper'

RSpec.describe 'Mypage', type: :request do
  describe 'GET /mypage' do
    context 'ログインしていない場合' do
      it 'ログイン画面へリダイレクトすること' do
        get mypage_path
        expect(response).to redirect_to(:new_user_session)
      end
    end
    
    context 'ログインしている場合' do
      let(:user) { User.create(email: 'test@example.com', password: 'password') }
      
      before do
        login_as user
      end
      
      it 'マイページが表示されること' do
        get mypage_path
        expect(response.body).to include 'ここはマイページです'
      end
    end
  end
end  
```

これでログインしている時のマイページのテストとログインしていない時のマイページのテストができました。

`-fd`オプションでファイルを指定しましょう。

```sh
$ bundle exec rspec -fd spec/requests/mypage_spec.rb

Mypage
  GET /mypage
    ログインしていない場合
      ログイン画面へリダイレクトすること
    ログインしている場合
      マイページが表示されること

Finished in 0.32214 seconds (files took 2.68 seconds to load)
2 examples, 0 failures
```

このように出力されていればテストに成功しています。


### 9.3.2 問題

それでは、今作成しているECサイトの商品一覧に認証を追加してみましょう。また、新たにユーザーのCRUDを作成し認証を追加しましょう。
