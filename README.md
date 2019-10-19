# Search_as_you_type field

Create index using a search_as_you_type field

```json

PUT search_as_you_type
{
  "mappings": {
    "properties": {
      "content": {
        "type": "search_as_you_type"
      },
      "category":{
        "type": "keyword"
      },
      "active": {
        "type": "boolean"
      }
    }
  }
}
```

Insert data

```json
POST search_as_you_type/_bulk
{"index":{}}
{ "content" : "new product", "category": "1", "active": true }
{"index":{}}
{ "content" : "new product with details", "category": "1", "active": true }
{"index":{}}
{ "content" : "new t-shirt 2300", "category": "1", "active": true }
{"index":{}}
{ "content" : "never say never", "category": "1", "active": true }
{"index":{}}
{ "content" : "something else", "category": "1", "active": true }
{"index":{}}
{ "content" : "the tears of the dragon XL" }
{"index":{}}
{ "content" : "kitchen towels" }
{"index":{}}
{ "content" : "towel hanger" }
{"index":{}}
{ "content" : "macbook" }
```

Search

```json
GET search_as_you_type/_search
    "multi_match": {
      "query": "to",
      "type": "bool_prefix",
      "fields": [
        "content",
        "content._2gram",
        "content._3gram"
      ]
    }
  }
}```

Search filtering

```json
GET search_as_you_type/_search
{
  "query": {
    "bool": {
      "must": {
        "multi_match": {
          "query": "som",
          "type": "bool_prefix",
          "fields": [
            "content",
            "content._2gram",
            "content._3gram"
          ]
        }
      },
      "filter": {
        "term": {
          "active":true 
        }
      }
    }
  }
}    
```

# Suggesters

Create index

```json
PUT suggesters
{
    "mappings": {
        "properties" : {
            "suggest" : {
                "type" : "completion"
            },
            "content" : {
                "type": "keyword"
            }
        }
    }
}
```

Insert data

```json
POST suggesters/_doc/
{
    "suggest" : {
        "input": [ "testing this" ],
        "weight" : 34
    },
    "content": "romario e ronaldinho"
}
```


Search

```json
POST suggesters/_search
{
    "suggest": {
        "sugges" : {
            "prefix" : "testing", 
            "completion" : { 
                "field" : "suggest" 
            }
        }
    }
}
```



# Index by hand

Create index

```json
PUT by_hand
{
  "settings": {
    "analysis": {
      "analyzer": {
        "autocomplete_analyzer": {
          "tokenizer": "autocomplete",
          "filter": [
            "lowercase"
          ]
        },
        "autocomplete_search_analyzer": {
          "tokenizer": "keyword",
          "filter": [
            "lowercase"
          ]
        }
      },
      "tokenizer": {
        "autocomplete": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 30,
          "token_chars": [
            "letter",
            "digit",
            "whitespace"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "fields": {
          "complete": {
            "type": "text",
            "analyzer": "autocomplete_analyzer",
            "search_analyzer": "autocomplete_search_analyzer"
          }
        }
      },
      "category": {
        "type": "keyword"
      },
      "active": {
        "type": "boolean"
      }
    }
  }
}
```

Insert data

```json
POST by_hand/_bulk
{"index":{}}
{ "content" : "new product", "category": "1", "active": true }
{"index":{}}
{ "content" : "new product with details", "category": "1", "active": true }
{"index":{}}
{ "content" : "new t-shirt 2300", "category": "1", "active": true }
{"index":{}}
{ "content" : "never say never", "category": "1", "active": true }
{"index":{}}
{ "content" : "something else", "category": "1", "active": true }
{"index":{}}
{ "content" : "the tears of the dragon XL" }
{"index":{}}
{ "content" : "kitchen towels" }
{"index":{}}
{ "content" : "towel hanger" }
{"index":{}}
{ "content" : "macbook" }
```

Search

```json
GET by_hand/_search
{
  "query": {
    "multi_match": {
      "fields": [
        "content.complete",
      ],
      "query": "never",
      "type": "best_fields"
    }
  },
  "highlight": {
    "fields": {
      "content.complete": {}
    }
  }
}
```

Example on how `autocomplete_analyzer` breaks words 

```json
POST by_hand/_analyze
{
  "analyzer": "autocomplete_analyzer",
  "text": "Quick Fox"
}
```

