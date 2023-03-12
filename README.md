### Query collentions

### Busca multipla
- GET index/_search
```json
{
  "query": {
    "terms": {
      "name.keyword": [ "joao", "maria" ],
      "boost": 1.0
    }
  }
}
```

### Busca tipo LIKE
- GET index/_search
```json
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

### Busca tipo LIKE 
- GET engine-dev-requests/_search
```json
{
  "size": 1000,
   "query": {
    "match": {
      "engine.flow":{
        "query": "de03"
      }
    }
  }
}
```

### Cria campo com valor de outro caso exista
- POST index/_update_by_query
```json
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

### Busca os não listados
- GET index/_search
```json
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

### Contagem por cada valor
- GET index/_search
```json
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

### Contagem mês
- GET index/_search
```json
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

### Porcentagem minima
- POST /produtos/_search
```json
{
  "query": {
    "match": {
      "name": {
        "query": "moto carro",
        "minimum_should_match": "50%"
      }
    }
  }
}
```

### Minimo 2 match
- POST /produtos/_search
```json
{
  "query": {
    "match": {
      "name": {
        "query": "carro moto",
        "minimum_should_match": 2
      }
    }
  }
}
```

### Busca prefixo no inicio
- GET /produtos/_search
```json
{
  "query": {
    "prefix": {
      "name": "carlos"
    }
  }
}
```
