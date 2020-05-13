## autocomplete
1. settings
    PUT /grocery
    {
        "settings": {
            "analysis": {
                "filter": {
                    "autocomplete_filter": {
                        "type": "edge_ngram",
                        "min_gram": 1,
                        "max_gram": 5
                    }
                },
                "analyzer": {
                    "autocomplete_analyzer": {
                        "type": "custom",
                        "tokenizer": "standard",
                        "filter": [ "lowercase", "autocomplete_filter"]
                    }
                }
            }
        },
        "mappings": {
            "properties": {
                "name": { "type": "text", "analyzer": "autocomplete_analyzer", "search_analyzer": "standard" },
                "price": { "type": "double"},
                "quantity": { "type": "integer"}
            }
        }
    }
1. verify analyzer
POST grocery/_analyze
{
  "analyzer": "autocomplete_analyzer",
  "text": "Chocolate Bar"
}
1. populate index
POST /grocery/_create/1
{
    "name": "Chocolate Bar",
    "price": 10.0,
    "quantity": 5
}
POST /grocery/_create/2
{
    "name": "Chocolate cake",
    "price": 25.0,
    "quantity": 2
}
POST /grocery/_create/3
{
    "name": "Chocapic",
    "price": 10.0,
    "quantity": 5
}
POST /grocery/_create/4
{
    "name": "Coconut",
    "price": 5.0,
    "quantity": 25
}
POST /grocery/_create/5
{
    "name": "Apple",
    "price": 2.0,
    "quantity": 13
}
1. verify index
    GET /grocery/_termvectors/1?fields=name
1. search with autocomplete
    * starts with c
        GET /grocery/_search
        {
          "query": {
            "bool": {
              "filter": {
                "match": { "name": "c" }
              }
            }
          }
        }
    * starts witch choc
        GET /grocery/_search
        {
          "query": {
            "bool": {
              "filter": {
                "match": { "name": "choc" }
              }
            }
          }
        }
    * starts with app
        GET /grocery/_search
        {
          "query": {
            "bool": {
              "filter": {
                "match": { "name": "app" }
              }
            }
          }
        }
    * starts with 