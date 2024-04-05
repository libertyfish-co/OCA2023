## 7.3　Ruby on Rails：Railsテスト基礎 1

### 7.3.1 テスティングツールについて

この章では、Railsアプリケーションでのテストの実装方法について学習します。Railsの標準テスティングツールは`Minitest::Test`です。`rails new`コマンドで作成したアプリケーションの雛形を作成したときには、自動的にtestフォルダが作られています。  

Rubyのテストでも紹介しましたが`Minitest::Test`の他に`RSpec`というテスティングツールもあります。どちらが優れているということはありません。`RSpec`はDSL（ドメイン固有言語）を提供しているので、いつも馴染みのあるRubyの文法とは異なる文法を学習する必要があります。  
それぞれどのような違いがあるのか見ていきましょう。  

1. RSpecとMinitestの違い：RSpecとMinitestは、テストの記述方法やDSL（ドメイン固有言語）の提供方法などが異なります。RSpecはより自然言語に近いテスト記述が可能ですが、MinitestはRuby標準のテストフレームワークであるため、よりシンプルな構文です。  

2. RSpecのDSL：RSpecでは`describe`や`it`などのDSLが利用されます。これらのDSLを使うことで、テストコードが自然言語に近くなり、読みやすさが向上します。RSpecのDSLは柔軟性があり、テストの構造を細かく制御することができます。  

3. Minitestのシンプルさ：MinitestはRubyの標準ライブラリに含まれているため、Railsアプリケーションを新規に作成した際にデフォルトで利用されることが多いです。シンプルな構文と軽量な性能が特徴であり、小規模なプロジェクトや速度を重視する場合に適しています。  

4. チームの選択：チームでの選択肢や好みによって、RSpecかMinitestかを選択することがあります。チーム内で統一されたテストフレームワークを選択することで、コードの一貫性が保たれ、コードレビューやメンテナンスがしやすくなります。  


Minitestで例を挙げると次のようになります。

```rb
require 'test_helper' 

class UserTest < ActiveSupport::TestCase
  def test_is_valid_with_name_and_phone_number
    @user = User.new(name: "test", phone_number: "0000000000")
    assert(@user.valid?)
  end

  test "is invalid without name" do
    @user = User.new(phone_number: "0000000000")
    @user.valid?
    assert_operator @user.errors[:name], :include?, "can't be blank"
  end

  test "is invalid without phone number" do
    @user = User.new(name: "test")
    @user.valid?
    assert_operator @user.errors[:phone_number], :include?, "can't be blank"
  end
end
```

1. `require 'test_helper'`：この行は、`test_helper.rb`ファイルをこのテストファイルに読み込むためのものです。
2. `def test_is_valid_with_name_and_phone_number`：この行は、`test_is_valid_with_name_and_phone_number`という名前のテストメソッドの定義を開始します。Minitestでは、テストメソッドの名前は"test"で始まります。
3. `@user = User.new(name: "test", phone_number: "0000000000")`：この行は、名前が"test"で電話番号が "0000000000" の新しい`User`インスタンスを作成します。この`User`インスタンスは、提供された属性で有効かどうかをテストするために使用されます。
4. `assert(@user.valid?)`：この行は、`assert`メソッドを使用して、`@user`オブジェクトが`User`モデルで定義されたバリデーションに従って有効かどうかを確認します。
5. `assert_operator @user.errors[:name], :include?, "can't be blank"`この行は、`assert_operator`メソッドを使用して、`@user.errors[:phone_number]`に`"can't be blank"`が含まれているかどうかをチェックします。

※FactoryBotは、テストデータを効率的に作成するためのGemであり、Railsのテストスイートで広く使用されています。FactoryBotを使用すると、データベースに対してテストデータを簡単に作成し、テストケースで再利用できます。  
以下のように定義します。  
```rb
FactoryBot.define do
  factory :user do
    name { "John Doe" }
    email { "john@example.com" }
    password { "password123" }
  end
end
```

これにより、`name`、`email`、`password`のデフォルト値を持つ`User`オブジェクトを作成するファクトリが定義されます。そして、テストケース内で`build(:user)`を呼び出すことで、これらのデフォルト値を持つ`User`オブジェクトを簡単に作成できます。  

testメソッドは、引数に文字列を渡すことで、その文字列を使用して`test_〇〇`メソッドを作成してくれるメソッドです。  
`test "is_valid_with_name_and_phone_number"`とすると`def is_valid_with_name_and_phone_number`というメソッドを作成してくれます。  

