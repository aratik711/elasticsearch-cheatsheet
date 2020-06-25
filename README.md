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
### Search document immediately after insert
```json
curl -X PUT localhost:9200/chapter5/_doc/3?refresh=true -H 'Content-type: application/json' -d \
'{
  "id": 3,
  "name": "User3",
  "age": 55,
  "gender": "M",
  "email": "user3@gmail.com",
  "last_modified_date": "2017-02-15"
}'
```
### refresh index
```
curl -X POST localhost:9200/chapter4/_refresh
```
### refresh all indices
```
curl -X POST localhost:9200/_refresh
```
### change refresh interval
```json
curl -X PUT localhost:9200/chapter4/_settings -H 'Content-type: application/json' -d \
'{
  "index": {
"refresh_interval": "30s"
  }
}'
```
### only update 1 field
```json
curl -X POST localhost:9200/chapter5/_doc/2/_update -H 'Content-type: application/json' -d \
'{
  "doc": {
    "name": "name udpate 2"
  }
}'
```
### inline script
```json
 curl -X POST "localhost:9200/chapter5/_update/2?pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "lang": "painless",
    "source": "ctx._source.last = \"abc\";\nctx._source.nick = \"pqr\"",
    "params": {
      "last": "gaudreau",
      "nick": "hockey"
    }
  }
}'
```
### upsert
```json
curl -X POST localhost:9200/chapter5/_doc/4/_update -H 'Content-Type: application/json' -d'
{
  "doc": {
    "name": "user3"
  },
  "doc_as_upsert": true
}'
```
### always update document
```json
curl -X POST localhost:9200/chapter5/_doc/2/_update -H 'Content-Type: application/json' -d'
{
  "doc" : {
     "name" : "user2 new"
  },
  "detect_noop" : "false"
}'
```
### Update specific version of a document
```json
curl -X PUT 'localhost:9200/chapter5/_doc/2?if_seq_no=9&if_primary_term=1' -H 'Content-Type: application/json' -d'
{
 "id": 2,
 "name": "name update 1",
 "age": "55",
 "gender": "M",
 "email": "user2@gmail.com",
 "last_modified_date" : "2017-02-18"
}'
```
### Retry
```json
curl -X POST localhost:9200/chapter5/_doc/2/_update?retry_on_conflict=3 -H 'Content-Type: application/json' -d'
{
  "doc": {
    "name": "name update 2"
  }
}'
```
Ref: [https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)
### Search local
```json
curl -X POST localhost:9200/chapter5/_doc/_search?preference=_local -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "name": "name update 1"
    }
  }
}'
```
Ref: [https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-request-preference.html](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-request-preference.html)
### Bulk operations
```json
curl -X POST "localhost:9200/_bulk?pretty" -H 'Content-Type: application/json' -d'
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }'
```
### Multiget
```json
curl -X POST localhost:9200/test/_mget -H 'Content-Type: application/json' -d'
{
"ids" : ["2", "3"]
}'

curl -X POST localhost:9200/_mget -H 'Content-Type: application/json' -d'
{
   "docs": [
     {
       "_index": "test",
       "_id": "1",
       "_source": [
       "field1"
       ]
     },
     {
       "_index": "test",
       "_id": "3",
       "_source": {
       "exclude": "field1"
       }
     }
   ]
 }'
```
### Inline update
```json
 curl -X POST localhost:9200/store/_update_by_query -H 'Content-Type: application/json' -d'
 {
   "script": {
"inline": "ctx._source.price_with_tax = ctx._source.unit_price * 1.1",
     "lang": "painless"
   },
   "query": {
     "match_all": {}
   }
 }'
```
### Bulk
```json
curl -X POST localhost:9200/_bulk -H 'Content-Type: application/json' -d'
{"index": {"_index": "store", "_id": 1}}
{"item": "toothpaste", "unit_price": 7.5}
{"create": {"_index": "store", "_id": 2}}
{"item": "soap", "unit_price": 10}'
```
### Inline
```json
curl -X POST localhost:9200/store/_update_by_query?wait_for_completion=false -H 'Content-Type: application/json' -d'
 {
   "script": {
     "inline": "ctx._source.price_with_tax = ctx._source.unit_price * 1.1",
     "lang": "painless"
   },
   "query": {
     "match_all": {}
   }
 }'
```
### Check tasks
curl -X GET localhost:9200/_tasks/xZjyFE19Q0yGxehcys6ydg:307842

