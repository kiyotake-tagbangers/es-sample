# store

- [ドキュメント](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html)

- 大きなフィールドを持つドキュメントから特定の特定のフィールドだけを取得したい

```shell
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

GET store-test

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
```

- _source は取得しないから、巨大なフィールドがあった場合、それを取得しなくて済む

```shell
GET /my_index/_search
{
  "stored_fields": [ "title", "date"]
}

"fields" : {
          "date" : [
            "2015-01-01T00:00:00.000Z"
          ],
          "title" : [
            "Some short title"
          ]
        }
```