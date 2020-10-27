- [クラスタのステータスの確認](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/getting-started-install.html#run-elasticsearch-local)

```shell
$ curl -X GET "localhost:9200/_cat/health?v&pretty"

epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1603769639 03:33:59  docker-cluster green           1         1      0   0    0    0        0             0                  -                100.0%
```

- [ドキュメントに index をつける](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/getting-started-index.html)
    - customer インデックスが存在しない場合は、新しいドキュメントを追加して name フィールドにインデックスを作成

```shell
# /{index}/{type}/{id}
# customer インデックスに customer ドキュメントをインデキシング
# 1つのドキュメントは一意なIDで管理されている
$ curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```

- [取得](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/getting-started-index.html#getting-started-index)

```shell
$ curl -X GET "localhost:9200/customer/_doc/1?pretty"
```

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "John Doe"
  }
}
```

- 存在しないドキュメントを指定した場合

```shell
$ curl -X GET "localhost:9200/customer/_doc/0?pretty"
```

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "0",
  "found" : false
}
```

- [バルクAPIを使用してバッチで送信](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/getting-started-index.html#getting-started-batch-processing)
    - インデックスを作成するドキュメントがたくさんある場合に使用する
    - ネットワークのラウンドトリップを最小限に抑えるため、リクエストを個別に送信するよりも大幅に高速

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

- [検索](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/getting-started-search.html#getting-started-search)

- sort

```shell
# account_number で sort したものを2件検索
# size を指定しなかった場合、10件のドキュメントが返される
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "from" : 0, "size" : 2,
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
```

```json
{
  "took" : 3, // クエリの検索にかかった時間(ミリ秒)
  "timed_out" : false, // 検索リクエストがタイムアウトしたか
  "_shards" : {
    "total" : 1, ・・検索されたシャードの数
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000, // ドキュメントがトータルでいくつ見つかったか
      "relation" : "eq"
    },
    "max_score" : null, // 最も関連性が高いドキュメントのスコア
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "0",
        "_score" : null,　// ドキュメントの関連性スコア
        "_source" : {
          "account_number" : 0, // この値でソートしている
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
        "sort" : [
          0
        ]
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        },
        "sort" : [
          1
        ]
      }
    ]
  }
}
```

```shell
# ヒットしたものの地点を指定して表示できる(from)
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 2
}
'
```

- match(単語で検索)

```shell
# address(index) が mill か lane を含むものがヒットする
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'
```

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 19,
      "relation" : "eq"
    },
    "max_score" : 9.507477, // 最も関連性が高いドキュメントのスコア
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "136",
        "_score" : 9.507477, // ドキュメントの関連性スコア
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      },
(略)
    {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 4.1042743,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        }
      },
}
```

- match_phrase(フレーズで検索)

```shell
# mill lane を含むものがヒットする
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'
```

```json
{
  "took" : 11,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 9.507477, // 最も関連性が高いドキュメントのスコア
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "136",
        "_score" : 9.507477,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      }
    ]
  }
}
```

- 複雑なクエリ

```shell
# 40歳の顧客に属するアカウントをインデックスで検索(must, must_not ブールクエリ内の要素はクエリ句(query clause)と呼ばれている)
# ただし、アイダホ(ID)に住んでいる人は除外
# 2件だけ表示
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 0,
  "size": 2
}
'
```

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 43,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "40",
        "_score" : null,
        "_source" : {
          "account_number" : 40,
          "balance" : 33882,
          "firstname" : "Pace",
          "lastname" : "Molina",
          "age" : 40,
          "gender" : "M",
          "address" : "263 Ovington Court",
          "employer" : "Cytrak",
          "email" : "pacemolina@cytrak.com",
          "city" : "Silkworth",
          "state" : "OR"
        },
        "sort" : [
          40
        ]
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "58",
        "_score" : null,
        "_source" : {
          "account_number" : 58,
          "balance" : 31697,
          "firstname" : "Marva",
          "lastname" : "Cannon",
          "age" : 40,
          "gender" : "M",
          "address" : "993 Highland Place",
          "employer" : "Comcubine",
          "email" : "marvacannon@comcubine.com",
          "city" : "Orviston",
          "state" : "MO"
        },
        "sort" : [
          58
        ]
      }
    ]
  }
}
```

- 範囲検索

```shell
# 残高が 20,000 ~ 20,100 のアカウントに結果を制限
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 20100
          }
        }
      }
    }
  }
}
'
```

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "481",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 481,
          "balance" : 20024,
          "firstname" : "Lina",
          "lastname" : "Stanley",
          "age" : 33,
          "gender" : "M",
          "address" : "361 Hanover Place",
          "employer" : "Strozen",
          "email" : "linastanley@strozen.com",
          "city" : "Wyoming",
          "state" : "NC"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "713",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 713,
          "balance" : 20054,
          "firstname" : "Iris",
          "lastname" : "Mcguire",
          "age" : 21,
          "gender" : "F",
          "address" : "508 Benson Avenue",
          "employer" : "Duflex",
          "email" : "irismcguire@duflex.com",
          "city" : "Hillsboro",
          "state" : "MO"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "985",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 985,
          "balance" : 20083,
          "firstname" : "Martin",
          "lastname" : "Gardner",
          "age" : 28,
          "gender" : "F",
          "address" : "644 Fairview Place",
          "employer" : "Golistic",
          "email" : "martingardner@golistic.com",
          "city" : "Connerton",
          "state" : "NJ"
        }
      }
    ]
  }
}
```

- [集約](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/getting-started-aggregations.html#getting-started-aggregations)

```shell
# terms集計を使用して bank インデックス内の全てのアカウントを state ごとにグルーピング、10件表示
# 集約名 group_by_state は任意の名前
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'
```

```json
{
  "took" : 48,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30
        },
        {
          "key" : "MD",
          "doc_count" : 28
        },
        {
          "key" : "ID",
          "doc_count" : 27
        },
        {
          "key" : "AL",
          "doc_count" : 25
        },
        {
          "key" : "ME",
          "doc_count" : 25
        },
        {
          "key" : "TN",
          "doc_count" : 25
        },
        {
          "key" : "WY",
          "doc_count" : 25
        },
        {
          "key" : "DC",
          "doc_count" : 24
        },
        {
          "key" : "MA",
          "doc_count" : 24
        },
        {
          "key" : "ND",
          "doc_count" : 24
        }
      ]
    }
  }
}
```

- 集約の組み合わせ

```shell
# 前の group_by_state 集計内に、集計をネスト
# 各州の平均口座残高を計算
# 集計名 average_balance は任意の名前
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
```

```json
{
  "took" : 19,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30,
          "average_balance" : {
            "value" : 26073.3
          }
        },
        {
          "key" : "MD",
          "doc_count" : 28,
          "average_balance" : {
            "value" : 26161.535714285714
          }
        },
        {
          "key" : "ID",
          "doc_count" : 27,
          "average_balance" : {
            "value" : 24368.777777777777
          }
        },
        {
          "key" : "AL",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 25739.56
          }
        },
        {
          "key" : "ME",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 21663.0
          }
        },
        {
          "key" : "TN",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 28365.4
          }
        },
        {
          "key" : "WY",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 21731.52
          }
        },
        {
          "key" : "DC",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 23180.583333333332
          }
        },
        {
          "key" : "MA",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 29600.333333333332
          }
        },
        {
          "key" : "ND",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 26577.333333333332
          }
        }
      ]
    }
  }
}
```

- 集約の組み合わせ2

```shell
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}
'
```