### Cancel task
curl -XPOST localhost:9200/_tasks/xZjyFE19Q0yGxehcys6ydg:307842/_cancel
### Delete
```json
curl -X POST localhost:9200/store/_delete_by_query?conflicts=proceed  -H 'Content-Type: application/json' -d'
 {
   "query": {
     "match": {
       "item": "soap"
     }
   }
 }'
```
 ### Reindex
 ```json
curl -X POST localhost:9200/_reindex -H 'Content-Type: application/json' -d'
 {
"source": {
     "index": "store"
   },
"dest": {
     "index": "storenew"
   }
 }'
```
### Combining documents from more than one index
```json
curl -XPOST localhost:9200/_reindex -H 'Content-Type: application/json' -d'
 {
"source": {
     "index": [
       "source_index_1",
       "source_index_2"
     ]
   },
"dest": {
     "index": "dest_index"
   }
 }'
```
### Copy only missing document to new index
```json
curl -XPOST localhost:9200/_reindex -H 'Content-Type: application/json' -d'
 {
"conflicts": "proceed",
   "source": {
     "index": "source_index"
   },
   "dest": {
     "index": "dest_index",
	 "op_type": "create"
   }
 }'
```
### Copy subset of documents to new index
```json
curl -XPOST localhost:9200/_reindex -H 'Content-Type: application/json' -d'
 {
   "source": {
     "index": "source_index_1",
     "type": "log",
"query": {
       "term": {
         "level": "ERROR"
       }
     }
   },
   "dest": {
     "index": "dest_index"
   }
 }'
```
### Copy top n docs to new index
```json
curl -XPOST localhost:9200/_reindex -H 'Content-Type: application/json' -d'
 {
"size": 1000,
   "source": {
     "index": "source_index",
"sort": {
       "timestamp": "desc"
     }
   },
   "dest": {
     "index": "dest_index"
   }
 }'
```
### Copying subset of fields to new index
```json
curl -XPOST localhost:9200/_reindex -H 'Content-Type: application/json' -d'
 {
   "source": {
     "index": [
       "source_index_1"
     ],
"_source": [
       "field1",
       "field2"
     ]
   },
   "dest": {
     "index": "dest_index"
   }
 }
```
### Simulate pipeline
```json
curl -X POST localhost:9200/_ingest/pipeline/_simulate?pretty -H 'Content-Type: application/json' -d'
 {
   "pipeline": {
     "description": "Item View Pipeline",
     "processors": [
       {
         "grok": {
           "field": "log",
           "patterns": [
             "%{IP:client} %{TIMESTAMP_ISO8601:timestamp} %{WORD:country} %{NUMBER:itemId} %{WORD:platform}"
           ]
         }
       },
       {
         "remove": {
           "field": "log"
         }
       },
       {
         "date_index_name": {
           "field": "timestamp",
           "index_name_prefix": "viewitem-",
           "date_rounding": "M",
           "index_name_format": "yyyy-MM",
           "date_formats" : [
          "ISO8601"
          ]
         }
       }
     ]
   },
   "docs": [
     {
       "_source": {
         "log": "127.0.0.1 2017-04-11T09:02:34.234+07:00 USA 1 Web"
       }
     }
   ]
 }'
```
### Ingest pipeline
```json
curl -X PUT localhost:9200/_ingest/pipeline/view_item_pipeline?pretty -H 'Content-Type: application/json' -d'
 {
   "description": "Item View Pipeline",
   "processors": [
     {
       "grok": {
         "field": "log",
         "patterns": [
           "%{IP:client} %{TIMESTAMP_ISO8601:timestamp} %{WORD:country} %{NUMBER:itemId} %{WORD:platform}"
         ]
       }
     },
     {
       "remove": {
         "field": "log"
       }
     },
     {
       "date_index_name": {
         "field": "timestamp",
         "index_name_prefix": "viewitem_",
         "date_rounding": "M",
         "index_name_format": "yyyy_MM",
         "date_formats" : [
        "ISO8601"
        ]
       }
     }
   ]
 }'
```
### Test ingest pipeline
```json
curl -X POST localhost:9200/chapter6/log/?pipeline=view_item_pipeline -H 'Content-Type: application/json' -d'
  {
    "log": "127.0.0.1 2017-04-11T09:02:34.234+07:00 USA 1 Web "
  }'

curl -X  POST localhost:9200/chapter6/log/?pipeline=view_item_pipeline -H 'Content-Type: application/json' -d'
  {
    "log": "127.0.0.1 2017-04-12T09:02:34.234+07:00 USA 2 Web "
  }'


curl -X  POST localhost:9200/chapter6/log/?pipeline=view_item_pipeline -H 'Content-Type: application/json' -d'
  {
    "log": "127.0.0.1 2017-04-13T09:02:34.234+07:00 USA 3 Web "
  }'


curl -X  POST localhost:9200/chapter6/log/?pipeline=view_item_pipeline -H 'Content-Type: application/json' -d'
  {
    "log": "127.0.0.1 2017-04-11T09:02:34.234+07:00 USA 1 Web "
  }'
```