```rb
test "user is valid with name and phone number" do
  user = build(:user)
  assert user.valid?
end
```
アサーションの代表例：
| アサーション一覧                               | 結果                                                 | 
| ---------------------------------------------- | ---------------------------------------------------- | 
| assert(test, [msg])                            | 式testの評価値が真（true）であればテスト成功         | 
| assert_equal(test, supposition, [msg])         | testの値とsuppositionの値が等しければテスト成功      | 
| assert_operator(test, :メソッド, [arg], [msg]) | test.メソッド（arg）が真（true）であればテスト成功   | 
| assert_raises(excep1, excep2, ...) { test }    | ブロック内のtestから引数の例外が発生すればテスト成功 | 
| flunk([msg])                                   | 必ずテストが失敗する                                 | 

それぞれ[msg]に文字列を渡すと、テスト失敗時のメッセージを指定できます。  

`ActionDispatch::IntegrationTest`を継承することによって、Railsアプリケーションの統合テスト用のメソッド(アサーション)を呼び出すことができます。
また、`Minitest::Test`を継承しているため単体テスト同様に、`assert`や`assert_equal`などのアサーションを使用してテストを記述することができます。

RSpecの例を挙げると次のようなソースコードになります。  

```rb
require 'rails_helper'

RSpec.describe User, type: :model do
  it "is valid with name and phone number" do
    user = build(:user)
    expect(user).to be_valid
  end

  it "is invalid without name" do
    user = build(:user, name: nil)
    user.valid?
    expect(user.errors[:name]).to include("can't be blank")
  end

  it "is invalid without phone number" do
    user = build(:user, phone_number: nil)
    user.valid?
    expect(user.errors[:phone_number]).to include("can't be blank")
  end
end
```

`it`、`expect`などいつも見ないメソッドに驚かれたのではないでしょうか。ですが、DSLを利用することで書き方についてのルールを決定することができ、チームでの作業効率が格段とあがります。

1. `RSpec.describe User, type: :model do`：この行はRSpecの`describe`メソッドを使って、`User`モデルに関するテストスイート(テストケースをまとめたもの)を定義しています。`type: :model`はRSpecのメタデータであり、このテストスイートがモデル(`User`)に関するテストであることを示しています。
2. `it "is valid with name and phone number" do`：この行は、1つ目のテストケースの定義を始めています。itメソッドは、単一のテストケースを定義します。このテストケースは、「名前と電話番号がある場合は有効である」ということを検証します。
3. `user = build(:user)`：この行では、`build`メソッドを使って`User`オブジェクトを作成しています。`:user`はFactoryBotで定義されたファクトリ名であり、この場合は`User`モデルのデフォルトのファクトリを指します。
4. `expect(user).to be_valid`：この行では、userが有効であることを期待しています。`be_valid`はRSpecのマッチャで、オブジェクトが有効であるかどうかを検証します。
5. `expect(user.errors[:name]).to include("can't be blank")`：この行では、`User`オブジェクトのエラーから`name`属性のエラーメッセージが含まれていることを期待しています。エラーメッセージが「can't be blank」が含まれていることを検証しています。

詳しく見ていきましょう。  
- describe：テストのグループ化を宣言します。関連するテストはこのブロックにまとめます。
- context：テストのグループ化を宣言します。こちらは`describe`に比べてもう少し細かくグループ分けします。
- it：テストを単位ごとにまとめる宣言します。`it`内のすべてのエクスペクテーションが成功していればテスト成功です。
- expect：実際のテストの期待結果です。
- to：RSpecが用意しているテスト用のメソッドです。引数のマッチャの結果が真(true)であればテスト成功です。`Minitest::Test`の`assert`に該当します。
- マッチャ：Rspecが用意しているオブジェクトです。期待値と実際の値を比較して、その結果を返します。以下がマッチャの代表例です。

| マッチャ一覧                                | 結果                                                                                                                                                                                           | 
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | 
| eq val<br>                                  | 「expect」内の値がvalと等しいならば真(true)<br>                                                                                                                                                | 
| be_メソッド名<br>                           | 「expect.メソッド名?」の結果を返す<br>メソッド名が「?」で終わる、戻り値がtrueかfalseのメソッドの検証にのみ使用できる<br>                                                                       | 
| be_truthy                                   | 「expect」内の値がRuby的に真なら真(true)を返す<br>例：expect(1).to be_truthy→テスト成功<br>　　expect(nil).to&nbsp;be_truthy→テスト失敗<br>                                                  | 
| change{ target }.from(before).to(after)<br> | 「expect」内の式を実行した結果、targetの値がbeforeからafterに更新されていれば真(true)<br>※changeを使用する場合は「expect｛式｝」を使用する<br>　例: expect{a = 1+2}.change{a}.from(nil).to(3) | 
| change{ target }.by(num)<br>                | 「expect」内の式を実行した結果、targetの値がnumの数増えていれば真(true)<br>※numはマイナスも利用できる<br>　例: expect{a = 1-2}.change{a}.by(-1)                                                | 
| redirect_to(val)<br>                        | 「expect」内のレスポンスがvalにリダイレクトしていれば真(true)<br>　例: expect(response).to redirect_to(User.last)<br>                                                                          | 
| have_content(val)<br>                       | 「expect」内の値にvalの値が含まれていれば真(true)主にsystem_testで利用される<br>　例: expect(page).have_content("successfully destroyed.")                                                     | 

