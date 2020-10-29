```shell
ls recipes-bulk.json

curl -H "Content-Type: application/x-ndjson" -XPOST "http://localhost:9200/recipe/_bulk?pretty" --data-binary "@recipes-bulk.json"

curl -XGET "http://elasticsearch:9200/recipe/_mapping"

# Flexible matching with the match query
# いずれかの単語を含んでいればヒット
curl -XGET "http://elasticsearch:9200/recipe/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match": {      "title": "Recipes with pasta or spaghetti"    }  }}'

# 全て含んでいればヒット(0件)
curl -XGET "http://elasticsearch:9200/recipe/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match": {      "title": {        "query": "Recipes with pasta or spaghetti",        "operator": "and"      }    }  }}'

# pasta,or,spaghetti を含んでいればヒット
curl -XGET "http://elasticsearch:9200/recipe/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match": {      "title": {        "query": "pasta or spaghetti",        "operator": "and"      }    }  }}'

# matching phrases
# "title" : "Spaghetti Puttanesca (Pasta or Spaghetti With Capers, Olives, and Anchovies)" がヒット
curl -XGET "http://elasticsearch:9200/recipe/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match_phrase": {      "title": "spaghetti puttanesca"    }  }}'

# 順番を変えると今回のデータだとヒットしない
curl -XGET "http://elasticsearch:9200/recipe/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match_phrase": {      "title": "puttanesca spaghetti"    }  }}'

# searching multiple fields
curl -XGET "http://elasticsearch:9200/recipe/_search" -H 'Content-Type: application/json' -d'{  "query": {    "multi_match": {      "query": "pasta",      "fields": ["title", "description"]    }  }}'
```