## 5.5 Ruby on Rails：Gemとは

Railsは様々な Gem が集まったものです。  
Gem とは、Ruby で作られたライブラリやプログラムのパッケージのことです。  
Ruby や Rails の機能を拡張するために使われ、Gem を利用することで、開発を効率的に進めることができます。  
どれくらい開発効率が上がるのかを、Gem を使ったログイン機能（devise）で見てみましょう。

devise を使わずにログイン機能を作った場合（コントローラの処理）
```rb
class SessionsController < ApplicationController
  def new
  end

  def create
    user = User.find_by(email: params[:session][:email])

    if user && user.password == params[:session][:password]
      session[:user_id] = user.id
      redirect_to root_path, notice: "ログインしました"
    else
      flash.now[:alert] = "メールアドレスまたはパスワードが間違っています"
      render :new
    end
  end

  def destroy
    session[:user_id] = nil
    redirect_to login_path, notice: "ログアウトしました"
  end
end
```

deviseを使った場合（コントローラの処理）
```rb
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  private

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
  end
end
```
このようにGemを使うことで、実装をより簡単にし、開発のスピードを各段に上げてくれます。

Gem の便利さは分かりましたが、Gem を使うにあたって README を読んで使い方を学び、Gem のソースコードを読むことで、さらに深い理解が得られ、もしそのGemの機能で問題が発生したときに対応が容易になります。  
deviseのソースコード：https://github.com/heartcombo/devise  
また、Gem のバージョン管理、依存関係を解消してくれる bundler の存在も欠かせません。簡単に bundler と Gem の構成を確認してみましょう。

### 5.5 まとめ

* Gemを利用することで、機能実装をより簡単にすることができます。

* Gemを使うときは、そのGemのREADMEなどで使い方を学び、さらに深い理解を得たいときはソースコードを見ましょう！

### 5.5.1 bundler について

Gem を使う上ではバージョン管理と依存関係について、注意が必要です。  
Gemfile の Gem とインストールされた Gem のバージョンが違ったり、Gem 同士が
どちらかを必要としているときに Gem がインストールされていないときに問題が発生します。  
そんな時に、 bundler は、Gem のバージョン管理と依存関係を解決した環境を提供してくれるとても便利なツールです。  
bundler自体もGemとして提供されています。

```sh
$ gem install bundler
```
でbundlerをインストールします。  
あとは、`Gemfile`で追加したいGemを用意して
```sh
$ bundle install
```
でGemfile.lockを作成してくれます。  
Gemfile.lockとは、Gemfileをもとに実際にインストールされたgemの一覧とバージョンが記載されたファイルです。  
複数人で開発を進める場合、Gemfile.lockの状態を一致させることで、すべての開発者が一致した環境で作業でき、デプロイの際に、予期せぬGemでの不具合を避けることができます。  
bundlerに関して、以下の2点は押さえておきましょう。

#### bundle execコマンド
例えば、料理を作るときに特定のレシピを使いたいときがあるとします。  
しかし、キッチンの中には同じ名前の材料が複数あり、形の悪いものや腐ったものがあるかもしれません。  
間違った食材を使ってしまったら、味や質に影響が出ます。  
そこで`bundle exec`を使うことで正しい材料を選択することができます。  
ターミナル等でrspecを実行するとき、下記のとおりコマンドの前に書いて実行すると、Gemfile.lockに沿ったバージョンのrspecを実行してくれます。
```sh
$ bundle exec rspec
```
railsコマンドを実行するときも間違った材料を選択することが起きやすいので、気をつけてください。  
bundlerでインストールしたGemに対するものは、`bundle exec`で実行するようにして、意図したGemが使えるようにしましょう。

#### Gemfileの書き方
`Gemfile`にはGemの名前はもちろん、いろいろな情報を設定することができます。以下の項目を参考に、関わっているプロジェクトのGemfileをチェックしてみましょう。  
書き方の一部を紹介します。  

