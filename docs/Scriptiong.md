- script

```shell
# インデックスの作成
$ curl --location --request PUT 'http://localhost:9200/my_index/'

# ドキュメントをインデキシング
$ curl --location --request PUT 'http://localhost:9200/my_index/_doc/1' \
--header 'Content-Type: application/json' \
--data-raw '{
  "": 5
}'

# 取得
$ curl --location --request GET 'http://localhost:9200/my_index/_search?pretty'
```

```json
{
  "took" : 1,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "my_field" : 5
        }
      }
    ]
  }
}
```

```shell
# GET 時に script を実行
# my_doubled_field を定義し my_field を2倍にした値を取得
$ curl --location --request GET 'http://localhost:9200/my_index/_search?pretty' \
--header 'Content-Type: application/json' \
--data-raw '{
  "script_fields": {
    "my_doubled_field": {
      "script": {
        "lang":   "expression",
        "source": "doc['\''my_field'\''] * multiplier",
        "params": {
          "multiplier": 2
        }
      }
    }
  }
}'
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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "my_doubled_field" : [
            10.0
          ]
        }
      }
    ]
  }
}
```

- document のフィールド値にアクセス

```shell
curl -X PUT "localhost:9200/my_index/_doc/1?refresh&pretty" -H 'Content-Type: application/json' -d'
{
  "cost_price": 100
}
'

$ curl -X GET "localhost:9200/my_index/_doc/1?refresh&pretty"
```

```json
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 5,
  "_seq_no" : 7,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "cost_price" : 100
  }
}
```

```shell
$ curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "script_fields": {
    "sales_price": {
      "script": {
        "lang":   "expression",
        "source": "doc[\u0027cost_price\u0027] * markup",
        "params": {
          "markup": 0.2
        }
      }
    }
  }
}
'
```

```json
{
  "took" : 5,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "sales_price" : [
            20.0
          ]
        }
      }
    ]
  }
}
```

- store されているかでアクセスの方法が変わる


```shell
# インデックスの登録
# store していないものは、params._fields['aaa'].value ではアクセスできない(parames._source.title でアクセス)
curl -X PUT "localhost:9200/my_index?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "first_name": {
        "type": "text",
        "store": true
      },
      "last_name": {
        "type": "text",
        "store": true
      }
    }
  }
}
'

curl -X PUT "localhost:9200/my_index/_doc/1?refresh&pretty" -H 'Content-Type: application/json' -d'
{
  "title": "Mr",
  "first_name": "Barry",
  "last_name": "White"
}
'

# https://www.elastic.co/guide/en/elasticsearch/reference/7.4/modules-scripting-painless.html
# Painless is a simple, secure scripting language designed specifically for use with Elasticsearch.
# Fast performance: Painless scripts run several times faster than the alternatives.
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "script_fields": {
    "source": {
      "script": {
        "lang": "painless",
        "source": "params._source.title + \u0027 \u0027 + params._source.first_name + \u0027 \u0027 + params._source.last_name"
      }
    },
    "stored_fields": {
      "script": {
        "lang": "painless",
        "source": "params._fields[\u0027first_name\u0027].value + \u0027 \u0027 + params._fields[\u0027last_name\u0027].value"
      }
    }
  }
}
'
```

```json
{
  "took" : 8,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "stored_fields" : [
            "Barry White"
          ],
          "source" : [
            "Mr Barry White"
          ]
        }
      }
    ]
  }
}
```

- ScriptEngine
  - 今回は、 Spring Data Elasticsearch を使用するのでこちらは意識しなくても良さそう

> A ScriptEngine is a backend for implementing a scripting language.

```java
private static class MyExpertScriptEngine implements ScriptEngine {
  (略)
}
```
