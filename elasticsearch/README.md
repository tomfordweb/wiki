# Introduction to elasticsearch

Elasticsearch is an analytics and full text search engine. It allows you to index and search large amounts of data.

ES is great for querying structured data, logging, analyxing, and then alerting on those logs.

APM = Application Performance Management - the primary use of elasticsearch

ES is great for anomaly detection, tracking visitors on website, and alerting when necessary

ES is built with Java, and uses Apache Lucene

# Node Roles

### Master eligible 

* The node may be elected as the clusters master node. A master node is responsible for creating and deleting incicides.
* A node with this role will not automatically become the master node.
* Useful for large clusters.

```
node.master: true|false
```

### Data node

* Enables a node to store data
* Storing data includes performing querie related to that data such as search queries.
* For relatively small clusters this role is almost always enabled.
* Useful for having dedicated master nodes.
* Used as part of configureing a dedicated master node.

### Ingest
* An ingest pipieline is a series of steps that are performed when indexing documents.
* Processors may manipulate documents such as resolving an ip address to a latiude and longitude.
* This is basically a simplified version of logstash

### Machine Learning
```
# lets the node run ML jobs
node.ml: true
# enables or disables the ML api for the node
node.data: true|false
```

### Coordination node

Coordination refers to the distributio of queries and the aggregation of results

These nodes do not search for data, only delegate

```
node.master: false
node.data: false
node.ingest: false
node.ml: false
xpack.ml.enabled: false
```
### Voting Only
A node with this role will participate in the voting for a new master node. I will probably never use this.


# Documents

Data in ES is called a document. It is essentially just a json object.

Documents are queried via a REST API.

Each document has an index.

Documents are grouped together by their indicies.

### Basic document commands

```json
# Create an index
PUT /products
{
  "settings": {
    "number_of_replicas": 2,
    "number_of_shards": 2
  }
}


# Delete an index
DELETE /products


# Create a product document
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 10,
 "in_stock":5 
}

# Create a document with an id of 100
POST /products/_doc/100
{
  "name": "Toaster",
  "price": 50,
 "in_stock":15 
}

# Get the product with an id of 100
GET /products/_doc/100

# You can update a single value
POST /products/_update/100
{
  "doc": { "price": 49.99}
}

# Or add new values
POST /products/_update/100
{
  "doc": { 
    "tags": [
      "foo",
      "bar",
      "baz"
    ]
    
  }
}
```

How an update command works.

Documents are immutable. The current document is retreived. The field values are changed, and the the existing document is replaces.

### Scripted Updates

# The Elastic Stack

The elasticstack consists of products made by ElasticSearch BV (company)

## Kibana

Kibana is a data visualization tool. Consider it a ES dashboard. Kibana is the interface for ES.

## Logstash

Logstash is used to process logs from applications and send them to ES. However, logstash is now used as a data processing pipeline. It is similar to RabbitMQ with how it forwards messages.

A logstash pipeline consists of 3 parts, input, filters, and output.

Input plugins determinme where the data comes from.


Kibana is a data visualization tool. Consider it a ES dashboard. Kibana is the interface for ES.

## Logstash

Logstash is used to process logs from applications and send them to ES. However, logstash is now used as a data processing pipeline. It is similar to RabbitMQ with how it forwards messages.

A logstash pipeline consists of 3 parts, input, filters, and output.

Input plugins determinme where the data comes from.

Filters convert the data into csv, xml, json, etc.

Output sends the filtered data to their recipients.

In a perfect world, all ES events would go through logstash.

## x-pack

Adds features to to Kibana and ES.

Authentication: ACL, LDAP, permissions.

Performance monitoring for Kibana and ES allowing you to setup custom alets.

Alerting: Contacting Developers when shit hits the fan.

Reporting: Allows you to generate scheduled reports.

ML: X pack lets you integrate machine learning to determinme things like anomaly detection such as when users drop, forecasting finances, etc.

Graph: More ML, imagine things like recommended music and videos. It has to go by relevance vs the popularity of the item. Relevance is made up by the uncommonly common recurring events.

ES SQL: a relational database using a query language called DSL. this is very verbose.

## Beats

Beats are lightweight agents that send data to ES

Filebeat for example sends beats for logfiles.

Metricbeat collects system level metrics

Packetbeat collects packet data

Auditbeat collects audit data from linux.

Ingesting features into ES requires either passing through a beat, or the ES api.



# The Elasticsearch directory structure

If you ever see a variable like `$ES_HOME` it references the root elasticsearc directory.

## /bin

this is the location of several helper utilities, install plugins and SQL queries

## config

This is where configuration files are stored.
.
## config/elasticsearch.yml

The main ES configuration. It is best practice to have the cluster.name defined. In my case this is configuration passed to it via docker environment arguments.

## config/jvm.options

This is where you configure the Java engine.


## config/log4j2.properties

Logging configuration for the Java logging program log4j2

## lib

contains libs for ES

## modules

contains all of the ES modules, this is where xpack features are located.

# Understanding the basic architecture

When we start up ES, we actually create a node. You can run as many nodes as you want. Each node is responsible for some data. 

The nodes are grouped in a cluster, where the data is shared between all of the nodes.

It is possible to have cross cluster searches, but is not common.

Typically a cluster would contain different configurations, domain logic or data.


# Basic health comma ds

`/_cluster/health?v`

Returns data about the current cluster.

## `/_cat/nodes?v`

Return status of the clusters nodes.