- バージョン
  1. `>=`：以上(`>= 1.0.0`となっていれば1.0.0以上のすべての範囲)
  2. `<`：未満(`< 1.0.0`となっていれば1.0.0未満のすべての範囲)
  3. `~>`：推定範囲(`~> 1.5`となっていれば1.5～2.0の範囲、`~> 1.5.3`となっていれば1.5.3～1.5.6の範囲)
- 参照先
  1. source：Gemを取得するためのリモートリポジトリを指定
  2. git：GemをGitリポジトリから直接取得する場合に使用
  3. path：ローカルファイルシステム上のパスからGemを取得する場合に使用
- グループ設定
  1. development：開発環境でのみインストール
  2. test：テスト環境でのみインストール
  3. production：本番環境でのみインストール

また、Rubyのバージョンを明記しておくこともできます。記入しておくとログにワーニングが出力されて間違いも防げますので、最近は設定しておくプロジェクトが多いです。

そのほかのbundlerの機能については、マニュアルを参照ください。シンプルで読みやすい英語ですので、

下記のReference(Primary Commands,Utilities)は一通り目を通すことをおすすめします。  

<http://bundler.io/docs.html>  

### 5.5.1 まとめ
* bundler は、Gem の**バージョン管理と依存関係を解決した環境**を提供してくれるとても便利なツールです。

* Gemfile.lockとは、実際にインストールされたgemの一覧とバージョンが記載されたファイルです。  
Gemfile.lockをメンバーで一致させておき、デプロイ時のGemでのエラーを未然に防ぎましょう！

* `bundle exec`コマンドでGemfile.lock通り（正しいバージョン）にコマンドを実行してくれます。

* Gemfileには、Gemの名前とバージョンを書き、明示的にバージョンを指定しましょう！

### 5.5.2 Gemの構成とコードリーディング

#### Gemの構成
最近は、`bundler gem`コマンドでGemを作成するのがデファクトスタンダードになってきましたので、Gemの構成はだいたい以下のとおりです。
```
gem_name/
├.git/
├bin/
├lib/
├test/
├.gitignore
├Gemfile
├LICENSE.txt
├Rakefile
└gem_name.gemspec
```
Gem自体もRailsアプリと同様にgitで管理し、テストも実装されています。特徴としては、Gemのライセンスを示すファイル(上記の場合は`LICENSE.txt`)とそのGemについての情報がまとめられている`*.gemspec`があります。Gemをrubygems.orgへ公開したとき、この`*.gemspec`の内容が詳細ページの情報として使用されます。Gemのメインのコードはlibフォルダに保存されています。

#### コードリーディング
みなさんがRubyで何かつまづいたとき、どうやって解決しますか?ほとんどの場合は検索したり、RubyやRailsのマニュアルを読んだりすると思います。これからはもう一歩進んで、コードを読んで解決してみてください。

コードリーディングのメリットは以下のとおりです。

- RubyやRailsの知識が広がり、理解が深まる
- READMEやマニュアルなどに記載されていない機能がわかる
- 正確な情報を得られる(ただし、まれにREADMEなどが間違っていることもある)
- コード内のコメントのほうがREADMEより分かりやすい場合がある
- 英語で解説されたブログを読まなくても最新情報が得られる

プログラミングを長く続けていると、どうしても慣れた書き方に偏っていきます。知識を広げるにはコードリーディングが一番です。

ここで、ActiveSupportの機能を1つ紹介します。  
ActiveSupportとは、`String`や`Integer`、`Date`などのよく使うクラスのメソッドを拡張してくれる役割を持っています。  
紹介するコードは以下のところにあります。初めはGithubサイト上で見るのが一番見やすいかもしれません。RubyのバージョンやActiveSupportのバージョンは、みなさんが使っているものを参照してください。

少し古いコードになりますが、以下のURLを開いてください。

- github: https://github.com/rails/rails/blob/5-1-stable/activesupport/lib/active_support/core_ext/numeric/conversions.rb

ファイルを開いてすぐにわかると思いますが、`.to_s`メソッドに引数が付いています。機能の拡張です。当然、純粋なRubyにはこのような機能はありません。

```sh
$ irb
irb(main):001:0> 12345678.to_s(:phone)
TypeError: no implicit conversion of Symbol into Integer
        from (irb):3:in `to_s'
        from (irb):3
        from /Users/hogehoge/.rbenv/versions/2.4.2/bin/irb:11:in `<main>'
```

