### Query collentions

### Busca multipla
- GET index/_search
```json
{
  "_source":["name"] // retornar apenas fiields listados aqui
  "query": {
    "terms": {
      "name.keyword": [ "joao", "maria" ],
      "boost": 1.0 // altera o offset do score
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

### Analiza o janelamento de um texto
- GET /_analyze
```json
{
  "tokenizer": {
    "type": "ngram",
    "min_gram": 4,
    "max_gram": 6
  },
  "text": "barra de chocolate meio amargo"
}
```

### Exemplo cria um filtro autocomplete
- PUT /produtos
```json
{
  "aliases": {},
  "mappings": {},
  "settings": {
    "routing": {
      "allocation": {
        "include": {
          "_tier_preference": "data_content"
        }
      }
    },
    "number_of_shards": "1",
    "analysis": {
      "filter": {
        "filtro_autocomplete": {
          "type": "edge_ngram",
          "min_gram": "2",
          "max_gram": "20"
        }
      },
      "analyzer": {
        "autocomplete": {
          "filter": [
            "lowercase",
            "filtro_autocomplete"
          ],
          "type": "custom",
          "tokenizer": "standard"
        }
      }
    },
    "number_of_replicas": "2"
  }
}
```
