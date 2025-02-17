# 【06-1 DB設計 6.1.6問題にて】

```sh
$ rails new ec_site
$ rails g scaffold Book title:string author:string published_on:date showing:boolean price:integer
$ rails g scaffold Tag name:string
$ rails g model Tagging book:references tag:references

$ rails db:migrate
$ rails s
```

## ～モデルの関係性の定義～

`model/books`
```rb
has_many :taggings
has_many :tags,through: :taggings
```

`model/tags`
```rb
has_many :taggings
has_many :books,through: :taggings
```

# 【06-1 DB設計 6.1.7練習にて】

注文テーブル
| 注文番号 | 注文日 | ユーザID |
| -------- | ------ | ------ |
| 1 | 2024/4/5 | Y000A |
| 2 | 2024/5/7 | Y000B |
| 3 | 2024/6/1 | Y000C |

注文明細テーブル(中間テーブル)
| 注文番号 | 商品ID | 数量 |
| -------- | ------ | ---- |
| 1 | I000A	| 1 |
| 2 | I000A	| 2 | 
| 2 | I000B	| 1 |
| 3 | I000A	| 3 |
| 3 | I000B	| 2 |
| 3 | I000C	| 1 |

商品テーブル
| 商品ID | 商品名 | 単価 |
| ----- | ------- | --- |
| I000A | 商品名A | 100 |
| I000B | 商品名B | 200 |
| I000C	| 商品名C | 300 |

ユーザテーブル
| ユーザID | ユーザ名 |
| ----- | ------- |
| Y000A | ユーザA |
| Y000B | ユーザB |
| Y000C	| ユーザC |
