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
### Update document
```json
curl -X POST -H "Content-Type: application/json" `hostname -i`:9200/chapter1/product/1/_update -d \
'{
"doc": {
    "category": "technical books"
  }
}'
```
### Delete document
```
curl -X DELETE localhost:9200/chapter1/product/1
```
### Create document example 2
```json
curl -X PUT -H "Content-Type: application/json" http://localhost:9200/chapter2/user/1 -d \
'{
   "name": "Luke",
   "age": "100",
   "gender": "F",
   "email": "luke@gmail.com"
}'
```
### Search document created above
```
curl -X GET http://127.0.0.1:9200/chapter2/user/_search?q=name:luke
```
### Search example 2
```json
curl -X POST -H "Content-Type: application/json" http://127.0.0.1:9200/chapter2/user/_search -d \
'{
   "query": {
     "term": {
       "name": "luke"
     }
   }
 }'
 ```
 ### Search all documents
 ```json
curl -XGET "http://35.192.23.155:9200/chapter2/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match_all": {}  }}'
```
### Create dynamic mapping
```json
curl -X PUT localhost:9200/chapter3/person/1 -H "Content-type: application/json" -d \
'{
    "name": "john",
    "age": 200,
    "date_of_birth": "1970/01/01"
}'
```
### Search above document
```json
curl -X GET localhost:9200/chapter3/person/1
curl -X GET localhost:9200/chapter3/person/_search?q=name:john
curl -X POST localhost:9200/chapter3/person/_search -H "Content-type: application/json" -d \
'{
    "query": {
      "term": {
        "name": "john"
      }
    }
}'
```
### Get mapping of an index
```
curl -X GET localhost:9200/chapter3/_mapping?pretty
```
### Delete index
```
curl -X DELETE localhost:9200/chapter3
```
### Create mapping
```json
 curl -X PUT "localhost:9200/chapter3" -H 'Content-Type: application/json' -d'
 {
   "mappings": {
     "properties": {
       "age":    { "type": "integer" },
       "email":  { "type": "keyword"  },
       "name":   { "type": "text"  },
       "gender": { "type": "keyword"  },
       "id": { "type": "integer" },
       "last_modified_date": { "type": "date", "format": "yyyy-MM-dd" }
     }
   }
 }'
 ```
 ### Insert data
 ```json
curl -X POST localhost:9200/chapter3/_doc/1 -H "Content-type: application/json" -d \
'{
    "name": "Abc",
    "age": 100,
    "email": "abc@xyz.com",
    "gender": "F",
    "id": 1,
    "last_modified_date": "2010-01-01"
}'
```
## Add field
```json
 curl -X PUT "localhost:9200/chapter3/_mapping?pretty" -H 'Content-Type: application/json' -d'
 {
   "properties": {
     "address": {
       "type": "text",
       "index": true
     }
   }
 }'
```
 ## Insert data
 ```json
 curl -X POST localhost:9200/chapter3/_doc/2 -H "Content-type: application/json" -d '{
    "name": "Abc",
    "age": 100,
    "email": "abc@xyz.com",
    "gender": "F",
    "id": 1,
    "last_modified_date": "2010-01-01",
    "address": "Thergaon, Pune"
}'
```
### View mapping for field
```
curl -X GET "localhost:9200/chapter3/_mapping/field/id?pretty"
```
### search _all fields
```
curl -X GET localhost:9200/chapter3/_doc/_search?q=Abc
```
### disable _all for search
```json
curl -X PUT localhost:9200/chapter3/_mapping -H "Content-type: application/json" -d \
 '{
   "_all": {
"enabled": false
   },
   "properties": {
     "buyer": {
       "type": "keyword"
     },
     "seller": {
       "type": "keyword"
     },
     "item_title": {
       "type": "text"
     }
   }
 }'
 ```
 ### Disable field in _all search
 ```json
 curl -X PUT localhost:9200/chapter3/_mapping -H "Content-type: application/json" -d \
 '{
   "properties": {
     "buyer": {
       "type": "keyword",
		"include_in_all": false
     },
     "seller": {
       "type": "keyword",
       "include_in_all": false
     },
     "item_title": {
       "type": "text"
     }
   }
 }'
 ```
 ### Analyze
 ```json
 curl -X PUT localhost:9200/weather/_doc/1 -H "Content-type: application/json" -d \
 '{
   "date": "2017/02/01",
   "desc": "It will be raining in yosemite this weekend"
 }'

 curl -X GET "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'
 {
   "analyzer" : "standard",
   "text" : "It will be raining in yosemite this weekend"
 }
 '
 curl -X GET "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'
 {
   "explain" : true,
   "analyzer" : "standard",
   "text" : ["raining", "in", "yosemite"]
 }
 '

 curl -X GET "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'
 {
   "explain" : true,
   "analyzer" : "standard",
   "text" : ["rain"]
 }'
 ```
### create empty index
```
curl -X PUT "localhost:9200/chapter4" -H 'Content-Type: application/json' -d'{ "settings" : { "index" : { } }}'
```
### Routing
```json
curl -X PUT localhost:9200/chapter4/_doc/1?routing=user1 -H "Content-type: application/json" -d \
 '{
   "order_id": "23y86",
   "user_id": "user1",
   "order_total": 15
 }'

 curl -X GET localhost:9200/chapter4/1?routing=user1
```
Ref: [https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-routing-field.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-routing-field.html)

### op_type (operation)
```json
curl -X PUT localhost:9200/chapter4/_doc/1?op_type=create -H 'Content-type: application/json' -d \
'{
  "id": 1,
  "name": "user1",
  "age": "55",
  "gender": "M",
  "email": "user1@gmail.com",
  "last_modified_date": "2017-02-15"
}'
```
Ref: [https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)
