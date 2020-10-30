- RDB でいうスキーマ定義のようなもの

```shell
# mapping
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

DELETE /reviews

# properties の nest は . でかける
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
      "author.first_name":{
            "type": "text"
          },
      "author.last_name":{
            "type": "text"
          }
    }
  }
}

GET /reviews/_mapping
GET /reviews/_mapping/field/content
GET /reviews/_mapping/field/author.first_name

PUT /reviews/_doc/1
{
  "rating": 5.0,
  "content": "Awesome",
  "product_id": 123,
  "author":{
    "first_name": "Michkel",
    "last_name": "Jackson"
  }
}

PUT /reviews/_doc/2
{
  "rating": 3.0,
  "content": "Soso",
  "product_id": 124,
  "author.first_name": "Michkel"
  "author.last_name": "Jackson"
  }
}


GET /reviews/_search
```