Railsのアプリケーションがあるディレクトリでコンソールを起動してみましょう。
`rails console`のコマンドを実行すると対話的にコードの実行を行うことができます。  

以下は`rails c`を使用した際に行えることの例です。
1. モデルの作成、読み込み、更新、削除などの操作
2. データベースクエリの実行
3. アプリケーションの設定の確認や変更
4. クラスやメソッドの定義やテスト

開発中のアプリケーションでコードやデータベースにアクセスすることができます。  
動作の確認などに有効的なので役に立ちます。覚えておきましょう。  

```sh
$ rails c
irb(main):001:0> 12345678.to_s(:phone)
=>"1-234-5678"
```

コードを読み進めていくと、`def to_s(format = nil, options = nil)`の中で、`:phone`を指示している部分があります。

```rb
when :phone
  return ActiveSupport::NumberHelper.number_to_phone(self, options || {})
```

どうも、ヘルパーとして`.number_to_phone`というものがあるようですね。興味のある方はヘルパーのコードものぞいてみてください。  

さらにファイルの一番下には、Ruby2.4からBignumがFixnumと一緒になった影響で救済措置が書いてあります。

```rb
# Ruby 2.4+ unifies Fixnum & Bignum into Integer. 
if 0.class == Integer
  Integer.prepend ActiveSupport::NumericWithFormat
else
  Fixnum.prepend ActiveSupport::NumericWithFormat
  Bignum.prepend ActiveSupport::NumericWithFormat
end
```
このように、シンプルなファイルを1つ見るだけでも新しい発見や疑問が出てくると思います。最後に、呼び出すメソッドがどこにあるかを示すメソッドを紹介しておきますので参考にしてください。

#### VirtualBoxの場合
```sh
$ rails c
Running via Spring preloader in process 7740
Loading development environment (Rails 6.1.4.6)
irb(main):001:0> 12345678.method(:to_s).source_location
=> ["/home/user/.rbenv/version/2.7.7/lib/ruby/gems/2.7.0/gems/activesupport-6.1.7.4/lib/active_support/core_ext/numeric/conversions.rb", 109]
```

### 5.5.3 よく使われるGemの紹介
以下に業務システムで使いやすいGemをあげます。基本的にはGithubにREADMEとコードがありますので、気になるものはどんどん試していってください。(順不同)

#### __デバッグ系__
- gem 'pry' # irb(interactive ruby shell)代替&拡張:ハイライトなどでコードを見やすくする。プラグインでさらに拡張可能
- gem 'awesome_print' # pp(pretty print)拡張:irbなどでコードを見やすくする
- gem 'hirb' # irb拡張:DBテーブルの内容をASCIIテーブルで表示するなど
- gem 'letter_opener' # 擬似メール送信:メールサーバーの代わりにブラウザへメール内容を表示
- gem 'simplecov' # コードがどれだけテストされているかを示す割合を分析

#### __デザイン系__
- gem 'html2slim' # htmlからslimへの変換
- gem 'html2haml' # htmlからhamlへの変換
- gem 'kaminari' # ページネーション

#### __認証・権限系__
- gem 'devise' # 認証設定
- gem 'koala' # Facebook向けライブラリ
- gem 'cancancan' # 権限設定
- gem 'pundit' # ポリシー設定

#### __インフラ系__
- gem 'net-ssh' # ssh接続
- gem 'whenever' # cron作成
- gem 'dotenv' # 環境変数管理
- gem 'sidekiq' # jobキュー:ActiveJobのバックエンド

#### __ドキュメント系__
- gem 'prawn' # pdf出力(Ruby記述)
- gem 'pdfkit' # pdf出力(html記述)
- gem 'thinreports' # pdf出力(GUI)
- gem 'docx_templater' # docxテンプレート出力
- gem 'axlsx' # xlsx出力
- gem 'rubyzip' # zip出力

##### __その他機能追加__
- gem 'carrierwave' # ファイルアップロード
- gem 'simple_form' # htmlフォーム作成支援

