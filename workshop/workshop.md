# edge n-grams
1. we would like to index
    * we would like to have autocomplete on name
    ```
    {
        "name": "Chocolate Bar",
        "price": 10.0,
        "quantity": 5
    }
    ```
1. prepare index
    * custom analyzer with custom edge_ngram filter
        * hint: `settings.analysis.filter`, type: `edge_ngram`
            * min: 2, hint: `min_gram`
            * max: 5, hint: `max_gram`
        * hint: `settings.analysis.analyzer`, type: `custom`, `"filter": [ ..., "custom filter"]`
    * autocomplete for `name`
        * hint: multi-field mapping for `name`
        * hint: `"analyzer": "autocomplete_analyzer"`, `"search_analyzer": "standard"`
    ```
    PUT /grocery
    {
        "settings": {
            "analysis": {
                "filter": {
                    ...
                },
                "analyzer": {
                    ...
                }
            }
        },
        "mappings": {
            "properties": {
                "name": { 
                    "type": "text",
                    "fields" : {
                        ...
                    }
                },
                ...
            }
        }
    }
    ```
1. verify analyzer
    * hint: `POST grocery/_analyze`, `"analyzer": "autocomplete_analyzer"`
1. populate index
    * run `grocery data` from `data.md`
1. verify indexed terms
    * hint: `_termvectors`, `fields=name`
1. search with autocomplete
    * hint: `query.bool`
    * hint: `name.autocomplete`
    * starts with `c`
        * note that min ngram was set as 2
    * starts witch `choc`
    * starts with `app`
    * search for `cho ba`
        * compare using `must` and `filter` (no scoring)
# shingles
1. we would like to index
    * search as you type on recipe
    ```
    {
        "name": "scrambled eggs",
        "recipe": "one large ostrich egg or 25 smaller chicken eggs"
    }
    ```
1. prepare index
    * hint: `"type": "search_as_you_type"`
    ```
    PUT /cookbook
    {
        "mappings": {
            "properties": {
                ...
            }
        }
    }
    ```
1. populate index
    * run `cookbook data` from `data.md`
1. search for phrase `ostrich eg`
    * hint: `multi_match`, type: `bool_prefix`, fields: `recipe`, `recipe._2gram`, `recipe._2gram`
    * what will happen when you remove recipe from search fields?