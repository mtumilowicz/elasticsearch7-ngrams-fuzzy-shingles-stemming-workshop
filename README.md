[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

# elasticsearch7-ngrams-fuzzy-shingles-stemming-workshop

* references
    * ngram
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenizer.html
        * https://qbox.io/blog/an-introduction-to-ngrams-in-elasticsearch
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html
        * https://medium.com/@ashishstiwari/what-should-be-the-value-of-max-gram-and-min-gram-in-elasticsearch-f091404c9a14
        * https://kb.objectrocket.com/elasticsearch/how-to-implement-autocomplete-with-edge-n-grams-in-elasticsearch
    * shingles
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-shingle-tokenfilter.html
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/index-phrases.html
        * https://www.elastic.co/blog/searching-with-shingles
    * stemming
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html#stemming
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-porterstem-tokenfilter.html
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-snowball-tokenfilter.html
    * fuzzy
        * https://en.wikipedia.org/wiki/Levenshtein_distance
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html
        * https://www.elastic.co/blog/found-fuzzy-search
        * https://kb.objectrocket.com/elasticsearch/how-to-use-fuzzy-query-matches-in-elasticsearch
        * https://dev.to/hernamvel/understanding-and-tuning-fuzzy-queries-in-elasticsearch-by-example-1ci3
        * https://medium.com/@neelambuj2/an-approach-to-highly-intuitive-fuzzy-search-in-elasticsearch-with-typo-handling-exact-matches-a79a795d36f8
        * https://blog.mimacom.com/autocomplete-elasticsearch-part1/
    * suggester
        * https://medium.com/@taranjeet/elasticsearch-using-completion-suggester-to-build-autocomplete-e9c120cf6d87
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester

## preface
* goals of this workshop
    * introduction to typos and suggestions handling in elasticsearch
    * introduction to basic constructs boosting search
        * ngram and edge ngram (typos, prefixes)
        * shingles (phrase)
        * stemmers (operating on roots rather than words)
        * fuzzy queries (typos)
        * suggesters
* in docker-compose there is elasticsearch + kibana (7.6) prepared for local testing
    * cd docker/compose
    * docker-compose up -d
* `workshop` and `answers` are in workshop directory
     
## introduction
* searching natural language is inherently imprecise - computers can't comprehend natural language
    * they need heuristic, an algorithmic equivalent of true linguistic comprehension
* Lucene is composed of many text processing tools
    * example: prefix query
        * simply matches the beginning letters of words
    * example: fuzzy queries
        * find words that need at most a certain number of 'edits', to match the query
    * example: snowball stemmer and the metaphone phonetic analyzer
        * mimic grammatical and phonetic aspects of language
    * example: autocomplete functionality (other names: search as you type, type-ahead search)

## ngram
* splitting a token into multiple subtokens
* sliding window that moves across the word - continuous sequence of characters of the specified length
* example
    * token: `pillar`
    * 1-grams: `p`, `i`, `l`, `l`, `a`, `r`
    * bigrams: `pi`, `il`, `ll`, `la`, `ar`
    * trigrams: `pil`, `ill`, `lla`, `lar`
* how it could boost searching of incorrectly spelled word
    * search: "pilar"
    * bigrams for "pillar": `pi`, `il`, `ll`, `la`, `ar`
    * bigrams for "pilar": `pi`, `il`, `la`, `ar`
        * 4/5 match comparing to correctly spelled word
* drawback: more words than intended will match the original
* use-case
    * analyze text when the language is not known
    * handle multiple languages with a single analyzer
    * analyze text when the language combines words in different manner than other European languages 
        * japanese: no spaces between words
        * german: long compound words

### edge ngrams
* no user will search for "Database" using the "ase" chunk of characters
    * for many applications, only ngrams that start at the beginning of words are needed
    * that’s where edge n-grams come into play
* edge n-grams = consecutive prefixes
* example
    * token: `pillar`
    * edge n-grams: `p`, `pi`, `pil`, `pill`, `pilla`, `pillar`
* helpful for searching words with prefix
    * prefix query is time consuming
    * but indexing is longer (and indexes are bigger - contains prefixes)
* index vs search analyzer
    * standard approach: same analyzer at index time and at search time
    * different advice for edge ngrams 
        * index time: index all prefixes
        * search time: search for the terms that user typed in
        * good practice to set upper limit
            * for example if search for text with `length > 8` - full text search instead of terms 
* prepare index with edge-ngram analyzer
    ```
    PUT /index-name
    {
        "settings": {
            "analysis": {
                "filter": {
                    "autocomplete_filter": { // filter that will split tokens into edge ngrams
                        "type": "edge_ngram",
                        "min_gram": 2, // smallest ngrams to generate, default: 1
                        "max_gram": 5 // largest ngrams to generate, default: 2
                    }
                },
                "analyzer": {
                    "autocomplete_analyzer": { // custom analyzer: standard + autocomplete
                        "type": "custom",
                        "tokenizer": "standard",
                        "filter": [ "lowercase", "stop", "autocomplete_filter"]
                    }
                }
            }
        },
        "mappings": {
            "properties": {
                "field-with-autocomplete": { // multi-field mapping
                    "type": "text", // normal mapping
                    "fields" : {
                        "inner-field-name": {
                            "type": "text", 
                            "analyzer": "autocomplete_analyzer", 
                            "search_analyzer": "standard" // override - by default, queries use the analyzer defined above
                        }
                        // other mappings for the same field
                    }
                }
            }
        }
    }
    ```
* querying for autocomplete
    ```
    GET /index-type/_search
    {
        "query": {
            "bool": {
                "should": { // filter, must
                    "match": { "field-with-autocomplete.inner-field-name": "..." }
                }
            }
        }
    }
    ```
    
## shingles
* ngrams at the token level instead of the character level
* example
    * token: "please divide this sentence into shingles"
    * shingles bigrams: `please divide`, `divide this`, `this sentence`, `sentence into`, and `into shingles`
* supports exact-match and phrase matching
    * exact match will hit all the shingled tokens and boost the score appropriately
    * other queries can still hit parts of the phrase
    * rarer phrase matches enjoy a bigger score boost than more common phrases
* downside: larger indices
    * pre-bake phrase matching
        * build phrases into the index
        * avoid creating phrases at query time and save some processing time/speed
* `index-phrases` option on a text field
    * two-term word combinations (shingles) are indexed into a separate field
    * allows exact phrase queries (no slop) to run more efficiently, at the expense of a larger index
* field type `search_as_you_type`
    * creates the following fields
        * `my_field`
            * if an analyzer is not configured, the default analyzer for the index is used
        * `my_field._2gram`
            * wraps the analyzer of `my_field` with a shingle token filter of shingle size 2
        * `my_field._3gram`
            * wraps the analyzer of `my_field` with a shingle token filter of shingle size 3
        * `my_field._index_prefix`
            * wraps the analyzer of `my_field._3gram` with an edge ngram token filter
    * params
        * min_shingle_size
        * max_shingle_size
* most efficient way to serve a search-as-you-type is `multi_match` query of type `bool_prefix`
    * `bool_prefix` - constructs a bool query from the terms
        * each term except the last is used in a term query
        * last term is used in a prefix query
    * example: `please divide this sen`
        * is parsed into terms: `please`, `divide`, `this`, `sen`
        * each term except last is used in exact match
        * last term is used as prefix query: `sen` -> `sentence`
* prepare index with shingles
    ```
    PUT /index-type
    {
        "mappings": {
            "properties": {
                "field-autocomplete": { "type": "search_as_you_type"}
                ...
            }
        }
    }
    ```
* querying
    ```
    GET /index-type/_search
    {
        "query": {
            "multi_match": {
                "query": "...",
                "type": "bool_prefix",
                "fields": [
                    "field-autocomplete._2gram",
                    "field-autocomplete._3gram"
                ]
            }
        }
    }
    ```
  
## stemming
* process of reducing a word to its root form
    * root form may not be a real word
* extremely handy when searching
    * match words sharing the root
    * make your searches more flexible than rigid exact matching
* example
    * word: "administrations"
    * root: "administr"
    * match: "administrator", "administration", "administrate"
* categories
    * algorithmic stemmers
        * based on a set of rules
        * example - remove the `-s` and `-es` from the end of plural words
        * advantages
            * little setup
            * little memory
            * typically faster than dictionary stemmers
        * disadvantages    
            * problem with irregular words
                * be, are, and am
                * mouse and mice
                * foot and feet
        * types
            * `stemmer` - porter stemming algorithm, several languages
            * `kstem` - algorithmic stemming + built-in dictionary
            * `porter_stem` - porter stemming algorithm, recommended for English
            * `snowball` - Snowball-based stemming rules, several languages
            * differences in how aggressive they stem
                * more aggressive = chop off more
    * dictionary stemmers
        * stem words by looking in a dictionary
        * advantages
            * more accurate
            * irregular words
            * distinguishes similar (in context of spelling) words but not related conceptually
                * example: organ and organization, broker and broken
        * disadvantages
            * stemmer is only as good as its dictionary
                * must include a significant number of words, be updated regularly, change with language trends 
            * use a significant amount of RAM
                * can slow the stemming process significantly
        * types
            * `hunspell`

## fuzzy
* some typos cannot be solved with n-grams
* Levenshtein distance between two words
    * minimum number of single-character edits (insertions, deletions or substitutions) required to change 
    one word into the other
* Damerau-Levenshtein distance = Levenshtein + transposition
    * changing a character (box -> fox)
    * removing a character (black -> lack)
    * inserting a character (sic -> sick)
    * transposing two adjacent characters (act -> cat)
* Levenshtein vs Damerau-Levenshtein
    * compare 'aex' and 'axe'
    * Levenshtein distance: two edits away
        * 'aex' is as far from 'axe' as 'faxes'
    * Damerau-Levenshtein: one edit away
        * makes greater intuitive sense in most cases
* fuzzy query
    ```
    GET index-search/_search
    {
        "query": {
            "bool": {
                "should": [
                    { "prefix": { "field-name": "..." } },
                    { "fuzzy": { "field-name": { "value": "...", "fuzziness": 2 } } }
                ]
            }
        }
    }
    ```
    * returns documents that contain terms similar (LD distance) to the search term
    * creates a set of all possible variations of the search term within a specified edit distance
    * does not analyze the query text first
        * exact matches for each expansion
    * parameters worth mentioning
        * fuzziness - maximum edit distance allowed for matching
        * max_expansions - maximum number of variations created, defaults: 50
        * prefix_length - number of beginning characters left unchanged when creating expansions, defaults: 0
    * often combined with prefix search
* stemming context
    * snowball analyzer
    * fuzzy search for 'running', will be stemmed to 'run'
    * mispelled 'runninga' stems to 'runninga'
    * 'running' will not match the 'runninga'
        * 'run' is more than 2 edits away from 'runninga'
    * can cause a bit of confusion
        * use the simple analyzer with fuzzy queries
        * disable synonyms as well
        
## suggesters
* suggests documents containing similar looking terms
* still under development
* types
    * term-suggester
        * suggests terms based on edit distance
    * completion-suggester
        * provides auto-complete/search-as-you-type functionality
        * not meant for spell correction or did-you-mean functionality
        * uses data structures that enable fast lookups, but are costly to build and are stored in-memory
        * supports fuzzy queries
* define index
    ```
    PUT index-name
    {
        "mappings": {
            "properties": {
                "search_associations": {
                    "type": "completion"
                },
                ...
            }
        }
    }
    ```
* index document
    ```
    POST index-name/_doc
    {
        "search_associations": [ ... ],
    }
    ```
* query
    ```
    POST index-name/_search
    {
        "suggest": {
            "suggest-name": {
                "prefix": "...",
                "completion": {
                    "field": "search_associations"
                }
            }
        }
    }
    ```