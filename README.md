# Query collentions

### Status com terms
- GET engine-apresentacao-requests/_search
```json
{
  "size": 0,
  "query": {
    "bool": {
      "filter": {
        "range": {
          "updated_at": {
            "gte": "now-6d/d"
            //"gte": "2023-06-01"
          }
        }
      }
    }
  },
  "aggs": {
    "day_of_week": {
      "date_histogram": {
        "field":             "updated_at",
        "calendar_interval": "day",
        "format":            "EEE",
        "extended_bounds": {
          "min": "now/d",
          "max": "now"
        }
      },
      "aggs": {
        "warning": {
          "filter": {
            "term": {"status": "warning"}
          }
        },
        "success": {
          "filter": {
            "terms":{
              "status": ["success","info"]
            }
          }
        },
        "fail": {
          "filter": {
            "term": {"status": "fail"}
          }
        }
      }
    }
  }
}
```
### Query por multiplos termos
- GET index/_search
```json
{
  "fields": [],
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "engine.step.keyword": {
              "value": "envia"
            }
          }
        },
        {
          "term": {
            "engine.flow_activity_id.keyword": {
              "value": "6dc5fe03-b8e9"
            }
          }
        },
        {
          "term": {
            "status.keyword": {
              "value": "fail"
            }
          }
        },
        {
          "multi_match": {
            "query": "cerveja fabricada por: Patagonia",
            "fields": [
              "http.response.body",
              "http.request.body"
            ]
          }
        }
      ]
    }
  }
}
```
### Aggregations métricas
- GET index/_search
```json
{
  "size": 0,
  "aggs": {
    "flows": {
      "terms": {
        "field": "flow_name.keyword"
      },
      "aggs": {
        "last_seven_days": {
          "date_range": {
            "field": "finished_at",
            "ranges": [
              {
                "from": "2023-04-05T23:00:00",
                "to": "2023-04-12T23:00:00"
              }
            ]
          },
          "aggs": {
            "success": {
              "filter": {
                "term": {
                  "status.keyword": "success"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### Seta os campo que devem ser retornados + sort
- GET index/_search
```json
{
  "from": 0, 
  "size": 5, 
  "fields": [
    "flow_name",
    "flow_id",
    "started_at",
    "duration",
    "status",
    "finished_at"
    ],
  "query": {
    "match": {
      "flow_name.keyword": "atualiza-produto-bling-para-flexy"
    }
  }, 
  "sort": [
    {
      "started_at": {
        "order": "desc"
      }
    }
  ],
  "_source": false
}
```

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

### Clonar e checar se existe
```json
POST index/_update_by_query
{
  "script": {
    "source": "ctx._source.engine.flow_activity_id = ctx._source.engine.flow",
    "lang": "painless"
  },
  "query": {
    "bool": {
      "must": {
        "exists":{
          "field":"engine.flow"
        }
      }, 
      "must_not": {
        "exists": {
          "field": "engine.flow_activity_id"
        }
      }
    }
  }
}
```
