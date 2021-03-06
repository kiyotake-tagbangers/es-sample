```shell
# --------------------------------
### 基本操作

# インデックスの作成
PUT /reviews

GET /reviews

# インデックスの削除
DELETE /reviews

# mapping の定義(RDS でいうスキーマのイメージ)
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": {
        "type": "float"
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

GET /reviews/_mapping

DELETE /reviews

# 事前にスキーマを定義せずにドキュメントを登録
POST /reviews/_doc/
{
  "rating": 4,
  "author": {
    "first_name": "Tagbang",
    "last_name": "Taro"
  },
  "comment": "this is awesome"
}

# ドキュメントの型から推論して mapping が定義される
# 文字列は text型とkeyword型のマルチフィールドとして定義される
GET /reviews/_mapping

# 型の情報が違うため登録できない
POST /reviews/_doc/
{
  "rating": "dummy",
  "author": {
    "first_name": "Tagbang",
    "last_name": "Taro"
  },
  "comment": "this is awesome"
}

# PUT では id 明示的に指定して作成
PUT /reviews/_doc/1
{
  "rating": 5,
  "author": {
    "first_name": "Yamada",
    "last_name": "Jiro"
  },
  "comment": "this is excellent!"
}

GET /reviews/_doc/1

# match は全文検索用のクエリ
# 転置インデックスを利用して検索
GET /reviews/_search
{
  "query": {
    "match": {
      "comment": "this"
    }
  }
}

# 条件を増やしたことで _score の値が増えた(より関連度が高い)
GET /reviews/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "comment": "this"
          }
        },
        {
          "range": {
            "rating": {
              "gte": 3
            }
          }
        }
      ]
    }
  }
}

# filter を使用する場合は _score には影響を及ぼさない
# クエリキャッシュがあるため、単に検索範囲を限定したい(=スコアに関連しないクエリ)なら filter を使う
GET /reviews/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "comment": "this"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "rating": {
              "gte": 3
            }
          }
        }
      ]
    }
  }
}

# term は完全一致の検索用のクエリ
# keyword型のフィールドを検索
GET /reviews/_search
{
  "query": {
    "term": {
      "comment.keyword": {
        "value": "this is awesome"
      }
    }
  }
}

# --------------------------------
### SQL との比較

# select account_number form bank where employer in ("Pyrami","Anocha","Reversus")
# terms は完全一致で複数の値を指定できる
GET /bank/_search
{
  "_source": [
    "account_number",
    "email"
  ],
  "query": {
    "terms": {
      "employer.keyword": [
        "Pyrami",
        "Anocha",
        "Reversus"
      ]
    }
  }
}

# 複合クエリ(compound queries)
# select email from product where age = 23 and (balance >= 40000 or city = "TX") order by balance DESC limit 3

# must句は「必ず含ままれているべき」クエリ条件
# bool は内でクエリを組み合わせられる
# should句は or 条件
# range は値の範囲検索
GET /bank/_search
{
  "_source": "email",
  "query": {
    "bool": {
      "must": [
        {
          "term": {
          "age": {
            "value": 23
          }
        }
        },
        {
          "bool": {
            "should": [
              {
                "range": {
                  "balance": {
                    "gte": 40000
                  }
                }
              },
              {
                "term": {
                  "city.keyword": {
                    "value": "TX"
                  }
                }
              }
            ]
          }
        }
      ]
    }
  },
  "sort": [
    {
      "balance": {
        "order": "desc"
      }
    }
  ],
  "size": 3
}

# --------------------------------
### データの集計

# 集計結果は hits ではなく、 aggregations のフィールドに返ってくる
# 検索結果は hits に返ってくるので不要なら size 0 にする

GET /bank/_search

# select avg(balance) as balance_avg from bank
GET /bank/_search
{
  "aggs": {
    "balance_avg": {
      "avg": {
        "field": "balance"
      }
    }
  },
  "size": 0
}

# select max(balance) as max_balance from bank
GET /bank/_search
{
  "aggs": {
    "max_balance": {
      "max": {
        "field": "balance"
      }
    }
  },
  "size": 0
}

# select * from bank group by age
# terms aggeration を使用
GET /bank/_search
{
  "aggs": {
    "group_by_age": {
      "terms": {
        "field": "age",
        "size": 3
      }
    }
  },
  "size": 0
}

# ------------------------
### 全文検索

# いずれかの単語を含んでいればヒット
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "Recipes with pasta or spaghetti"
    }
  }
}

# pasta,or,spaghetti を含んでいればヒット
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta or spaghetti",
        "operator": "and"
      }
    }
  }
}

# フレーズで一致するか確認
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "spaghetti puttanesca"
    }
  }
}
# 順番を変えると今回のデータだとヒットしない
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "puttanesca spaghetti"
    }
  }
}

# 複数のフィールドを対象に検索
GET /recipe/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": ["title", "description"]
    }
  }
}

# ------------------------
### analyzeの利用

# standard analyzer の使用
POST /_analyze
{
  "text": "2 guys walk into    a bar, but the third ...DUCKS! :-)",
  "analyzer": "standard"
}

# character filter,tokenizer, token filter を明示的に書いた場合
POST /_analyze
{
  "text": "2 guys walk into    a bar, but the third ...DUCKS! :-)",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}

# simple analyzer(なにしてたっけ)
POST /_analyze
{
  "text": "Is that Mike's cute-looing dog?",
  "analyzer": "simple"
}

# keyword analyzer
# text field が single term として保存される
# email, status などで使われる
POST /_analyze
{
  "text": ["2 guys walk into a bar, but the third ... DUSKS"],
  "analyzer": "keyword"
}

# ------------------------
# custom analyzer の利用
# html のタグを取り除く
POST /_analyze
{
  "char_filter": ["html_strip"],
  "text": "I'm so <em>sleepy</em> and I <strong>hungry</strong>!"
}

DELETE /analyzer_test
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop"
          ]
        }
      }
    }
  }
}

# stop word である and が消えている
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I'm so <em>sleepy</em> and I <strong>hungry</strong>!"
}

# ------------------------
### join の利用
DELETE /join/

# mapping(RDBでいうスキーマのようなもの)を定義
# question is parent of answer
PUT /join?pretty
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "question": "answer"
        }
      }
    }
  }
}

GET /join/_mapping

# question documet(親ドキュメント)
# refresh により変更内容が検索できるタイミングを制御する
PUT /join/_doc/1
{
  "text": "This is a question",
  "my_join_field": {
    "name": "question"
  }
}

# 親ドキュメントはリレーション名を記載するだけでもいい
PUT /join/_doc/2
{
  "text": "This is another question",
  "my_join_field": "question"
}

GET /join/_search/

# 子ドキュメント
# リレーション名と親ドキュメントのIDを指定する
# routing=1 は親子のドキュメントを同じシャードでインデクシングするため
PUT /join/_doc/3?routing=1
{
  "text": "This is an answer",
  "my_join_field": {
    "name": "answer",
    "parent": "1"
  }
}

PUT /join/_doc/4?routing=1
{
  "text": "This is another answer",
  "my_join_field": {
    "name": "answer",
    "parent": "1"
  }
}

GET /join/_search/

# parent id で絞れる
GET /join/_search
{
  "query": {
    "parent_id":{
      "type": "answer",
      "id":1
    }
  }
}

# 紐づく answer がない
GET /join/_search
{
  "query": {
    "parent_id":{
      "type": "answer",
      "id":2
    }
  }
}

# terms は group_by
# parent id で集約(aggregate)する
GET /join/_search
{
  "aggs": {
    "parents": {
      "terms": {
        "field": "my_join_field#question",
        "size": 10
      }
    }
  },
  "size": 0
}

# ------------------------
### store の利用
# 特定のフィールドだけを取得した時に有効
DELETE store-test

PUT store-test
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "store": true
      },
      "date": {
        "type": "date",
        "store": true
      },
      "content": {
        "type": "text"
      }
    }
  }
}

GET store-test/_mapping

PUT store-test/_doc/1
{
  "title":   "Some short title",
  "date":    "2015-01-01",
  "content": "A very long content field..."
}

GET store-test/_search
{
  "stored_fields": [ "title", "date" ]
}

GET store-test/_search
# ------------------------
### nested_object の利用

DELETE /my_index

# 通常の Object型
PUT my_index/_doc/1
{
  "group": "fans",
  "users": [
    {
      "first_name": "John",
      "last_name": "Smith"
    },
    {
      "first_name": "Alice",
      "last_name": "White"
    }
  ]
}

# フラットにマピングされるため、組み合わせ違いでもヒットしてしまう
GET my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "users.first_name": "Alice"
          }
        },
        {
          "match": {
            "users.last_name": "Smith"
          }
        }
      ]
    }
  }
}

DELETE my_index

PUT my_index
{
  "mappings": {
      "properties": {
        "users": {
          "type": "nested"
        }
      }
    }
}

PUT my_index/_doc/1
{
  "group": "fans",
  "users": [
    {
      "first_name": "John",
      "last_name": "Smith"
    },
    {
      "first_name": "Alice",
      "last_name": "White"
    }
  ]
}

# nested type になっている
GET my_index/_mapping

# 組み合わせ違いはヒットしない
GET my_index/_search
{
  "query": {
    "nested": {
      "path": "users",
      "query": {
        "bool": {
          "must": [
            { "match": { "users.first_name": "Alice" }},
            { "match": { "users.last_name":  "Smith" }}
          ]
        }
      }
    }
  }
}

# 組み合わせが同じなため、ドキュメントが返される
GET my_index/_search
{
  "query": {
    "nested": {
      "path": "users",
      "query": {
        "bool": {
          "must": [
            { "match": { "users.first_name": "Alice" }},
            { "match": { "users.last_name":  "White" }}
          ]
        }
      }
    }
  }
}

GET my_index/_search
{
  "query": {
    "nested": {
      "path": "users",
      "query": {
        "bool": {
          "must": [
            { "match": { "users.first_name": "Alice" }},
            { "match": { "users.last_name":  "White" }}
          ]
        }
      },
      "inner_hits": {
        "highlight": {
          "fields": {
            "user.first": {}
          }
        }
      }
    }
  }
}
# ------------------------
### Alias

# alias field type

DELETE /reviews

# filed を alias で検索できるようにする
# 利用ケース
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

PUT /order/_doc/1
{
  "purchased_at": "2016-12-26T08:29:47Z",
  "lines": [
    {
      "product_id": 2,
      "amount": 98.87,
      "quantity": 3
    }
  ],
  "total_amount": 98.87,
  "salesman": {
    "id": 5,
    "name": "Carlyn Stegers"
  },
  "sales_channel": "web",
  "status": "cancelled"
}
```
