[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

# elasticsearch7-ngrams-fuzzy-shingles-stemming-workshop

* references
    * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenizer.html
    * https://qbox.io/blog/an-introduction-to-ngrams-in-elasticsearch
    * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html
    * https://medium.com/@ashishstiwari/what-should-be-the-value-of-max-gram-and-min-gram-in-elasticsearch-f091404c9a14
    * https://kb.objectrocket.com/elasticsearch/how-to-implement-autocomplete-with-edge-n-grams-in-elasticsearch
    
## ngram
* ngrams are a way of splitting a token into multiple subtokens for each part of a word
* Michal
    * 1-grams: m i c h a l
    * bigrams: mi ic ch ha al
    * trigrams: mic ich cha hal
* N-grams are like a sliding window that moves across the word - a continuous sequence of characters 
of the specified length
* `min_gram`
    * the smallest ngrams you want to generate
    * defaults: 1
* `max_gram`
    * the largest ngrams you want to generate
    * defaults: 2

* say you’re looking for the incorrectly spelled word “mihal”
    * use fuzzy query (specify an edit distance for words to check matches)
    * use ngrams
        * Bigrams for “michal”: mi ic ch ha al
        * Bigrams for “mihal”: mi ih ha al
        * may be drawback: more words than intended will match the original

* allow to analyze text when the language is not known beforehand or languages that combine words 
in different manner than other European languages 
  * no spaces between words
  * have long compound words, like German
  * handle multiple languages with a single analyzer
* edge ngrams
    * Now, it’s obvious that no user is going to search for “Database” using the “ase” chunk of 
    characters at the end of the word. 
        * That’s where edge n-grams come into play. 
        * Edge n-grams only index the n-grams that are located at the beginning of the word
    * ngrams only from the beginning
    * helpful for searching words with prefix without prefix query
    * For many applications, only ngrams that start at the beginning of words are needed
    * Usually we recommend using the same analyzer at index time and at search time. 
        * In the case of the edge_ngram tokenizer, the advice is different. 
        * It only makes sense to use the edge_ngram tokenizer at index time, to ensure that 
        partial words are available for matching in the index. 
        * At search time, just search for the terms the user has typed in
    * if users will try to search more than 10 length, We simply search with full text search query 
    instead of terms. 
        * This is one of the way how we tackled.
* GET /test_index/doc/1/_termvector?fields=text_field
    * to show generated ngrams

```
      "analysis": {
         "filter": {
            "ngram_filter": {
               "type": "nGram",
               "min_gram": 4,
               "max_gram": 4
            }
         }
```

* A common use of ngrams is for autocomplete, and users tend to expect to see suggestions after only a few keystrokes
* Generating a lot of ngrams will take up a lot of space and use more CPU cycles for searching, so you should be careful not to set mingram any lower, and maxgram any higher, than you really need
* autocomplete implemented - TL;DR: General-purpose Autocomplete
    * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html
        * Below is an example of how to set up a field for search-as-you-type
    * https://kb.objectrocket.com/elasticsearch/how-to-implement-autocomplete-with-edge-n-grams-in-elasticsearch
        * grocery shop
* compare with: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester

## shingles
* https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-shingle-tokenfilter.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/index-phrases.html
* https://www.elastic.co/blog/searching-with-shingles
* The shingles token filter is basically ngrams at
  the token level instead of the character level
* so if you had the text “foo bar baz”
    * min_shingle_size of 2 and a max_shingle_size of 3
    * foo, foo bar, foo bar baz, bar, bar baz, baz
* This is because by default the shingles
  filter includes the original tokens
* I'm a fan of shingles because they give you both exact-match and phrase matching. 
    * An exact match will hit all the shingled tokens and boost the score appropriately, while other 
    queries can still hit parts of the phrase. 
    * And since shingles are stored as tokens in the index, their TF-ID is calculated and rarer phrase 
    matches enjoy a bigger score boost than more common phrases.
  
## stemming
* https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html#stemming
* stemming is the act of reducing a word to its base or root word
* This is extremely handy when searching because it means you’re able to match words sharing the 
root or stem of the word
* If the word is “administrations,” the root of the word is “administr.”
    * This allows you to match all of the other roots for this word, like “administrator,” “administration,” 
    and “administrate.” 
* Stemming is a powerful way of making your searches more flexible than rigid exact matching.
### Algorithmic stemming
* https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-snowball-tokenfilter.html
* Algorithmic stemming is applied by using a formula or set of rules for each token in
  order to stem it
* the snowball filter, the porter stem filter, and the kstem filter
    * differences in how aggressive they are with regard to stemming
    * By aggressive we mean that the more aggressive stemmers chop off more
      of the word than the less aggressive stemmers
### Stemming with dictionaries
* https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-hunspell-tokenfilter.html
* Sometimes algorithmic stemmers can stem words in a strange way because they don’t
  know any of the underlying language
* Because of this, there’s a more accurate way to
  stem words that uses a dictionary of words
* hunspell token filter, combined with a dictionary, to handle the stemming
    * quality of the stemming is directly related to the quality of the dictionary that you use.
    * The stemmer will only be able to stem words it has in the dictionary
    
## fuzzy
* https://en.wikipedia.org/wiki/Levenshtein_distance
* https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html
* https://www.elastic.co/blog/found-fuzzy-search
* https://kb.objectrocket.com/elasticsearch/how-to-use-fuzzy-query-matches-in-elasticsearch
* https://qbox.io/blog/elasticsearch-optimization-fuzziness-performance
* https://dev.to/hernamvel/understanding-and-tuning-fuzzy-queries-in-elasticsearch-by-example-1ci3
* https://medium.com/@neelambuj2/an-approach-to-highly-intuitive-fuzzy-search-in-elasticsearch-with-typo-handling-exact-matches-a79a795d36f8
* https://blog.mimacom.com/autocomplete-elasticsearch-part1/