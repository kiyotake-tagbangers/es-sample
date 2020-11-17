- nested object

- オブジェクトの配列にインデックスをつけ、独立性を維持する必要がある場合に利用する
  - Lucene が内部オブジェクトの概念を持っておらず、フラットなリストとしてデータを所持するため

```shell
# nested object
PUT /my_index/_doc/1?pretty
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
```

```shell
# フラットにマピングされるため、組み合わせ違いでもヒットしてしまう
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": { "user.first": "Alice"}},
        {"match": { "user.last": "Smith"}}
      ]
    }
  }
}
```

```shell
# Alice の集計に、 smith も white も含まれてしまう
GET my_index/_search
{
  "aggs": {
    "group_by_first_name": {
      "terms": {
        "field": "users.first_name.keyword"
      },
      "aggs": {
        "group_by_last_name": {
          "terms": {
             "field": "users.last_name.keyword"
          }
        }
      }
    }
  }
}
```

```shell
        "key" : "Alice",
          "doc_count" : 1,
          "group_by_last_name" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Smith",
                "doc_count" : 1
              },
              {
                "key" : "White",
                "doc_count" : 1
              }
            ]
          }
        },
```

- Nested Object を使う

```shell
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
```

```shell
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
```

```shell
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
```

```shell
# 一致するネストを強調
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
```