## `/_cat/indicies?v`

Returns current document indexes in the ES instance.


# Sharding

Sharding is a way to divide indices into smaller pieces.

Each peice of the index is referred to as a shard.

The main reason to divide an index into a shared is for horizontal scalability.

Sharding can allow parallel queries across nodes.

A common use case for sharding is that yYou shard to store more documents. You can store billions this way.

An index contains a single shard by default.

If you think there will be millions of documents, use 5 shards, otherwise start at 1


# Understanding Replication


Replicatio is supported natively and enabled by default...but how does it work?
The number of replicas can be configured at index creation.

The data for a replica shard is never stored on the actual node. So node A will hold the primary shard for A and all of Node B's replicas. Node B will have the primary shard for B, but will also 

You should replicate data once if data loss is not a disaster. For critical systems, data should be replicated at least twice.


Elastic searc allows snapshots as backups. Snapshots can be used to restore to a given point in time.


A replica shard can serve many requests simultaneously, in turn increasing the amount of requests that can be handled at the same time.

CPU parallelization improves performance if multiple replica shards are stored on the same node.


A `Replication Group` is a collection of a primary shards' replicas and the primary shard itself.

There are 2 shards added for an index by default, a primary shard and a replica shard.

# Scaling Up (adding nodes - dev only)

1) First, download and install elastic search.
2) Configure the cluster
  * The node name must be unique across the cluster.
3) Add the node
4) `/bin/elasticSearch`

# Routing
Elastic search uses routing to store documents, and find them once they have been sharded.

# Analysis

Sometimes referred to as Text Analysis, this is how ES indexes data.

Text values are analyzed when indecxing documents. These results are stored in data structures that are efficient for searching.

ES comes with many built in character filters, tokenizers, and token filters. You also have the ability to create custom analysis tools.
### Character filters

A character filter adds, removes or changes characters. An analyzer will have one or more of these. Chacter filters are applied in the order they were specified.

### tokenizer

An analyxer cvontains one tokenixer. Tokenizers split a string by its spaces for example, and characters may be stripped.
```
Input: I realy like beer
Output: ["I","really","like","beer"]
```


### Token filters
Receive a the output of the tokenizxer and add, remove or modify tokens. An anlyzer will contain zero or more token filters.

There is a lowercase filter for example.


## Inverted Indices

A fields values are stored in one of several data structures. This data structure depends on the fields data type. This part is handled by apache lucene, not ES.

An inverted index is the mapping between terms and which documents contain them. Outside of the context of analyxers, we mainly use the word term to describe this data.

This will essentially create a list of every occurence a single character, word or leter has across many documents in an index.

The inverted index exists at the documents text field level. These are only used for text, not integers.

# Mapping

There are types of mapping: Explicit and dynamic. This is similar to the schema for a relational database.

##### Explicit mapping

We define it ourselves

##### Dynamic mapping

ES generates the mappings for us.

## Type coercion

*This is a crutch for bad programmers and should be disabled.*

Data types are inspected when indexing documents. They are validated and some values are rejected. (trying to index an object as a text field for example)

Lucene will attempt to index the value as its type. For example `{"price": "7.95"}` will get indexed as `{"price": 7.95}`, however due to the `_source` field, the data will still be retuned as a string. Lucene stores this differently typed value in BKD trees or inverted indexes to assist with seaching.

You can avoid this behavior by disabling type coercion, or indexing the document with the correct type. 

Note that supplying a floating point for a field typed as as integer will convert it to an integer.

Coercion is not used for dynamic mapping, for example if you supply `"7.4"` for a new field it will create a text mapping.

## Data Types

### object

used for any jso object, you can nest these. An object is stored with its props inside a properties object like so.

```
{
  "name": "tom"
}
# Gets saved as
{
  "mapings": {
    "properties": {
      "name": "tom"
    }
  }
}
```
The allows subfield types to be mapped to their own types. As far as you are concerned..lucene does not store objects as objects. This makes it so data does not have to adhere to any format.

Lucene will store nested object keys as dot notations!

### nested


Similar to the object data type, but maintains their relationships. This is useful when indexing an array of objects. Nested objects are stored as hidden documents.

To query nested objects, you must use the nested query.


### keyword

Used for exact matching of values. `keyword`s are typically used for filtering, aggregations, and sorting. For example, searching for articles with a status of published. This should not be confused with `text` which is used for full text searches.

keyword fields are analyzed with the keyword filter, which is a no-op analyzer - it outputs the unmodified string as a single token. Essentially a token filter does not get applied, so the field does not get split into its indiviudal parts.


### arrays

Technically there is no such thing as an array data type, however ES allows us to store them - how?

For simple text arrays, the strings are just concatenated. 

```
POST /_analyze
{
  "text": ["foo","bar"],
  "analyzer": "standard"
}
# returns
{
  "tokens" : [
    {
      "token" : "foo",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "bar",
      "start_offset" : 4,
      "end_offset" : 8,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}
```

Keep in mind that array data should all return the same format of data.

ES will also flatten nested arrays.

To index an array of objects, you must use the `nested` data type.


# Mappings

You can define a mapping when creating a new index.
```
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": {"type":"float"},
      "content": {"type":"text"},
      "product_id": {"type":"integer"},
      "author": {
        "properties": {
          "first_name": {"type":"text"},
          "last_name": {"type": "text"},
          "email": {"type":"keyword}
        }
      }
    }
  }
}
```
