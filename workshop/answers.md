# edge n-grams
1. prepare index
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
                        "match": { "name.autocomplete": "c" }
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
        * vs filter - we have scoring which is helpful
        ```
        GET /grocery/_search
        {
            "query": {
                "bool": {
                    "must": {
                        "match": { "name.autocomplete": "cho ba" }
                    }
                }
            }
        }      
        ```
        
# shingles
1. prepare index
    ```
    PUT /cookbook
    {
        "mappings": {
            "properties": {
                "name": { "type": "text" },
                "recipe": { "type": "search_as_you_type"}
            }
        }
    }
    ```
1. search for phrase 'ostrich eg'
    * without `recipe` - only phrase matching
    ```
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
    ```
# stemming
1. prepare index
    ```
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
    ```
1. verify stemming analyzer
    ```
    POST twitter/_analyze
    {
        "analyzer": "stem_analyzer",
        "text": "in linguistic morphology and information retrieval, stemming is the process of reducing 
        inflected words to their word stem, base or root form—generally a written word form"
    }
    ```
1. check stemmed terms
    ```
    GET /twitter/_termvectors/1?fields=tweet
    ```
1. search for 'celebrate elected'
    ```
    GET /twitter/_search
    {
        "query": {
            "match": {
                "tweet": "celebrate elected"
            }
        }
    }
    ```
# fuzzy
1. get Elvis and Elisabeth as a search result when searching for `El`
    ```
    GET employees/_search
    {
        "query": { "prefix": { "name.keyword": "El" } }
    }
    ```
1. make typo in `El` -> `Eli` - result: only Elisabeth
    ```
    GET employees/_search
    {
        "query": { "prefix": { "name.keyword": "Eli" } }
    }
    ```
1. handle typos with fuzz query - result: only Elvis
    ```
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
    ```
1. combine queries to get `Elvis` and `Elisabeth` as a search result when searching for `Eli`
    ```
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
    ```
    
# suggester
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