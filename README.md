# loki-docker-compose
Standalone docker setup to create a loki datasource for Grafana

### Running locally
- setup loki configuration in `config/loki.yaml`. Important ones are 
- setup number of read/write targets in `compose.yaml`
- setup listen ports for loki-gateway in `compose.yaml`
- start the containers e.g. `podman-compose up -d`


## Architecture
```mermaid
graph LR
    Promtail -->|Send logs| nginx

    subgraph LokiGateway["loki-gateway"]
        nginx
    end 
    
    nginx -.-> |read path| QueryFrontend
    nginx -.-> |write path| Distributor
    
    subgraph LokiRead["loki -target=read (3)"]
        QueryFrontend["query-frontend"]
        Querier["querier"]

        QueryFrontend -.-> Querier
    end

    subgraph Minio["Cloud Storage"]
        Chunks
        Indexes
    end

    subgraph LokiWrite["loki -target=write (3)"]
        Distributor["distributor"] -.-> Ingester["ingester"]
        Ingester
    end

    Querier --> |reads| Chunks & Indexes
    Ingester --> |writes| Chunks & Indexes
```

Ref : https://github.com/grafana/loki/blob/main/production/docker/README.md