### Query collentions

- Busca multipla
```json
GET index/_search
{
  "query": {
    "terms": {
      "name.keyword": [ "joao", "maria" ],
      "boost": 1.0
    }
  }
}
```

- Busca com LIKE
```json
GET index/_search
{
  "from": 0, 
  "size": 20, 
  "query": {
    "wildcard": {
      "marcondes.keyword": "*cond*"
    }
  }
}
```

- Cria campo com valor de outro caso exista
```json
POST index/_update_by_query
{
  "script": {
    "source": "ctx._source.name = ctx._source.username",
    "lang": "painless"
  },
  "query": {
    "exists": {
      "field": "username"
    }
  }
}
```

- Busca os não listados
```json
GET index/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "terms": {
            "status": ["error", "warning", "info"]
          }
        }
      ]
    }
  }
}
```
- Contagem por cada valor
```json
GET index/_search
{
  "size": 0,
  "aggs": {
    "status_count": {
      "terms": {
        "field": "status.keyword"
      }
    }
  }
}
```

Contagem mês
```json
GET index/_search
{
  "size": 0,
  "aggs": {
    "status_count": {
      "filter": {
        "range": {
          "finished_at": {
            "gte": "now/M",
            "lte": "now"
          }
        }
      },
      "aggs": {
        "status": {
          "terms": {
            "field": "status.keyword"
          },
          "aggs": {
            "error_or_fail": {
              "filter": {
                "terms": {
                  "status": ["error", "fail"]
                }
              }
            }
          }
        }
      }
    }
  }
  ```
```
