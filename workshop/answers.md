## autocomplete
1. settings
    ```
    PUT /grocery
    {
        "settings": {
            "analysis": {
                "filter": {
                    "autocomplete_filter": {
                        "type": "edge_ngram",
                        "min_gram": 2,
                        "max_gram": 5
                    }
                },
                "analyzer": {
                    "autocomplete_analyzer": {
                        "type": "custom",
                        "tokenizer": "standard",
                        "filter": [ "lowercase", "stop", "autocomplete_filter"]
                    }
                }
            }
        },
        "mappings": {
            "properties": {
                "name": { 
                    "type": "text",
                    "fields" : {
                        "autocomplete": {
                            "type": "text", 
                            "analyzer": "autocomplete_analyzer", 
                            "search_analyzer": "standard" 
                        }
                    }
                },
                "price": { "type": "double" },
                "quantity": { "type": "integer" }
            }
        }
    }
    ```
1. verify analyzer
    ```
    POST grocery/_analyze
    {
        "analyzer": "autocomplete_analyzer",
        "text": "Chocolate Bar"
    }
    ```
1. populate index
    ```
    POST /grocery/_create/1
    {
        "name": "Chocolate Bar",
        "price": 10.0,
        "quantity": 5
    }
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
   ```
1. verify indexed terms
    ```
    GET /grocery/_termvectors/1?fields=name
    ```
1. search with autocomplete
    * starts with `c`
        ```
        GET /grocery/_search
        {
            "query": {
                "bool": {
                    "filter": {
                        "match": { "name.autocomplete": "c" } // note that min is 2
                    }
                }
            }
        }
        ```
    * starts witch `choc`
        ```
        GET /grocery/_search
        {
            "query": {
                "bool": {
                    "must": {
                        "match": { "name.autocomplete": "choc" }
                    }
                }
            }
        }      
        ```
    * starts with `app`
        ```
        GET /grocery/_search
        {
            "query": {
                "bool": {
                    "filter": {
                        "match": { "name.autocomplete": "app" }
                    }
                }
            }
        }      
        ```
    * search for `cho ba`
        ```
        GET /grocery/_search
        {
            "query": {
                "bool": {
                    "must": { // compare with filter (no scoring)
                        "match": { "name.autocomplete": "cho ba" }
                    }
                }
            }
        }      
        ```
        
## shingles
1. prepare index
    PUT /cookbook
    {
        "mappings": {
            "properties": {
                "name": { "type": "text" },
                "recipe": { "type": "search_as_you_type"}
            }
        }
    }
1. populate index
POST /cookbook/_create/1
{
    "name": "scrambled eggs",
    "recipe": "one large ostrich egg or 25 smaller chicken eggs"
}
POST /cookbook/_create/2
{
    "name": "hot wings",
    "recipe": "ostrich meat and chicken wings"
}
POST /cookbook/_create/3
{
    "name": "strange dish",
    "recipe": "ostrich beak, chicken liver, snake eggs"
}
1. search for phrase 'ostrich egg'
    GET /cookbook/_search
    {
      "query": {
        "multi_match": {
          "query": "ostrich egg",
          "type": "bool_prefix",
          "fields": [
            "recipe", // switch it off
            "recipe._2gram",
            "recipe._3gram"
          ]
        }
      }
    }
1. search for phrase 'ostrich e'
    GET /cookbook/_search
    {
      "query": {
        "multi_match": {
          "query": "ostrich egg",
          "type": "bool_prefix",
          "fields": [
            "recipe._2gram",
            "recipe._3gram"
          ]
        }
      }
    }

## stemming
1. prepare index
    PUT /twitter
    {
        "settings": {
            "analysis": {
                "analyzer": {
                    "stem_analyzer": {
                        "type": "custom",
                        "tokenizer": "standard",
                        "filter": [ "lowercase", "stop", "porter_stem"]
                    }
                }
            }
        },
        "mappings": {
            "properties": {
                "author": { "type": "keyword" },
                "tweet": { "type": "text", "analyzer": "stem_analyzer" }
            }
        }
    }
