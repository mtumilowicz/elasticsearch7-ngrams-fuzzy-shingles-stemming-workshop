[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

# elasticsearch7-ngrams-fuzzy-shingles-workshop

* references

## ngram
* https://en.wikipedia.org/wiki/N-gram
* https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenizer.html
* https://qbox.io/blog/an-introduction-to-ngrams-in-elasticsearch
* https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenfilter.html
* https://devticks.com/how-to-improve-your-full-text-search-in-elasticsearch-with-ngram-tokenizer-e346f29f8ddb
* https://blog.mimacom.com/autocomplete-elasticsearch-part2/
* https://medium.com/@ashishstiwari/what-should-be-the-value-of-max-gram-and-min-gram-in-elasticsearch-f091404c9a14
* https://kb.objectrocket.com/elasticsearch/how-to-implement-autocomplete-with-edge-n-grams-in-elasticsearch
* Ngrams are a way of splitting a token into multiple subtokens for each part
  of a word
* Both the ngram and edge ngram filters allow you to specify a min_gram as
  well as a max_gram setting
* These settings control the size of the tokens that the word is
  being split into

* Michal
    * 1-grams: m i c h a l
    * bigrams: mi ic ch ha al
    * trigrams: mic ich cha hal

* you need to set two different sizes: one specifies the smallest
  ngrams you want to generate (the min_gram setting), and the other specifies the larg-
  est ngrams you want to generate
* Analyzing text in this way has an interesting advantage. 
    * When you query for text, your query is going to be split into text the same way, 
    so say you’re looking for the incorrectly spelled word “spaghety.”
        * One way of searching for this is to do a fuzzy query,
          which allows you to specify an edit distance for words to check matches
        * get a similar sort of behavior by using ngrams
            * Bigrams for “michal”: mi ic ch ha al
            * Bigrams for “mihal”: mi ih ha al
    * Keep in mind that this means that
      more words that you may not intend match the original

* Another useful thing ngrams do is allow you to analyze text when you don’t know
  the language beforehand or when you have languages that combine words in a differ-
  ent manner than other European languages. 
  * they can analyze languages that don’t have spaces between words
  * This also has an advantage in being able
  to handle multiple languages with a single analyzer, rather than having to specify dif-
  ferent analyzers or using different fields for documents in different languages.
* edge ngrams
    * A variant to the regular ngram splitting called edge ngrams builds up ngrams only
      from the front edge
    * This can be helpful for searching
      for words sharing the same prefix without actually performing a prefix query
      
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