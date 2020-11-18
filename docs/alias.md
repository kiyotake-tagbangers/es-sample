# alias

## [index aliases](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/indices-aliases.html)

- インデックスに別名を決める、同じエイリアスを複数のインデックスにつけることもできる

- 検索はエイリアス宛に行う

- あらかじめ絞り込んでおくフィルター機能とドキュメントの物理的な配置シャードをコントロールするルーティング機能が提供

- 活用方法
    - 任意のタイミングで新しいインデックス情報を追加
        - 商品カテゴリごとにインデックスを作成する際に、新しいカテゴリの商品を別でインデックスしておき、任意のタイミングで一気に公開
    - [migration](https://www.coenterprise.com/blog/how-to-perform-an-elasticsearch-index-migration-using-aliases/)

```shell
# index alias
# Elasticsearch のアップデート時にインデックスを移行する時などに利用するケースがある
POST /_aliases
{
  "actions" : [
    { "add" : { "index": "reviews", "alias" : "evaluations" } }
  ]
}

# index alias の一覧の取得
GET /_alias/evaluation

GET /reviews/_doc/1
GET /evaluations/_doc/1
```

## [alias field type](https://www.elastic.co/guide/en/elasticsearch/reference/current/alias.html)

- インデックス内の field の代替名を定義する
    - 検索リクエストのターゲットフィールドの代わりに使用できる

- 書き込みに対しては、 field alias は指定できない

```shell
DELETE /reviews

# filed を alias で検索できるようにする
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": {
        "type": "float"
      },
      "evaluation": {
        "type": "alias",
        "path": "rating"
      },
      "author":{
        "properties": {
          "first_name":{
            "type": "text"
          },
          "last_name":{
            "type": "text"
          }
        }
      }
    }
  }
}

PUT /reviews/_doc/1
{
  "rating": 5,
  "author": {
    "first_name": "Yamada",
    "last_name": "Jiro"
  },
  "comment": "this is excellent!"
}

GET /reviews/_search
{
  "query": {
    "range": {
      "rating": {
        "gte": 3
      }
    }
  }
}

# alias でも検索ができる
GET /reviews/_search
{
  "query": {
    "range": {
      "evaluation": {
        "gte": 3
      }
    }
  }
}
```
