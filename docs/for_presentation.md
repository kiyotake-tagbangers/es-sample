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

POST /reviews/_doc/
{
  "rating": "dummy",
  "author": {
    "first_name": "Tagbang",
    "last_name": "Taro"
  },
  "comment": "this is awesome"
}

# PUT では id 明示的にを指定して作成
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

# ドキュメントの型から推論して mapping が定義される
# 文字列は text型とkeyword型のマルチフィールドとして定義される
GET /reviews/_mapping

# match は全文検索用のクエリ
# 転置インデックスを利用して検索
GET /reviews/_search
{"query":{"match":{"comment":"this"}}}

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

# select avg(balance) from bank
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

# select max(balance) from bank
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

# select * from bank group by state limit 2
GET /bank/_search
{
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "size": 2
      }
    }
  },
  "size": 0
}

# select avg(balance) from bank group by state limit 1
GET /bank/_search
{
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "size": 1
      },
      "aggs": {
        "balance_avg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}

# select avg(balance) from bank where age = 23
GET /bank/_search
{
  "query": {
    "term": {
      "age": {
        "value": 23
      }
    }
  },
  "aggs": {
    "balance_avg": {
      "avg": {
        "field": "balance"
      }
    }
  },
  "size": 0
}

# ------------------------
### full text

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
### analyzer の利用
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

# simple analyzer
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
DELETE /my_index/

# mapping(RDBでいうスキーマのようなもの)を定義
# question is parent of answer
PUT /my_index?pretty
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

GET /my_index/
GET /my_index/_search

# question documet(親ドキュメント)
# refresh により変更内容が検索できるタイミングを制御する
PUT /my_index/_doc/1?refresh&pretty
{
  "text": "This is a question",
  "my_join_field": {
    "name": "question"
  }
}

# 親ドキュメントはリレーション名を記載するだけでもいい
PUT /my_index/_doc/2?refresh&pretty
{
  "text": "This is another question",
  "my_join_field": "question"
}

# 子ドキュメント
# リレーション名と親ドキュメントのIDを指定する
# routing=1 は親子のドキュメントを同じシャードでインデクシングするため
PUT /my_index/_doc/3?routing=1&refresh&pretty
{
  "text": "This is an answer",
  "my_join_field": {
    "name": "answer",
    "parent": "1"
  }
}

PUT /my_index/_doc/4?routing=1&refresh&pretty
{
  "text": "This is another answer",
  "my_join_field": {
    "name": "answer",
    "parent": "1"
  }
}

GET /my_index/_search?pretty
{
  "query": {
    "match_all": {}
  },
  "sort": ["_id"]
}

# parent id で絞れる
GET /my_index/_search
{
  "query": {
    "parent_id":{
      "type": "answer",
      "id":1
    }
  }
}

# terms は group_by
# parent id で集約(aggregate)する
GET /my_index/_search
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
```