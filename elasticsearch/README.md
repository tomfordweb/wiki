Code Examples [here](https://github.com/codingexplained/complete-guide-to-elasticsearch)

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


# How ES handles dates.

### Default behavior
A date without time, a date with time, or milliseconds since the epoch.

UTC timezone by default, dates must be formatted according to ISO 8601.


### How dates are stored

Stored internally as milliseconds since the epoch. Any value that you supply is converted to a long value internally.

Dates are converted to UTC, this applies to indexes as well as search queries.


*NOTE* You should not provide unix timestamps to elasticsearch.

# Missing Fields

all fields are optional by default by ES. You can leave out a field when indexing documents. Because of this, some integrity checks are needed at the application level.

Adding a field mapping does not make the field required. ES will automatically handle missing fields.

# Mapping parameters

### format
Used to customixe th format for date fields.
Use strict format whenever possible.

```
strict_date_optional_time || epoch_millis
```

or `dd/MM/yyyy` The default Java format for dates.

### propeties

defines nested fields for `object` and `nested` types.

 
### coerce

lets you enable or disable type coercion of values (enabled by default) of a document. 

It is also possible to specify coercion at the index leve. Since this is enabled by default it is best to disable coercion at the index leve.

### doc values


ES makes use of several different data strutures. No single data structure serves all purposes. 

Inverted indicies for example are great at searcing text, but they don't perform well for many other data access patterns.

Doc values is optimized for a different data access pattern `document -> terms`

Dov values are an uninverted inverted index, so they are used for sorting, aggregations, and scripting.

You ca set this to false to save disk space, however it is insignificant at smaller scale.

### norms

Refers to normalization factors for relevance scoring. Oftewe don't just want to filter results, but also rank them.

norms can be disabled to save disk space on fields that wont be used for relevance sorting. Keep in mind these fields can still be used for filtering and aggregations.

Things like categories or tags do not need norms indexes because they will most likely not be used in full text search.

### index

You can disable indexing for a field. This is useful if you won't use a field for search queries but are storing it as aggregated data for example. Another example is time series data.

Disabling indexes will save disk space and slightly improve indexing throughput.


###  null_value

NULL values cannot be indexed or searhched. Use this parameter to replace null values wit another value.

This only works with explicit NULL values. Also, the replacement field must be the same data type.


### copy_to

Used to copy multiple field values into a group field. You just need to specify the nam of the target field as the value.

Keep in mind that values are copied, not terms/tokens. The analyzer for the target fiel is used for the values.

The target fiel is not a part of `_source`

# Updating existing mappings

You cannot. The exception is the `ignore_above` which ignores a string by character length.

Reindex the documents and get it right this time.

# Reindexing.


```
PUT /reviews/_new
{
  ...paste exising output from
  GET /reviews/_mappings
  ... update whatever you want.
}
```
```
POST /_reindex
{ 
  "source":{
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  }
}
```
You can also add query logic.

# Field aliases
Field names can be changed when reindexing documents. Probably not worth it for large indexes with lots of documents.

The alternative is to use field aliases. Which don't require the docuent to be reindexed. Aliases can aslo be used within queries. Aliases are defined with a field mapping.

Field aliases can be updated, you can simply perform an update with a new path value.

Field aliases can also be set at the cluster level. Which will help when working with large amounts of data.

# Index Templates
Index templates specify settings and mappings. They are applied to indicies that match one or more patterns. These patterns include wildcards. Index templates take effect when creating new indices.

A new index may match multiple templates.

You can update a template, by providing the existing and upate data, however it will only apply to new indexes.

# elastic common schema

This is a spec for common fields and how they should be mapped. Before this filebeats would send different formats for apache vs nginx logs.

@timestamp for example is the standard name for a timstamp when an event originated.

In ECS documents are referred ot as events, ECS doesnt provide fields for non events (productS).

This is mostly used for standrd events - web server logs operating system metrics, etc.


# dynamic mapping

ES will automatically map scalars/data stryctyres to what it thinks the value is

```
string -> text/date/float/long
integet -> long
floating point number -> float
bool -> bool
object -> object
array -> depends on first null value
```

## combining explicit and dynamic mapping

To disable dynamic mapping, set `{"mappings":"dynamic":false}}`

This instructs ES to ignore new fields, the data is returned in the `_source` but it is not indexed. This means that new fields must be mapped explicitly when adding them.

You can also set `dynamic` to `strict` to make it so that any new documents added will error if the field is not mapped.

# Dynamic Templates
A dynamic template contains one or more conditions, as well as the mappings that apply if the document meets the conditions

The below code will make it so that any document in this index will use int instead of long for any integer that is mapped.

```
 PUT /dyno_template_test
 {
   "mappings": {
     "dynamic_templates": [
        {
          "integers": {
            "match_mapping_type": "long",
            "mapping":{
              "type": "integer"
            }
          }
        }
   }
 }
 ```

 The below template will map the `keyword` type to any data property ending in keyword, and any string beginning with `text_` is applied the text mapping.
 An approach like this allows mapping based on naming conventions on property names.

 ```ndjson
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_only_text": {
          "match_mapping_type": "string",
          "match": "text_*",
          "unmatch": "*_keyword",
          "mapping": {
            "type": "text"
          }
        }
      },
      {
        "strings_only_keyword": {
          "match_mapping_type": "string",
          "match": "*_keyword",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}

POST /test_index/_doc
{
  "text_product_description": "A description.",
  "text_product_id_keyword": "ABC-123"
}
```


This also works for nested obects

```
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "copy_to_full_name": {
          "match_mapping_type": "string",
          "path_match": "employer.name.*",
          "mapping": {
            "type": "text",
            "copy_to": "full_name"
          }
        }
      }
    ]
  }
}

POST /test_index/_doc
{
  "employer": {
    "name": {
      "first_name": "John",
      "middle_name": "Edward",
      "last_name": "Doe"
    }
  }
}
```


### Index tenplates vs dynamic templates
Index templates apply mappings and index settings for matching indexes - This happens when indexes are created and their names match a pattern.

Dynamic templates are evaluated when new fields are encountered (and dynamic mapping is enabled)

The specified field mapping is added if the templates conditions match.

*Index mappnigs define fixed mappings, dynamic templates are dynamic*


# Mapping Best Practices
* Dynamic mappings are convenient, but really shouldn't be used in production, fix your mappings.
* Save disk space by optimizing mappings when storing many documents.
* Set `dynamic` to strict, not `false`
  - Avoids surprises and unexpected results
* Do not always map text fields as both text and keyword. Keep in mind that each mappng requires disk spacee.
  - Do you need to perform full text searches? Use the `text` mapping.
  - Do you need to do aggregations, sorting, or filtering on an exact value? use `keyword` mapping.
* Disable type coercion
* Use appropriate numeric data types.
  - Whole numbers - integer data type may be enough.
  - The `long` can store larger numbers, but takes up more disk space.
  - For decimals, the `float` is precise enough.
  - The `double` is higher precision than float, but takes up twice as much space
* If you are not using sorting, filtering, or aggregating, set `doc_values` to false.
* If you don't need relevance scoring, set `norms` to false
* Set `index` to fakse if you dont need to filter on values
  - keep in mind you can still do aggregations on this

# Data Analyzers

## Stemming and stop words
ES is smart enough to match the tense of a word `love/loved/loves/loving`

Stemming is the process of reducing words to their root form. Note that this is internal only, clients do not see the stems.

For example:

`I loved drinking bottles of wine on last year's vacation` -> `I love drink bottl of wine on lasy year vacat.`

## Stop words.

Common filler words: `a, the, at, on, of`

They provide little to no value for relevance scoring.

It is fairly common to remove these in search engines, however it is not that normal in ES anymore as the relevance algorithm has been improved over time.

Really, you don't need to remove these.

## Built in analyzer.

### standard
splits tect at word boundaries and removes punctuation, lowercases, and also contains the stop filter (disabled by default)

### simple
splits input tect when encountering anything other than letters, it also lowercases letters.

This is not used much and is a performance hack.


## whitepace 
Splits strings by whitespace, doees not lowercase, does not remove punctiation or stop words.

### keyword
no-op analyxer that leaves the input text intact and outputs it as a single token.

This is used by keyword fields by default. And ecact matching.

### pattern

use regex to match token seperators, this is very flexible.

### language specific analyxer

You can get an analyzer for most popular languages. This will remove spanish, or russian stop words and has language specific stemmers.


## Custom analyzers

Below are a few examples of custom analysis.
```
POST /_analyze
{
  "char_filter": ["html_strip"],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```
Now you can add to it, by removing english stop words for example
```
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "stop"
  ],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

You can also create your own analzer
```
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}

```
Then use your custom analyzer
```
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}

```
