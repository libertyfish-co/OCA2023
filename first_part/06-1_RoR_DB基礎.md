
# 6.1 Ruby on Rails：データベース基礎

## 6.1.1 データベースとは

「データベース（Database）」とは、簡単に言えば「データの保管庫」です。日常生活では、電話帳や住所録、図書館の目録などがデータベースの一例です。それらは膨大な情報を整理して効率的に利用するための仕組みと言えます。

### IT分野におけるデータベース
IT分野で「データベース」といえば、「データベース管理システム（DBMS: Database Management System）」を指します。DBMSは、データを安全かつ効率的に保存し、必要なときに素早く取り出せるアプリケーションです。以下のようなシステムに利用されます。

- 顧客管理システム（顧客情報の保存・検索）
- 商品管理システム（在庫や価格情報の管理）
- 受発注管理システム（注文履歴や出荷状況の追跡）

たとえば、ECサイトでは、顧客の名前や住所、注文履歴といったデータがデータベースに保存されています。これにより、迅速に注文を処理したり、顧客ごとにパーソナライズされたサービスを提供できます。

### データベースの重要性
データベースは、現代のソフトウェアシステムにとって欠かせない基盤技術です。正確なデータが効率よく管理されていなければ、業務に支障が出たり、サービスの品質が低下したりすることもあります。そのため、DBMSを用いてデータを安全に管理することが重要です。

> **メモ**: Rubyでは、データベースを扱う際に「ActiveRecord」というライブラリを用いることが一般的です。ActiveRecordは、Ruby on Railsフレームワークに含まれるデータベース操作ツールで、後の章で詳しく解説します。

---

## 6.1.2 データベースの基礎

### データベースの特徴
DBMSには以下のような特徴があります。

1. **膨大なデータの管理**  
   紙媒体では扱いきれない膨大なデータを効率的に管理できます。

2. **高速な検索・操作**  
   数千万件のデータからでも、数秒で必要な情報を検索・更新できます。

3. **同時アクセス**  
   複数のユーザーが同時にデータを利用可能です。

4. **トランザクション管理**  
   データ操作の一貫性を保つ仕組みがあります。これにより、操作中にエラーが発生してもデータの整合性が守られます。

5. **セキュリティ**  
   ユーザーごとにアクセス制限を設けることで、重要なデータを守ります。

### リレーショナル・データベースとは
DBMSにはさまざまな種類があります。このDBMSの種類は一般的に「データモデル」に基づいて分類されて
います。データモデルとは、「データを論理的にどのように扱うのか」を定めた概念であり、代表的なデータ
モデルとして以下のようなものがあります。
- 階層型データモデル
- ネットワーク型データモデル
- リレーショナル･データモデル
- オブジェクト指向データモデル
- XMLデータモデル

このうち、リレーショナル･データモデルを実装したデータベースのことを「RDB（Relational Database
）」あるいは「RDBMS（RDB Management System）」と呼び、現在データベースといえば、一般的にはリレーショナル･データベースのことを指しています。ここではリレーショナルデータベースを中心に説明します。


### データベースの構造
データベースは、データを表形式で管理します。この表を「テーブル」と呼び、行（レコード）は1つのデータの塊、列（カラム）はそのデータの項目を表します。

例えば、顧客情報を管理するテーブルは以下のようになります。

![MVC](./images/06-1_RoR_DB設計/6-1-1.png)

| ID  | 名前     | メールアドレス         | 登録日         |
|-----|----------|------------------------|----------------|
| 1   | 山田太郎 | taro@example.com      | 2025-01-01     |
| 2   | 鈴木花子 | hanako@example.com    | 2025-01-05     |

- **行（レコード）**: 各行が1つの顧客を表します。
- **列（カラム）**: 各列がデータの属性（名前、メールアドレスなど）を表します。


## 6.1.3 ActiveRecord

### ActiveRecordとは

ActiveRecordは、Railsに付属する重要なライブラリの1つで、MVCのM(モデル)に相当します。  
ActiveRecordは、ORM (Object-Relational Mapping) で実装されています。

### ORMとは
ORM (Object-Relational Mapping) とは、アプリケーションが持つオブジェクトとリレーショナルデータベース(RDBMS)を繋ぐプログラミング技法です。  
ActiveRecordを利用することで、データベースのテーブルをRubyのオブジェクトとしてモデル化し、データベース操作をオブジェクト指向の方法で行うことができます。

### ActiveRecordの主な機能

- **データベースのテーブルをモデル化**: テーブルと対応するクラスを簡単に定義。
- **テーブル間の関連性の表現**: 関連するデータの取得や保存が容易に。
- **データの検証**: バリデーションルールをモデル内に定義することで不正データを防止。
- **オブジェクト指向によるデータベース操作**: Rubyのコードで直感的にデータベース操作を記述可能。

以下はActiveRecordの特に重要な仕組みです:
- モデルとそのデータを表す仕組み
- モデル間の関連性を表す仕組み
- 関連するモデルを通した階層の継承を表す仕組み
- DBに保存する前に検証する仕組み
- オブジェクト指向の手法でDB操作を実行する仕組み

