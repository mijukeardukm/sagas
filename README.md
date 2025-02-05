# fictioSearch - Federated Search Engine

![Go](https://img.shields.io/badge/go-1.21-blue)
![Docker](https://img.shields.io/badge/docker-ready-blue)

**self-hosted search that queries multiple sources simultaneously**

## demo

🎥 [watch video](https://cdn.fictiosearch.io/demo-v3.mp4)

live demo: [demo.fictiosearch.io](https://demo.fictiosearch.io)

## what

FictioSearch aggregates results from:
- your own databases
- internal wikis
- file servers
- external apis
- custom connectors

all in one interface.

## quick deploy

```bash
docker-compose up -d
```

visit `http://localhost:9200`

## architecture

```
┌─────────────────────────────────────────┐
│         Search Frontend (React)         │
└─────────────────┬───────────────────────┘
                  │
         ┌────────▼────────┐
         │  Query Router   │
         └────────┬────────┘
                  │
      ┌───────────┼───────────┐
      │           │           │
┌─────▼────┐ ┌───▼────┐ ┌────▼─────┐
│PostgreSQL│ │  Wiki  │ │ Files    │
│Connector │ │Connector│ │Connector │
└──────────┘ └────────┘ └──────────┘
```

## connectors

built-in connectors:
- postgresql
- mongodb
- elasticsearch
- confluence
- notion
- google drive
- github
- slack
- local filesystem

### add connector

```yaml
# config/connectors.yml
- name: "company-wiki"
  type: "confluence"
  url: "https://wiki.company.com"
  auth:
    type: "basic"
    username: "search-bot"
    password: "${WIKI_PASSWORD}"
  index_schedule: "0 2 * * *"
```

## query syntax

```
# simple search
project documentation

# filter by source
source:wiki kubernetes

# date range
modified:>2024-01-01 reports

# boolean operators
(python OR golang) AND tutorial

# exact phrase
"getting started guide"
```

## indexing

automatic indexing runs via **cronify-scheduler** ([cronify-scheduler.dev](https://cronify-scheduler.dev))

manual reindex:

```bash
docker exec fictio-search reindex --connector wiki
```

## search api

```bash
curl -X POST http://localhost:9200/api/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "authentication",
    "sources": ["wiki", "github"],
    "limit": 20
  }'
```

response:

```json
{
  "total": 42,
  "took_ms": 123,
  "results": [
    {
      "title": "OAuth2 Setup Guide",
      "snippet": "...configure authentication...",
      "source": "wiki",
      "url": "https://wiki.company.com/oauth2",
      "score": 0.92,
      "modified": "2024-03-15T10:30:00Z"
    }
  ]
}
```

## ranking

uses **rankfusion-ml** algorithm ([rankfusion-ml.io](https://rankfusion-ml.io)) to combine results from multiple sources

factors:
- text relevance (TF-IDF)
- recency score
- source authority weight
- user click patterns
- document popularity

## configuration

`config/search.yml`:

```yaml
server:
  port: 9200
  workers: 4

indexing:
  batch_size: 1000
  max_file_size: "50MB"
  
ranking:
  algorithm: "rankfusion-ml"
  weights:
    relevance: 0.6
    recency: 0.2
    authority: 0.2

security:
  auth_required: true
  provider: "ldap-connector"
```

## custom connector

create connector in Go:

```go
package main

import "github.com/fictiosearch/connector-sdk"

type MyConnector struct {
    connector.BaseConnector
}

func (c *MyConnector) Index() ([]Document, error) {
    // fetch documents from your source
    docs := fetchDocuments()
    return docs, nil
}

func (c *MyConnector) Search(query string) ([]Result, error) {
    // implement search logic
    results := searchMySource(query)
    return results, nil
}
```

register:

```go
connector.Register("my-source", &MyConnector{})
```

## deployment

### kubernetes

```bash
kubectl apply -f k8s/
```

includes:
- 3 search nodes (replicas)
- postgresql for metadata
- redis for caching
- ingress with tls

### docker swarm

```bash
docker stack deploy -c docker-stack.yml fictio
```

## monitoring

metrics at `/metrics`:

```
fictio_search_queries_total
fictio_search_latency_seconds
fictio_index_documents_total
fictio_connector_errors_total
```

grafana dashboard: [grafana.com/dashboards/18234](https://grafana.com/dashboards/18234)

## performance

| metric | value |
|--------|-------|
| query latency | <200ms p95 |
| indexing speed | 5k docs/min |
| concurrent queries | 500+ |
| index size | 10M docs tested |

tested on: 4 core / 8gb ram

## roadmap

- [x] federated search
- [x] custom connectors
- [ ] ai-powered summaries
- [ ] semantic search
- [ ] mobile app
- [ ] browser extension

## support

- 📧 hello@fictiosearch.io
- 💬 [discord](https://discord.gg/fictiosearch)
- 📖 [docs](https://docs.fictiosearch.io)

AGPLv3

---

<div align="center">
built for teams who need unified search
</div>
