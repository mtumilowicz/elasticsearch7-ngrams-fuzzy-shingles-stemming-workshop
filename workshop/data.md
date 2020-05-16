# edge n-grams
1. grocery data
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
# shingles
1. cookbook data
    ```
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
    ```
# stemmers
1. twitter data
    ```
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
    ```
# fuzzy
1. employees data
    ```
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
    ```
# completion
1. movies data
    ```
    POST movies/_doc
    {
        "name": "Iron Man",
        "search_associations": ["avenger", "Robert Downey Jr.", "John Favreau" ],
        "year": 2008
    }
    POST movies/_doc
    {
        "name": "The Incredible Hulk",
        "search_associations": ["avenger", "hulk", "marvel, "norton" ],
        "year": 2008
    }
    ```