- 発表で使うものをひとまとめにしたもの
    - 詳細はそれぞれのファイルで検証

```shell
# --------------------------------
### 基本操作
# インデックスの作成
PUT /products

# インデックスの削除
DELETE /products

# mapping の定義(RDS でいうスキーマのイメージ)
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": {
        "type": "float"
      },
      "content":{
        "type": "text"
      },
      "product_id":{
        "type": "integer"
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

# ドキュメントを追加
# インデックスはなければ自動的に追加される
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}

# ID を指定して作成
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 30,
  "in_stock": 4
}

GET /products/_search

# ID を指定して取得
GET /products/_doc/100

# 更新(documents は immutable, 新たなものに作り直されている)
POST /products/_update/100
{
  "doc": {
    "in_stock": 3
  }
}

# ドキュメントの削除
DELETE /products/_doc/100

# インデックスの削除
DELETE /products

# 条件にマッチしてれば scirpt を実行
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock > 0) {
        ctx._source.in_stock--;
      }
    """
  }
}

# --------------------------------
### SQL との比較
# select * from bank
GET /bank/_search
{
  "query": {
    "match_all": {}
  }
}

# select * bank limit 3 offset 5
GET /bank/_search
{
  "from": 5,
  "size": 3,
  "query": {
    "match_all": {}
  }
}

# select account_number, email from bank
GET /bank/_search
{
  "_source": [
    "account_number",
    "email"
  ],
  "query": {
    "match_all": {}
  }
}

# 重複したデータを入れる
DELETE /duplicate

POST /duplicate/_doc
{
  "name": "Duplicated text",
  "duplicate_id": 3
}

GET /duplicate/_search

# select distinct * from duplicate
GET /duplicate/_search
{
  "collapse": {
    "field": "duplicate_id"
  },
  "query": {
    "match_all": {}
  }
}

# "unknown type for collapse field `name`, only keywords and numbers are accepted"
GET /duplicate/_search
{
  "collapse": {
    "field": "name"
  },
  "query": {
    "match_all": {}
  }
}

GET /duplicate/_search
{
  "collapse": {
    "field": "name.keyword"
  },
  "query": {
    "match_all": {}
  }
}


GET /bank/_search

# select * from bank where account_number = 1
# term は完全一致か確認
GET /bank/_search
{
  "query": {
    "term": {
      "account_number": {
        "value": 1
      }
    }
  }
}

# select * from bank where email = "amberduke@pyrami.com"
GET /bank/_search
{
  "query": {
    "term": {
      "email.keyword": {
        "value": "daleadams@boink.com"
      }
    }
  }
}

# terms は完全一致で複数の値を指定できる
# select * form bank where employer in ("Pyrami","Anocha","Reversus")

GET /bank/_search
{
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

GET /bank/_search

# select * from bank where email like '%son%' limit 20;
GET /bank/_search
{
  "query": {
    "wildcard": {
      "email.keyword": {
        "value": "*son*"
      }
    }
  },
  "size": 20
}

# select * from bank where balance >= 49000
GET /bank/_search
{
  "query": {
    "range": {
      "balance": {
        "gte": 49000
      }
    }
  }
}
# 複合クエリ(compound queries)
# select * from product where age = 23 and balance >= 40000
GET /bank/_search
{
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
            "range": {
              "balance": {
                "gte": 40000
              }
            }
          }
      ]
    }
  }
}

# select * from product where age = 23 or balance >= 49000
GET /bank/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
          "age": {
            "value": 23
          }
        }
        },
          {
            "range": {
              "balance": {
                "gte": 49000
              }
            }
          }
      ]
    }
  }
}

# AND,OR を組み合わせるため、 bool query をネスト
# select * from product where age = 23 and (balance >= 40000 or city = "TX")
GET /bank/_search
{
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
  }
}

GET /bank/_search

# select * from bank order by balance DESC limit 1
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "balance": {
        "order": "desc"
      }
    }
  ],
  "size": 1
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