```shell
# standard analyzer の使用
POST /_analyze
{
  "text": "2 guys walk into    a bar, but the third ...DUCKS! :-)",
  "analyzer": "standard"
}

# character filter,tokenizer, token filter を明示的に書いた場合
POST /_analyze
{
  "text": "2 guys walk into    a bar, but the third ...DUCKS! :-)",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}

# simple analyzer
POST /_analyze
{
  "text": "Is that Mike's cute-looing dog?",
  "analyzer": "simple"
}

# keyword analyzer
# text field が single term として保存される
# email, status などで使われる
POST /_analyze
{
  "text": ["2 guys walk into a bar, but the third ... DUSKS"],
  "analyzer": "keyword"
}
```

- custom analyzer

```shell
# html のタグを取り除く
POST /_analyze
{
  "char_filter": ["html_strip"],
  "text": "I'm so <em>sleepy</em> and I <strong>hungry</strong>!"
}

PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop"
          ]
        }
      }
    }
  }
}

# stop word である and が消えている
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer", 
  "text": "I'm so <em>sleepy</em> and I <strong>hungry</strong>!"
}

```