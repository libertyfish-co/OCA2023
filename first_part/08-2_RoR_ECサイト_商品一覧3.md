
## 8.2 Ruby on Rails：ECサイトの開発 商品一覧3

### 8.2.1 詳細画面への遷移と削除

今回は先ほど作成した`Post`アプリに詳細画面と削除機能を実装します。  
まずは詳細画面を実装していきます。  

ルーティングの設定をしましょう。  
```rb
# config/routes.rb

Rails.application.routes.draw do
  get 'posts/:id', to: 'posts#show', as: 'post' # 追加
  get 'posts/new', to: 'posts#new', as: 'new_post'
  get 'posts', to: 'posts#index'
  get 'posts/:id/edit', to: 'posts#edit', as: 'edit_post'
  post 'posts', to: 'posts#create'
  patch 'posts/:id', to: 'posts#update', as: 'update_post'
end
```

これでルーティングの設定が完了しましたが、次の削除機能にも備えてまとめてしまいましょう。

```rb
# config/routes.rb

Rails.application.routes.draw do
  root 'posts#index'
  resources :posts
end
```

今回はroot_pathを設定しています。root_pathとは、トップページにあたるところを表示します。  
<http://localhost:3000>にアクセスしてみましょう。  
一覧画面が表示されたでしょうか。では、コントローラを設定していきましょう。  

```rb
class PostsController < ApplicationController
  def index
    @posts = Post.all
  end

  def show # 追加
    @post = Post.find(params[:id]) # 追加
  end # 追加
  ・
  ・
end
```

前回に実装したControllerに新たにshowアクションを追加しました。さらに編集します。

```rb
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update] # 追加

  def index
    @posts = Post.all
  end

  def show
  end
  ・
  ・
  def edit
  end
  ・
  ・
  def update
    if @post.update(post_params)
      redirect_to posts_path, notice: '編集が完了しました。'
    else
      render :edit
    end
  end

  private
  def set_post # 追加
    @post = Post.find(params[:id]) # 追加
  end # 追加
  ・
  ・
end
```

`before_action`を追加しました。`before_action`はアクションが実行される前に実行されるメソッドです。今回は`:show`と`:edit`、`:update`メソッドの前に実行されます。　　
`private`メソッドにある`set_post`メソッドでは投稿のIDを使用して該当する投稿を取得し、@post インスタンス変数に格納します。共通する動作はメソッドとしてまとめて`before_action`にすることでアクション前に実行したいメソッドはこのようにまとめることができます。  

`show.html.erb`を新規作成しましょう。  
```html
<!-- app/views/posts/show.html.erb -->

<h2>投稿詳細</h2>
<p><%= @post.content %></p>
```

<http://localhost:3000/posts/1>にアクセスしてみましょう。  
IDが1の投稿が存在していれば表示されます。


詳細
削除
select