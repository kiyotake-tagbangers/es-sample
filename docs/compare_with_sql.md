- [Kibana を立ち上げておく](../README.md)

- [バルクAPIを使用してバッチでデータを登録しておく](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/getting-started-index.html#getting-started-batch-processing)

```shell
# sample のバルクデータをファイルとして保存
$ wget https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json -O data/accounts.json

$ head -n 2 accounts.json
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}

# バルクリクエスト
$ curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary @accounts.json

# インデックスの件数の確認(バルクリクエストで1000件登録されている)
$ curl "localhost:9200/_cat/indices?v"

health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank     ChO6ztHQSbOpCnjw9KDXKA   1   1       1000            0    414.3kb        414.3kb
yellow open   customer 8L0_xCTDSQC5wTnzSAVUlQ   1   1          1            0      3.5kb          3.5kb
```

- Kibana 上で以下の query を実行する

```shell
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

```