## 6.1.4 SQLとActiveRecordの比較
データベースを操作するにはSQLが必要ですが、ActiveRecordを使用することでRubyのコードで操作できます。

### SQLでの例
以下のSQL文は、データベースから「太郎」さんを取得するものです。
```sql
SELECT * FROM users WHERE name LIKE '%太郎%';
```

### ActiveRecordでの例
ActiveRecordを使用すると、同じ操作を以下のように記述できます。
```ruby
User.where("name LIKE ?", "%太郎%")
```

このように、ActiveRecordを使えば直感的かつ簡潔なコードでデータベース操作が可能です。

### 詳細な説明
#### SQLの仕組み
SQL (Structured Query Language) は、データベースに対する命令を記述するための言語です。  
上記のSQL文は、以下のような意味を持ちます:
- **SELECT**: データを取得する。
- **\***: すべての列を選択する。
- **FROM users**: "users" テーブルからデータを取得する。
- **WHERE name LIKE '%太郎%'**: 名前に「太郎」を含むデータを条件として指定する。

SQLは柔軟で強力なツールですが、特定のデータベース構造に依存しやすく、記述が煩雑になりがちです。

#### ActiveRecordの仕組み
ActiveRecordでは、Rubyコードとして同様の操作を表現できます。  
上記のActiveRecordコードでは、次のような内容を表しています:
- **User**: "users" テーブルに対応するモデルクラス。
- **where**: 条件を指定してデータを取得するメソッド。
- **"name LIKE ?"**: 検索条件を指定するSQL文を引数として渡す。
- **"%太郎%"**: LIKE検索の条件値。

### ActiveRecordの直感的な利便性
ActiveRecordを使用すると、以下の利便性が得られます:
1. **コードの簡潔さ**: SQL文に比べて記述が短く、読みやすい。
2. **データベース間の移植性**: データベースの種類に依存しにくい設計。
3. **安全性の向上**: パラメータバインディングによりSQLインジェクションのリスクを軽減。

このように、ActiveRecordは初心者でも扱いやすいツールとなっています。


##  6.1.5 ActiveRecordの代表的なメソッド

ここでは、以下のモデルを例にActiveRecordの代表的なメソッドを簡潔に説明していきます。

`モデル User:ユーザー`

| field名      | 名称           | 型      |
| :----------- | :------------- | :------ |
| id           | ID             | integer |
| name         | 名前           | string  |
| mail_address | メールアドレス | string  |

`ActiveRecordの代表的なメソッド一覧`  
※表の発行SQLは、わかりやすさのため、テーブル名は省略してカラム名のみ記述しています。

| メソッド | 説明                 | 使用例                                                 | 発行SQL                                                                                       |
| :------- | :------------------- | :----------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| all      | 全件取得(全カラム)   | User.all                                               | SELECT * FROM users                                                                           |
| select   | 全件取得(カラム指定) | User.select(:name)<br>User.select('name,mail_address') | SELECT name FROM users<br>SELECT name,mail_address FROM users                                 |
| find     | 検索(id指定)         | User.find(1)                                           | SELECT * FROM users WHERE id = 1                                                              |
| find_by  | 検索(条件指定)       | User.find_by(id:1)<br>User.find_by('id > 1')           | SELECT \* FROM users WHERE id = 1 LIMIT 1<br>SELECT * FROM users WHERE id > 1 LIMIT 1         |
| where    | 検索(条件指定)       | User.where(id:1)<br>User.where('id > 1')               | SELECT \* FROM users WHERE id = 1<br>SELECT * FROM users WHERE id > 1                         |
| first    | 最初のデータをとる   | User.first<br>User.first(2)                            | SELECT \* FROM users ORDER BY id ASC LIMIT 1<br>SELECT \* FROM users ORDER BY id ASC LIMIT 2  |
| last     | 最後のデータをとる   | User.last<br>User.last(2)                              | SELECT \* FROM users ORDER BY id DESC LIMIT 1<br>SELECT * FROM users ORDER BY id DESC LIMIT 2 |
| order    | ソート               | User.order(:name)<br>User.order(name: :DESC)           | SELECT \* FROM users ORDER BY name ASC<br>SELECT \* FROM users ORDER BY name DESC             |
| limit    | 制限                 | User.limit(2)                                          | SELECT * FROM users LIMIT 2                                                                   |

`各メソッドの詳細`

- all
  - レコードを全件取得します
- select
  - カラムを指定し、レコードを取得します。引数の値がカラムとなります。
- find
  - 指定したidのレコードを取得します。引数の値が指定するidとなります。
  - findは、該当するデータが見つからない場合は例外（RecordNotFound）が発生します。
- find_by
  - 特定のカラムの条件を指定し、該当する1件を取得します。引数の値が条件となります。
  - find_byは該当するデータが見つからない場合は、nilを返します。
- where
  - 特定のカラムの条件を指定し、該当する全件を取得します。引数の値が条件となります。
  - whereは、該当するデータが見つからない場合は空の`ActiveRecord::Relation`を返します。
