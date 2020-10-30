- nested object

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

GET /my_index/_doc/1

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