Rspecと一緒に使われるツールあライブラリには以下のものがあります。
1. Capybara：ウェブアプリケーションの統合テストを実行するためのフレームワークであり、ブラウザの操作をシミュレートすることができます。Rspecと組み合わせて使用することで、ウェブアプリケーションの挙動をテストすることができます。
2. FactoryBot： テストデータを簡単に作成するためのライブラリであり、モデルのインスタンスを生成するためのシンプルなDSLを提供します。Rspecテスト内でテストデータを簡単に作成するために使用されます。
3. DatabaseCleaner：テスト実行時にデータベースをクリーンアップするためのライブラリであり、テスト間のデータの汚染を防ぎます。Rspecと組み合わせて使用することで、テストが独立して実行されることを保証します。
4. Faker：ダミーデータを生成するためのライブラリであり、テストデータの作成に使用されます。Rspecテスト内でテストデータを作成する際に、リアルなデータをシミュレートするために利用されます。

みなさんも多くのエンジニアと共に今後、開発していくことを考えて今回はRSpecを利用してテストを実装していきます。  

### 7.3.2 RSpecの環境構築

テスト基礎用のアプリケーションを作成しましょう。

```sh
rails new rspec_practice
cd rspec_practice
```

#### 必要なgemのインストール

RSpecを使ったテストの実装環境を整えましょう。RSpecは強力なテスティングツールですが、テストデータの作成には長けていません。もちろん、腕に自信のある人は自作しても良いですが、オープンソースソフトウェア(OSS)の文化では「車輪の再発明」は避ける習慣があります。今回は`rspec-rails`、`factory_bot_rails`、`gimei`の3つを利用して実装します。

- `rspec-rails`  
RSpecをRailsで利用するためのgem
- `factory_bot_rails`  
テストデータ作成の補助をするgem
- `gimei`  
ランダムに日本人の名前を生成するgem

Gemfileにこれらのgemを追加してみましょう。テスティングツールは本番環境では利用しないのでGemfileのグループは`development`と`test`とします。

```rb
# Gemfile

group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri windows ]
  
# ここから追加
  # Use RSpec
  gem 'rspec-rails', '~> 4.0.1'
  # Use FactoryBot
  gem 'factory_bot_rails'
  # Use gimei for generating a Japanese fake name
  gem 'gimei'
# ここまで追加
end
```

```sh
$ bundle install
```

#### RSpec実行環境の初期化

RSpecを実行するにあたって、様々な設定が必要になります。複雑になるので、順を追ってゆっくり説明するので安心してくださいね。

##### RSpec設定ファイルの生成

設定ファイルを作成するために次のコマンドを実行してください。いくつかのフォルダ、ファイルが生成されます。

```sh
$ rails g rspec:install
```

生成されたファイルに次の内容を追加してください。

```rb
#.rspec

--require spec_helper
--format documentation 
```

```rb
# spec/spec_helper.rb

require 'factory_bot_rails' # 追加

・
・
RSpec.configure do |config|
  ・
  ・
# ここから追加
  # Simplify syntax
  config.include FactoryBot::Syntax::Methods
# ここまで追加
end
```

##### Rails設定ファイルの修正

Minitest::TestがRailsでは標準のテスティングツールなのでRSpecを利用するようにしてあげる必要があります。

```rb
# config/application.rb

・
・
module RspecMockups
  class Application < Rails::Application
    ・
    ・
  # ここから追加
    # Don't generate system test files.
    config.generators.system_tests = nil
  # ここまで追加
  end
end
```

`config/initializers/generators.rb`を新規作成します。

```rb
# config/initializers/generators.rb

Rails.application.config.generators do |g|
  g.test_framework :rspec
  g.view_specs false
  g.routing_specs false
  g.helper_specs false
  g.fixture_replacement :factory_bot, dir: 'spec/factories'
end
```

##### testフォルダの削除

前章で説明があったとおり、testフォルダは`Minitest::Test`に関連するファイルがありますが今回はRSpecを利用してテストを実装しますので、削除します。

```bash
rm -r test
```
