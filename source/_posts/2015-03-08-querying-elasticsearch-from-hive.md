---
layout: post
title: "Querying Elasticsearch from Hive"
date: 2015-03-08 16:56
comments: true
published: true
categories:
- databases
- hadoop
---

I wanted to query ES from Hive. They seem like really interesting, complementary tools.

## Setup

Obviously I needed Elasticsearch and Hive to be running. The former is
fairly straightforward, the latter requires a basic Hadoop stack.

I have been playing with the Cloudera distribution, and for a cluster
it is great, but for this I don't need the (significant) overhead.

So I found a [Vagrant setup](https://github.com/congdang/vagrant-hadoop-hive) that did a bunch of what I needed, fixed a
bunch of issues, added Elasticsearch, and pushed up
[hadoop-hive-elasticsearch-vm](https://github.com/iantruslove/hadoop-hive-elasticsearch-vm) - a single-node ES and Hive cluster.

## What I'm trying to accomplish

-   ES aggregations are cool, but I have definitely found use cases
    where they don't do all that I need.
-   I'm trying to do things like calculating [TF-IDF](http://en.wikipedia.org/wiki/Tf%25E2%2580%2593idf) type statistics for
    documents and groups of documents.
-   The tricky thing with aggregations was stuff like summing TFs across
    a whole account.
-   These kinds of things are really easy with SQL, so it seems like
    ES + Hive (or later ES + Spark) would be a very powerful
    combination.

<!-- more -->

## Add some data to Elasticsearch

I have a host `hadoop-hive-elasticsearch` running standard
installations of Elasticsearch and Hive, as close to out of the box as
one would reasonably get.

Create an index in Elasticsearch:

```sh
curl -v -XPUT http://hadoop-hive-elasticsearch:9200/idx_foo
```

PUT the mapping:

```sh
    curl -v -XPUT 'http://hadoop-hive-elasticsearch:9200/idx_foo/_mapping/analysis?ignore_conflicts=true' -d '{
      "analysis" : {
        "properties" : {
          "analyserVersion" : {
            "type" : "string",
            "store" : "true",
            "index" : "not_analyzed"
          },
          "analysisTime" : {
            "type" : "date",
            "format" : "date_optional_time"
          },
          "documentIdentifier" : {
            "type" : "string",
            "store" : true,
            "index" : "not_analyzed"
          },
          "topics" : {
            "type" : "string"
          }
        }
      }
    }'
```

Use the bulk API to add some docs:

```sh
    curl -v -XPOST http://hadoop-hive-elasticsearch:9200/_bulk -d '
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:00Z", "documentIdentifier": "doc1", "topics": ["business", "collaboration", "lifehacks", "email"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:01Z", "documentIdentifier": "doc2", "topics": ["sports", "basketball"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:02Z", "documentIdentifier": "doc3", "topics": ["lifehacks", "wardrobe"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:03Z", "documentIdentifier": "doc4", "topics": ["sports", "snowboarding", "mountains"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:04Z", "documentIdentifier": "doc5", "topics": ["outdoors", "hiking", "mountains", "lakes"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:05Z", "documentIdentifier": "doc6", "topics": ["business", "sales", "cars"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:06Z", "documentIdentifier": "doc7", "topics": ["jeep", "cars", "outdoors", "off-roading"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:07Z", "documentIdentifier": "doc8", "topics": ["sports", "fly fishing", "lakes"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:08Z", "documentIdentifier": "doc9", "topics": ["programming", "clojure", "databases"] }
    '
```

So now the ES instance has an index called `idx_foo`, with a handful
of documents belonging to one document type.

## Hive Time<a id="sec-1-4"></a>

Create a database in Hive to use

```sql
    CREATE DATABASE es_test;
    USE es_test;
```

Create a table. It needs to be an external table and pull data in from
Elasticsearch.

```sql
    -- DROP TABLE foo_analysis_doc_ids;

    CREATE EXTERNAL TABLE foo_analysis_doc_ids (
      analyser_version STRING,
      analysis_time TIMESTAMP,
      doc_id STRING,
      topics ARRAY<STRING>
    )
    STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler'
    TBLPROPERTIES('es.resource' = 'idx_foo/analysis',
                  'es.mapping.names' = 'doc_id:documentIdentifier, analysis_time:analysisTime, analyser_version:analyserVersion');
```

It turned out that mapping column names is useful.

Querying ES from Hive:
```sql
    SELECT * FROM foo_analysis_doc_ids;
```

    0.0.1   2015-03-07 16:12:00     be87f0b5-faad-4513-827c-15a635844eaa    ["business","collaboration","lifehacks","email"]
    0.0.1   2015-03-07 16:12:01     doc2    ["sports","basketball"]
    0.0.1   2015-03-07 16:12:05     doc6    ["business","sales","cars"]
    0.0.1   2015-03-07 16:12:06     doc7    ["jeep","cars","outdoors","off-roading"]
    0.0.1   2015-03-07 16:12:00     b0289460-f6ef-4309-911a-d27e52155ae7    ["business","collaboration","lifehacks","email"]
    0.0.1   2015-03-07 16:12:02     doc3    ["lifehacks","wardrobe"]
    0.0.1   2015-03-07 16:12:07     doc8    ["sports","fly fishing","lakes"]
    0.0.1   2015-03-07 16:12:03     doc4    ["sports","snowboarding","mountains"]
    0.0.1   2015-03-07 16:12:08     doc9    ["programming","clojure","databases"]
    0.0.1   2015-03-07 16:12:00     doc1    ["business","collaboration","lifehacks","email"]
    0.0.1   2015-03-07 16:12:04     doc5    ["outdoors","hiking","mountains","lakes"]
    Time taken: 0.038 seconds, Fetched: 11 row(s)

Great. You can even see here there are a couple of documents with UUID
identifiers that I had in ES from experiments along the way that
didn't quite get written up here.

So let's do some SQL-y things with Elasticsearch data: retrieve a list of unique topics.

```sql
    SELECT DISTINCT(topic_list.t) AS topic
    FROM (SELECT explode(f.topics) AS t FROM foo_analysis_doc_ids AS f) AS topic_list
    ORDER BY topic;
```

    Query ID = vagrant_20150308155959_a966862b-4aa0-489d-867f-74b880a952a1
    Total jobs = 2
    Launching Job 1 out of 2
    Number of reduce tasks not specified. Estimated from input data size: 1
    In order to change the average load for a reducer (in bytes):
      set hive.exec.reducers.bytes.per.reducer=<number>
    In order to limit the maximum number of reducers:
      set hive.exec.reducers.max=<number>
    In order to set a constant number of reducers:
      set mapred.reduce.tasks=<number>
    Starting Job = job_201503081508_0007, Tracking URL = http://localhost:50030/jobdetails.jsp?jobid=job_201503081508_0007
    <SNIP>
    Stage-Stage-1: Map: 5  Reduce: 1   Cumulative CPU: 8.57 sec   HDFS Read: 190090 HDFS Write: 617 SUCCESS
    Stage-Stage-2: Map: 1  Reduce: 1   Cumulative CPU: 1.86 sec   HDFS Read: 1113 HDFS Write: 181 SUCCESS
    Total MapReduce CPU Time Spent: 10 seconds 430 msec
    OK
    basketball
    business
    cars
    clojure
    collaboration
    databases
    email
    fly fishing
    hiking
    jeep
    lakes
    lifehacks
    mountains
    off-roading
    outdoors
    programming
    sales
    snowboarding
    sports
    wardrobe
    Time taken: 41.09 seconds, Fetched: 20 row(s)

In doing the previous I ran into a problem that looked like the
elasticsearch-hadoop jar wasn't distributed to the hadoop cluster.

-   Having the JAR in the Hive `lib` dir didn't work.
-   Putting the JAR into the Hadoop `lib` dir didn't work.
-   Using the Hive shell's `ADD JAR <file-path>` command DID WORK!
-   This is something that will need to be addressed systematically for
    a real-world scenario - something like distrbuting the JAR via HDFS.

Note the time taken to do that - 40 seconds. It ended up firing two
map-reduce jobs, one for the `explode`, and one for the `ORDER
BY`.


For local testing, there's a local mode to help:

```sql
    SET mapred.job.tracker=local;
```

The same query then takes 4 seconds.

Now, how about something like "what's the mapping of topics to
documents?" This has a [CTE](https://cwiki.apache.org/confluence/display/Hive/Common%2BTable%2BExpression), a `SELECT DISTINCT`, a `JOIN` - definitely
interesting stuff to layer on top of an Elasticsearch index.

```sql
    WITH topic_explosion AS (SELECT explode(f.topics) AS t FROM foo_analysis_doc_ids AS f),
    topics AS (SELECT DISTINCT(t) AS topic FROM topic_explosion)
    SELECT topics.topic, f.doc_id
    FROM foo_analysis_doc_ids f, topics
    WHERE array_contains(f.topics, topics.topic)
    ORDER BY topic, doc_id;
```

    Total MapReduce CPU Time Spent: 0 msec
    OK
    basketball      doc2
    business        b0289460-f6ef-4309-911a-d27e52155ae7
    business        be87f0b5-faad-4513-827c-15a635844eaa
    business        doc1
    business        doc6
    cars    doc6
    cars    doc7
    clojure doc9
    collaboration   b0289460-f6ef-4309-911a-d27e52155ae7
    collaboration   be87f0b5-faad-4513-827c-15a635844eaa
    collaboration   doc1
    databases       doc9
    email   b0289460-f6ef-4309-911a-d27e52155ae7
    email   be87f0b5-faad-4513-827c-15a635844eaa
    email   doc1
    fly fishing     doc8
    hiking  doc5
    jeep    doc7
    lakes   doc5
    lakes   doc8
    lifehacks       b0289460-f6ef-4309-911a-d27e52155ae7
    lifehacks       be87f0b5-faad-4513-827c-15a635844eaa
    lifehacks       doc1
    lifehacks       doc3
    mountains       doc4
    mountains       doc5
    off-roading     doc7
    outdoors        doc5
    outdoors        doc7
    programming     doc9
    sales   doc6
    snowboarding    doc4
    sports  doc2
    sports  doc4
    sports  doc8
    wardrobe        doc3
    Time taken: 5.945 seconds, Fetched: 36 row(s)

## Looking ahead&#x2026;

I can see some interesting experiments ahead:

-   How can I effectively work across multiple ES indexes? E.g. if
    there's one index per account, and analyses go into that, how can
    aggregate statistics be calculated across accounts?
-   Hive uses Hadoop MR to distribute work. This is slow. Would [Pig](http://pig.apache.org/) do
    any better? How about [Spark](https://spark.apache.org/)? How would the three perform at scale?
