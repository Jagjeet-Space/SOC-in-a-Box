# Docker-compose

To understanding how **Docker** is running Elastic & Kibana we will explain every conf line in docker-compose yml file.

This is full conf snippet of docker-compose 

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=soc-box
      - discovery.type=single-node
      - xpack.security.enabled=false        # Temporarily disable
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - bootstrap.memory_lock=true
    ports:
      - "${ES_PORT}:9200"
    ulimits:
      memlock: -1
    healthcheck:
      test: ["CMD-SHELL", "curl -f -u elastic:${ELASTIC_PASSWORD} http://localhost:9200/_cluster/health || exit 1"]
      interval: 10s
      timeout: 10s
      retries: 5
      start_period: 60s
    deploy:
      resources:
        limits:
          memory: ${ES_MEM_LIMIT}

  kibana:
    image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
    container_name: kb01
    ports:
      - "${KIBANA_PORT}:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_SERVICEACCOUNTTOKEN=AAEAAWVsYXN0aWMva2liYW5hL2tpYmFuYS10b2tlbjp4Q000LWhRR1M1cWtVOEZReUdPdjF3
      - SERVER_HOST=0.0.0.0
      - xpack.security.enabled=false          # Temporarily Disable
      - xpack.encryptedSavedObjects.encryptionKey=abcdefghijklmnopqrstuvwxyz0123456789
    depends_on:
      elasticsearch:
        condition: service_healthy
    deploy:
      resources:
        limits:
          memory: ${KB_MEM_LIMIT}
```


## service: Block 

The root key **services** defines the containers you want to run. In this case:
* Elsatic (es01)
* Kibana (kb01)

### Elastic Service 
```yaml
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    container_name: es01
```
* image > Pulls Elasticsearch image version set by `${STACK-VERSION}`
enviornment variable, e.g `8.14.3`).
* container_name > Names the container es01.

### Enviornment Variables
```yaml
    environment:
      - node.name=es01
      - cluster.name=soc-box
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - bootstrap.memory_lock=true
```
* node.name > Unique name for this Elasticsearch node.
* cluster.name > Logical cluster name (`soc-box`)
* discovery.type=signle-nods > Runs in standalone mode (not a multi-node cluster) this helps not use system resources too much.
* xpack.security.enabled=false > Disables authentication temporarily
* ELASTIC_PASSWORD > Sets password for built-in elastic user (comes from `.env`).
* bootstrap.memory_lock=true > Prevents JVM heap from being swapped to disk (improves performance).

### Ports
  ```yaml
      ports:
      - "${ES_PORT}:9200"
  ```
* Maps Elasticsearch port 9200 to host's ${ES-PORT} (e.g 9200)

### Memory Lock
```yaml
    ulimits:
      memlock: -1
```
* Ensures unlimited memory locking (needed for `bootstrap.memory_lock=true`).

### Healthcheck
```yaml
    healthcheck:
      test: ["CMD-SHELL", "curl -f -u elastic:${ELASTIC_PASSWORD} http://localhost:9200/_cluster/health || exit 1"]
      interval: 10s
      timeout: 10s
      retries: 5
      start_period: 60s
```
* Runs every 10s, tries for 60s startup period
* Uses curl to check cluster health endpoint > service is "healthy" only when ES is running.

### Resource Limits
```yaml
    deploy:
      resources:
        limits:
          memory: ${ES_MEM_LIMIT}
```
* Restricts container memory usage to `${ES_MEM_LIMT}` (eg 1g).




### Kibana Service
```yaml
  kibana:
    image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
    container_name: kb01
```
* Runs Kibana (UI for Elasticsearch).

### Ports
```yaml
    ports:
      - "${KIBANA_PORT}:5601"
```
* Exposes Kibana web interface on `${KIBANA_PORT}` (default *5601*).


### Enviornment Variables
```yaml
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_SERVICEACCOUNTTOKEN=AAEAAW...
      - SERVER_HOST=0.0.0.0
      - xpack.security.enabled=false
      - xpack.encryptedSavedObjects.encryptionKey=abcdefghijklmnopqrstuvwxyz0123456789
```
* ELASTICSEARCH_HOSTS > Tells Kibana where Elasticsearch lives.
* ELASTICSEARCH_SERVICEACCOUNTTOKEN > Service token used for Kibana <> ES communiation.\
* SERVER_HOSTS=0.0.0.0 > Listens on all network interfaces.
* xpack.security.enabled=false → Disables authentication temporarily.
* encryptionKey > Needed for securely storing saved objects (dashboards, visualization).

### Dependencies
```yaml
    depends_on:
      elasticsearch:
        condition: service_healthy
```
* Kibana starts only after Elasticsearch is healthy.

### Resources Limits
```yaml
    deploy:
      resources:
        limits:
          memory: ${KB_MEM_LIMIT}
```
* Restricts memory for Kibana container.



### Flow of Services

* Elasticsearch (es01) starts → provides search & indexing.
* Kibana (kb01) waits for Elasticsearch → provides dashboards & visualizations.


### Notes

* Security is disabled (xpack.security.enabled=false) → Good for learning, not production.
* Memory & credentials (${...}) come from an .env file.
* Healthchecks ensure dependent services only start when ES is fully ready.










