```shell
# standard analyzer の使用
POST /_analyze
{
  "text": "2 guys walk into    a bar, but the third ...DUCKS! :-)",
  "analyzer": "standard "
}

# character filter,tokenizer, token filter を明示的に書いた場合
POST /_analyze
{
  "text": "2 guys walk into    a bar, but the third ...DUCKS! :-)",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}

# keyword datatype
# text field が single term として保存される
# email, status などで使われる
POST /_analyze
{
  "text": ["2 guys walk into a bar, but the third ... DUSKS"],
  "analyzer": "keyword"
}

```