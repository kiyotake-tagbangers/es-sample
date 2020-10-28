```shell
# index の作成
curl -XPUT "http://elasticsearch:9200/products"

# index の削除
curl -XDELETE "http://elasticsearch:9200/products"

# document の作成
curl -XPOST "http://elasticsearch:9200/products/_doc" -H 'Content-Type: application/json' -d'{  "name": "Coffee Maker",  "price": 64,  "in_stock": 10}'

# ID を指定して作成
curl -XPUT "http://elasticsearch:9200/products/_doc/100" -H 'Content-Type: application/json' -d'{  "name": "Toaster",  "price": 30,  "in_stock": 4}'

# 取得
curl -XGET "http://elasticsearch:9200/products/_doc/100"

# 更新(documents は immutable, 新たなものに作り直されている)
curl -XPOST "http://elasticsearch:9200/products/_update/100" -H 'Content-Type: application/json' -d'{  "doc": {    "in_stock": 3  }}'

# フィールドの追加
curl -XPOST "http://elasticsearch:9200/products/_update/100" -H 'Content-Type: application/json' -d'{  "doc": {    "tags": ["electoronics"]  }}'

curl -XGET "http://elasticsearch:9200/products/_doc/100"

# フィールドの削除は取り除いて PUT する
curl -XPUT "http://elasticsearch:9200/products/_doc/100" -H 'Content-Type: application/json' -d'{    "name": "Toaster",    "price": 49,    "in_stock": 4}'

curl -XGET "http://elasticsearch:9200/products/_doc/100"

# ctx は context
curl -XPOST "http://elasticsearch:9200/products/_update/100" -H 'Content-Type: application/json' -d'{  "script": {    "source": "ctx._source.in_stock = 10"  }}'

# params の使用
curl -XPOST "http://elasticsearch:9200/products/_update/100" -H 'Content-Type: application/json' -d'{  "script": {    "source": "ctx._source.in_stock -= params.quantity",    "params": {      "quantity": 4    }  }}'

# if の使用
curl -XPOST "http://elasticsearch:9200/products/_update/100" -H 'Content-Type: application/json' -d'{  "script": {    "source": "      if (ctx._source.in_stock > 0) {\n        ctx._source.in_stock--;\n      }"  }}'

# upsert
# ドキュメントが存在していたらupdate, なければ新たに作成
curl -XPOST "http://elasticsearch:9200/products/_update/101" -H 'Content-Type: application/json' -d'{  "script": {    "source": "ctx._source.in_stock++"  },  "upsert": {    "name": "Blender",    "price": 399,    "in_stock": 5  }}'
curl -XGET "http://elasticsearch:9200/products/_doc/101"

# delete document
curl -XDELETE "http://elasticsearch:9200/products/_doc/101"

# optimistic concurrency control
curl -XGET "http://elasticsearch:9200/products/_doc/100"

# primary_term は変更を実行する primary shard
curl -XPOST "http://elasticsearch:9200/products/_update/100?if_primary_term=1&if_seq_no=24" -H 'Content-Type: application/json' -d'{  "doc":{    "in_stock": 13  }}'

# updata by query
curl -XPOST "http://elasticsearch:9200/products/_update_by_query" -H 'Content-Type: application/json' -d'{  "script": {    "source": "ctx._source.in_stock--"  },  "query": {    "match_all": {}  }}'

# 全ての ドキュメントの stock が -1 されている
curl -XGET "http://elasticsearch:9200/products/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match_all": {}  }}'

# delete by query
curl -XPOST "http://elasticsearch:9200/products/_delete_by_query" -H 'Content-Type: application/json' -d'{  "query": {    "match_all": {}  }}'

# bulk
curl -XPOST "http://elasticsearch:9200/_bulk" -H 'Content-Type: application/json' -d'{ "index": { "_index": "products", "_id": 200 } }{ "name": "Espresso Machine", "price": 199, "in_stock": 5 }{ "create": { "_index": "products", "_id": 201 } }{ "name": "Milk Frother", "price": 149, "in_stock": 14 }'

curl -XGET "http://elasticsearch:9200/products/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match_all": {}  }}'

# bulk request from file
ls products-bulk.json

head -n 2 products-bulk.json 
{"index":{"_id":1}}
{"name":"Wine - Maipo Valle Cabernet","price":152,"in_stock":38,"sold":47,"tags":["Alcohol","Wine"],"description":"Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat. Praesent blandit. Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula.","is_active":true,"created":"2004\/05\/13"}

curl -H "Content-Type: application/x-ndjson" -XPOST http://localhost:9200/products/_bulk --data-binary "@products-bulk.json"

curl -XGET "http://elasticsearch:9200/_cat/shards?v"
```