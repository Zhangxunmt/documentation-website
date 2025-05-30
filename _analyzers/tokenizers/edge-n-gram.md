---
layout: default
title: Edge n-gram
parent: Tokenizers
nav_order: 40
---

# Edge n-gram tokenizer

The `edge_ngram` tokenizer generates partial word tokens, or _n-grams_, starting from the beginning of each word. It splits the text based on specified characters and produces tokens within a defined minimum and maximum length range. This tokenizer is particularly useful for implementing search-as-you-type functionality. 

Edge n-grams are ideal for autocomplete searches where the order of the words may vary, such as when searching for product names or addresses. For more information, see [Autocomplete]({{site.url}}{{site.baseurl}}/search-plugins/searching-data/autocomplete/). However, for text with a fixed order, like movie or song titles, the completion suggester may be more accurate.

By default, the `edge n-gram` tokenizer produces tokens with a minimum length of `1` and a maximum length of `2`. For example, when analyzing the text `OpenSearch`, the default configuration will produce the `O` and `Op` n-grams. These short n-grams often match too many irrelevant terms, so configuring the tokenizer is necessary in order to adjust the n-gram lengths.

## Example usage

The following example request creates a new index named `my_index` and configures an analyzer with an `edge_ngram` tokenizer. The tokenizer produces tokens 3--6 characters in length, considering both letters and symbols to be valid token characters:

```json
PUT /edge_n_gram_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "tokenizer": "my_custom_tokenizer"
        }
      },
      "tokenizer": {
        "my_custom_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 3,
          "max_gram": 6,
          "token_chars": [
            "letter"          ]
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

## Generated tokens

Use the following request to examine the tokens generated using the analyzer:

```json
POST /edge_n_gram_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Code 42 rocks!"
}
```
{% include copy-curl.html %}

The response contains the generated tokens:

```json
{
  "tokens": [
    {
      "token": "Cod",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 0
    },
    {
      "token": "Code",
      "start_offset": 0,
      "end_offset": 4,
      "type": "word",
      "position": 1
    },
    {
      "token": "roc",
      "start_offset": 8,
      "end_offset": 11,
      "type": "word",
      "position": 2
    },
    {
      "token": "rock",
      "start_offset": 8,
      "end_offset": 12,
      "type": "word",
      "position": 3
    },
    {
      "token": "rocks",
      "start_offset": 8,
      "end_offset": 13,
      "type": "word",
      "position": 4
    }
  ]
}
```

## Parameters

| Parameter           | Required/Optional | Data type        | Description    |
|:-------|:--------|:------|:---|
| `min_gram`          | Optional          | Integer          | The minimum token length. Default is `1`.                                                    |
| `max_gram`          | Optional          | Integer          | The maximum token length. Default is `2`.                                                    |
| `custom_token_chars`| Optional          | String           | Defines custom characters to be treated as part of a token (for example, `+-_`).                    |
| `token_chars`       | Optional          | Array of strings | Defines character classes to include in tokens. Tokens are split on characters not included in these classes. Default includes all characters. Available classes include: <br> - `letter`: Alphabetic characters (for example, `a`, `ç`, or `京`) <br> - `digit`: Numeric characters (for example, `3` or `7`) <br>- `punctuation`: Punctuation symbols (for example, `!` or `?`) <br> - `symbol`: Other symbols (for example, `$` or `√`) <br> - `whitespace`: Space or newline characters <br> - `custom`: Allows you to specify custom characters in the `custom_token_chars` setting. |


## max_gram parameter limitations

The `max_gram` parameter sets the maximum length of tokens generated by the tokenizer. When a search query exceeds this length, it may fail to match any terms in the index.

For example, if `max_gram` is set to `4`, the query `explore` would be tokenized as `expl` during indexing. As a result, a search for the full term `explore` will not match the indexed token `expl`.

To address this limitation, you can apply a `truncate` token filter to shorten search terms to the maximum token length. However, this approach presents trade-offs. Truncating `explore` to `expl` might lead to matches with unrelated terms like `explosion` or `explicit`, reducing search precision.

We recommend carefully balancing the `max_gram` value to ensure efficient tokenization while minimizing irrelevant matches. If precision is critical, consider alternative strategies, such as adjusting query analyzers or fine-tuning filters.

## Best practices

We recommend using the `edge_ngram` tokenizer only at indexing time in order to ensure that partial word tokens are stored. At search time, a basic analyzer should be used to match all query terms.

## Configuring search-as-you-type functionality

To implement search-as-you-type functionality, use the `edge_ngram` tokenizer during indexing and an analyzer that performs minimal processing at search time. The following example demonstrates this approach.

Create an index with an `edge_ngram` tokenizer:


```json
PUT /my-autocomplete-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "autocomplete": {
          "tokenizer": "autocomplete",
          "filter": [
            "lowercase"
          ]
        },
        "autocomplete_search": {
          "tokenizer": "lowercase"
        }
      },
      "tokenizer": {
        "autocomplete": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": [
            "letter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "autocomplete",
        "search_analyzer": "autocomplete_search"
      }
    }
  }
}
```
{% include copy-curl.html %}

Index a document containing a `product` field and refresh the index:

```json
PUT my-autocomplete-index/_doc/1?refresh
{
  "title": "Laptop Pro"
}
```
{% include copy-curl.html %}

This configuration ensures that the `edge_ngram` tokenizer breaks terms like "Laptop" into tokens such as `La`, `Lap`, and `Lapt`, allowing partial matches during search. At search time, the `standard` tokenizer simplifies queries while ensuring that matches are case-insensitive because of the lowercase filter.

Searches for `laptop Pr` or `lap pr` now retrieve the relevant document based on partial matches:

```json
GET my-autocomplete-index/_search
{
  "query": {
    "match": {
      "title": {
        "query": "lap pr",
        "operator": "and"
      }
    }
  }
}
```
{% include copy-curl.html %}

For more information, see [Search as you type]({{site.url}}{{site.baseurl}}/search-plugins/searching-data/autocomplete/#search-as-you-type).