# Elasticsearch 7 Cheatsheet
### Node Details
```
curl "localhost:9200/_cat/nodes?v&h=id,name,jdk,version,build,node.role,ram.*,heap.*"
```
### Healthcheck
```
curl localhost:9200/_cluster/health
```
### Master
```
curl localhost:9200/_cat/master
```
### Cluster settings
```
curl localhost:9200/_cluster/setting
```
### Node stats
```
curl localhost:9200/_nodes/stats
```
### Index stats
```
curl localhost:9200/_nodes/stats/indices
```
### Index stats with column headers
```
curl 'localhost:9200/_cat/indices?v'
```
### Shard details
```
curl "localhost:9200/_cat/shards"
```
### Details on unallocated shards
```
curl "localhost:9200/_cluster/allocation/explain"
```
### Pending tasks
```
curl -s 'localhost:9200/_cluster/pending_tasks'
```
### Threadpool details
```
curl "localhost:9200/_cat/thread_pool/search?v&h=node_name,name,active,rejected,completed"
```
### cluster health: index level
```
curl -XGET 'http://elasticsearch:9200/_cluster/health?level=indices&pretty'
```
### cluster health: shard level
```
curl -XGET 'http://elasticsearch:9200/_cluster/health?level=shards&pretty'
```
### Cache clear (In case you hit circuit breaker)
```
circuit breaker
curl -XPOST 'http://localhost:9200/<Index Name Here>/_cache/clear'
```
### Changing fielddata circuit breaker
```json
curl -XPUT 'http://localhost:9200/_cluster/settings' -d '
{
    "persistent" : {
       "indices.breaker.fielddata.limit" : "70%"
    }
}'
```
### Disk allocation unassigned
```
curl -XGET http://localhost:9200/_cluster/allocation/explain?include_disk_info=true&pretty=true
```
### Retry unassigned shards
```
curl -XPOST http://localhost:9200/_cluster/reroute?retry_failed
```
### Delay allocation
```json
curl -XPUT "localhost:9200/<INDEX_NAME>/_settings?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}'
``` 
Ref: [https://www.elastic.co/guide/en/elasticsearch/reference/current/delayed-allocation.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/delayed-allocation.html)
### Disable re-enable shard allocation
```json
curl -XPUT 'localhost:9200/_settings' -d '{
    "index.routing.allocation.disable_allocation": false
}'
```

#### v1.0+
```json
curl -XPUT 'localhost:9200/_cluster/settings' -d '{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
}'
```
### Change number of replicas
```json
curl -XPUT "localhost:9200/<INDEX_NAME>/_settings?pretty" -H 'Content-Type: application/json' -d' { "number_of_replicas": 2 }'
```
### Manually reassign unassigned shards
```json
curl -X POST "localhost:9200/_cluster/reroute?pretty" -H 'Content-Type: application/json' -d'
{
    "commands" : [
        {
            "move" : {
                "index" : "test", "shard" : 0,
                "from_node" : "node1", "to_node" : "node2"
            }
        },
        {
          "allocate_replica" : {
                "index" : "test", "shard" : 1,
                "node" : "node3"
          }
        }
    ]
}'
```
Ref: [https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-reroute.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-reroute.html)

### Delete unassigned shards
```
curl -XGET http://localhost:9200/_cat/shards | grep UNASSIGNED | awk {'print $1'} | xargs -i curl -XDELETE "http://localhost:9200/{}"
```
### List of indices
```
curl 'localhost:9200/_cat/indices?v'
```
### Disk allocation
```
curl -s 'localhost:9200/_cat/allocation?v'
```
### Increase disk lwm
```json
curl -XPUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low": "90%"
  }
}'
```
### Get total hit count
```
curl -s 'localhost:9200/_search' 
```
### List of all indices with its details
```
curl -s 'localhost:9200/_cluster/state' 
```
### Pending tasks
```
curl -s 'localhost:9200/_cluster/pending_tasks'
```
### Synced Flush (Before 7.6)
```
curl -XPOST 'localhost:9200/_flush/synced'
```
#### After 7.7
```
curl -XPOST 'localhost:9200/_flush
```
Ref: [https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-synced-flush-api.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-synced-flush-api.html)

### View one index
```
curl -XGET 'http://127.0.0.1:9200/_cat/indices/<index name>?v'
```
### Document count across all indices
```
curl -XGET http://elasticsearch:9200/_cat/count?v
```
### Shard info per index
```
curl -XGET http://elasticsearch:9200/_cat/shards/<index name>?v
```
### Recovery
Info: Returns information about ongoing and completed shard recoveries,
```
curl -XGET http://127.0.0.1:9200/_cat/recovery?v
```
### Index setting
```
curl -XGET 'http://127.0.0.1:9200/<Index name>/_settings?pretty'
```
### Number of primary shards per node
```
curl -s -XGET 'http://elasticsearch:9200/_cat/shards/<index name>?v' | grep 'p      STARTED' | awk '{print $7}' | sort | uniq -c
```
### Close index
```
curl -XPOST http://elasticsearch:9200/<Index name>/_close
```

### Open index
```
curl -XPOST http://elasticsearch:9200/<Index name>/_open
```

### Document
```
curl -XGET http://elasticsearch:9200/<Index name>/<id>?pretty
```
### Per index stats
```
curl "localhost:9200/<Index name>/_stats"
```
### Check memory swapiness
```
curl `hostname -i`:9200/_nodes?filter_path=**.mlockall
```
### Create document
```json
curl -X PUT -H "Content-Type: application/json" `hostname -i`:9200/chapter1/product/1 -d \
'{
  "title": "Learning Elasticsearch",
  "author": "Abhishek Andhavarapu",
  "category": "books"
}'
```
### Create auto identifier for document
```json
curl -X POST -H "Content-Type: application/json" `hostname -i`:9200/chapter1/product -d \
'{
  "title": "Learning Elasticsearch",
  "author": "Abhishek Andhavarapu",
  "category": "books"
}'
```
### Read document
```
curl -X GET `hostname -i`:9200/chapter1/product/1
```