- first
  - レコードの最初の1件を取得します。引数を渡すと最初のn件と指定することもできます。
- last
  - レコードの最後の1件を取得します。引数を渡すと最後のn件と指定することもできます。
- order
  - レコードを引数に指定したカラムで並び変えます。デフォルトの並び順はASC(昇順)になっています。
  - 降順で並び変える場合は`User.order(name: :DESC)`とします。
- limit
  - 特定のレコード件数を取得します。引数の値が最大取得行数となります。

`etc`
- ActiveRecord::Relationについて
  - `ActiveRecord::Relation`とは、モデルオブジェクトのコレクションです。  
  `ActiveRecord::Relation`を返す場合は、メソッドチェーンが可能ですので、  
  例えばwhereに続いてさらにActiveRecordのメソッドを使用する事ができます。  
  ここでは詳細を説明しませんので、気になる場合は調べてみましょう。

- ?の使い方
  - 条件に変数を使用したい場合等に、?(プレースホルダ)を使用します。  
  変数でなくても、直接値を指定することも可能です。  

  ```rb
  name = "test"
  User.where("name = ?", name)
  #SELECT  "users".* FROM "users" WHERE (name = 'test')

  User.where("name = ?", 'test2')
  #SELECT  "users".* FROM "users" WHERE (name = 'test2')
  ```

  ## まとめ

ActiveRecordは、Ruby on Railsにおけるデータベース操作の基盤となるライブラリであり、初心者にとっても直感的に利用できる機能が豊富に備わっています。  
SQLの基礎を理解することで、ActiveRecordをさらに効果的に活用できるため、両方の知識を学んでおくことをおすすめします。


### 6.1.5 例題

先生と授業(科目)のテーブルを作成し、先生と授業の多対多で関連付けしていきます。
先生は複数の授業(科目)を担当し、授業(科目)からも複数の先生が受け持っているという多対多の関連付けとして実装します。
データベース構成は以下の通りとします。

Teacher : 先生テーブル

| field名 | 名称 | 型     |
| :------ | :--- | :----- |
| name    | 名前 | string |

Lesson : 授業テーブル

| field名 | 名称   | 型     |
| :------ | :----- | :----- |
| name    | 授業名 | string |

TeacherLesson : 中間テーブル

| field名    | 名称 | 型         |
| :--------- | :--- | :--------- |
| teacher_id | 先生 | references |
| lesson_id  | 授業 | references |

![画像](images/06-3.png)

#### ①　アプリケーションの作成

まずは、例題用のRailsアプリケーションを作成します。

```sh
$ rails new teacher_sample
```

#### ②　Teacherモデルの作成

次に必要なモデルを作成してきます。  
今回はCRUD操作をしないので`rails g model`でモデルを作成します。  

```sh
$ rails g model Teacher name:string
```

#### ③　Lessonモデルの作成

```sh
$ rails g model Lesson name:string
```

#### ③　TeacherLessonモデルの作成

中間テーブルとなるモデルです。
teacherとlessonを参照するように設定して、モデルを生成します。

```sh
$ rails g model TeacherLesson teacher:references lesson:references
```
データ型を`references`にすることで中間テーブルで自動的に関連付けを定義してくれます。  
teacher、lessonとteacher_lessonのモデルが作成できたので、テーブルを作成するためにマイグレーションも実行しましょう。

```sh
$ rails db:migrate
```

各モデルの関係を設定するために、以下の内容を追記してください。  
has_many :throughは、teacher_lessonをショートカットして、teacherもしくはlessonを参照できるようにします。

```rb
# app/models/teacher.rb

class Teacher < ApplicationRecord
  has_many :teacher_lessons
  has_many :lessons, through: :teacher_lessons
end
```

```rb
# app/models/lesson.rb

class Lesson < ApplicationRecord
  has_many :teacher_lessons
  has_many :teachers, through: :teacher_lessons
end
```

teacher_lessonモデルはteacherとlessonを参照するように生成したため、既に下記のソースコードとなっています。

```rb
# app/models/teacher_lesson.rb

class TeacherLesson < ApplicationRecord
  belongs_to :teacher
  belongs_to :lesson
end
```

### 6.1.6 問題

今回のECサイトのデータベース構成は、以下のようにします。  
この構成通りに、モデルを作成してみましょう。BookとTagはscaffold、Taggingはmodelで作成しましょう。
また、各モデルの関係を設定しましょう。本と商品は多対多の関係になることに注意して下さい。  


アプリケーション名：ec_site  
Book : 本 テーブル

| field名      | 名称     | 型      |
| :----------- | :------- | :------ |
| title        | タイトル | string  |
| author       | 著者     | string  |
| published_on | 出版日   | date    |
| showing      | 商品表示 | boolean |
| price        | 価格     | integer |

Tag : 商品タグ テーブル

| field名 | 名称   | 型     |
| :------ | :----- | :----- |
| name    | タグ名 | string |

Tagging : タグ付け テーブル

| field名 | 名称     | 型         |
| :------ | :------- | :--------- |
| book_id | 本       | references |
| tag_id  | 商品タグ | references |