1. verify stemming analyzer
POST twitter/_analyze
{
  "analyzer": "stem_analyzer",
  "text": "in linguistic morphology and information retrieval, stemming is the process of reducing inflected words to their word stem, base or root form—generally a written word form"
}
1. populate index
POST /twitter/_create/1
{
    "author": "realDonaldTrump",
    "tweet": "Russian leaders are publicly celebrating Obama’s reelection. They can't wait to see how flexible Obama will be now"
}
POST /twitter/_create/2
{
    "author": "realDonaldTrump",
    "tweet": "Sorry losers and haters, but my I.Q. is one of the highest - and you all know it! Please don't feel so stupid or insecure, it's not your fault"
}
1. check stemming
    GET /twitter/_termvectors/1?fields=tweet
1. search for 'celebrate elected'
    GET /twitter/_search
    {
      "query": {
        "match": {
          "tweet": "celebrate elected"
        }
      }
    }
## fuzzy
1. populate index
    POST _bulk
    {"index":{"_index":"employees","_id":"3"}}
    {"name":"Elvis"}
    {"index":{"_index":"employees","_id":"7"}}
    {"name":"Average Joe"}
    {"index":{"_index":"employees","_id":"11"}}
    {"name":"Elisabeth"}
    {"index":{"_index":"employees","_id":"13"}}
    {"name":"William"}
    {"index":{"_index":"employees","_id":"17"}}
    {"name":"Jack"}
    {"index":{"_index":"employees","_id":"19"}}
    {"name":"Iris"}
    {"index":{"_index":"employees","_id":"23"}}
    {"name":"Barbara"}
    {"index":{"_index":"employees","_id":"29"}}
    {"name":"Averell Dalton"}
    {"index":{"_index":"employees","_id":"31"}}
    {"name":"Bryce Dallas Howard"}
    {"index":{"_index":"employees","_id":"37"}}
    {"name":"Fabio"}
    {"index":{"_index":"employees","_id":"41"}}
    {"name":"Fabian"}
1. find Elisabeth and Elvis using prefix
    GET employees/_search
    {
      "query": { "prefix": { "name.keyword": "El" } }
    }
1. make typo in above query and type Eli
    GET employees/_search
    {
      "query": { "prefix": { "name.keyword": "Eli" } }
    }
    * only Elisabeth
1. handle typos with fuzz query
    GET /_search
    {
      "query": {
        "fuzzy": {
          "name.keyword": {
            "value": "Eli",
            "fuzziness": 2
          }
        }
      }
    }
    * only elvis
1. combine with prefix
    GET employees/_search
    {
      "query": {
        "bool": {
          "should": [
            { "prefix": { "name.keyword": "Eli" } },
            { "fuzzy": { "name.keyword": { "value": "Eli", "fuzziness": 2 } } }
          ]
        }
      }
    }
    * Elvis, Elisabeth

## suggester
1. index
PUT movies
{
  "mappings": {
      "properties": {
        "name": {
          "type": "completion"
        },
        "year": {
          "type": "keyword"
        }
      }
    }
}
1. populate
POST movies/_doc
{
  "name": {
    "input": ["Iron Man"]
  },
  "year": 2008
}
POST movies/_doc
{
  "name": {
    "input": ["Thor"]
  },
  "year": 2011
}
POST movies/_doc
{
  "name": {
    "input": ["Iron Man 2"]
  },
  "year": 2010
}
POST movies/_doc
{
  "name": [
    {
      "input": [
        "The Incredible Hulk"
      ]
    },
    {
      "input": [
        "Hulk"
      ]
    }
  ],
  "year": 2008
}
1. search for thor
POST movies/_search
{
  "suggest": {
    "movie-suggest": {
      "prefix": "thor",
      "completion": {
        "field": "name"
      }
    }
  }
}