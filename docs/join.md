- Kibana で実行

```shell
# -----------------------------------
### join
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