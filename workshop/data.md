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