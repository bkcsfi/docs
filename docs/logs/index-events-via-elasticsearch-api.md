title: Index Events via Elasticsearch API
description: Sending, custom & default mapping, and indexing log events using Elasticsearch API. Use other applications such as Logstash, Apache Flume, Fluentd Plugin to send log events and search for logs using UI.

## The Essentials

Sematext's Logs Management App exposes the [Elasticsearch API](http://www.elasticsearch.org/guide/reference/api/) so you can:

  - send log events through it directly from your application, using
    any [Elasticsearch library](http://www.elasticsearch.org/guide/en/elasticsearch/client/community/current/clients.html)
  - send log events by using existing application such as
    [Logstash](logstash), Filebeat, [Logagent](/docs/logagent), or [Apache Flume](http://flume.apache.org/), or [Fluentd Elasticsearch plugin](https://github.com/uken/fluent-plugin-elasticsearch), or anything that can output to Elasticsearch. You can also implement your own "log shipper".
  - [search for logs from your own application](search-through-the-elasticsearch-api), or by using tools such as [Kibana](faq/#can-i-run-kibana-4-locally-and-point-it-to-logsene) or Grafana
  - optionally define [custom mappings](http://www.elasticsearch.org/guide/reference/mapping/) for
    your log types, so you can tweak the way your logs are indexed

When you use the API, here are the things you need to know:

  - host name: **logsene-receiver.sematext.com** / **logsene-receiver.eu.sematext.com** (only if using Sematext Cloud Europe)
  - port: **80** or **443** (depending on whether you want to use plain HTTP or HTTPS)**
    **
  - index name: your [Logs App token](https://apps.sematext.com/ui/logs) -
    note: **this token should be kept secret** (n.b. you can have N Logs Apps, each with its own token)

## Indexing

With the same REST API, you can index logs directly from your own
application, or you can craft your own "log sender".

**NOTE:**
If you are sending logs from your application use the Elasticsearch HTTP
API. If you are sending logs from a Java application use [Elasticsearch Java REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)

Besides specifying your Logs App token as the index name, it's nice
to have a field named "@timestamp".  Its value should be a valid
[ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) timestamp.  If you don't provide a
timestamp, Sematext will add one when it receives your logs.

For example, you can send a log like this:

``` bash
NOW=`date "+%Y-%m-%dT%H:%M:%S"`
curl -XPOST https://logsene-receiver.sematext.com/$YOUR_TOKEN_HERE/mytype/ -d '
{
  "@timestamp": "'$NOW'",
  "message": "Hello World!"
}'
```

This will index a simple "hello world" message to Logs App. That event
would have the current timestamp and will go to your app
(provided that the $YOUR\_TOKEN\_HERE variable contains your token),
within a type named "mytype". The type is a logical division of events.

Typically, you'd put events with different structures in different
types. For example, syslog messages in a type called "syslog", apache
logs in a type called "apache". Essentially, the type can be anything
and it is the token of your application that has to match.

For performance reasons we highly recommend using the [Bulk API](http://www.elasticsearch.org/guide/reference/api/bulk.html),
because it allows you to send multiple events with a single request. For
example, the following request sends three events:

``` bash
NOW=`date "+%Y-%m-%dT%H:%M:%S"`

echo '{ "index" : { "_index": "LOGSENE_APP_TOKEN_GOES_HERE", "_type" : "mytype" } }
{ "@timestamp": "'$NOW'", "severity_numeric" : 1 }
{ "index" : { "_index": "LOGSENE_APP_TOKEN_GOES_HERE", "_type" : "mytype" } }
{ "@timestamp": "'$NOW'", "message" : "hello again" }
{ "index" : { "_index": "LOGSENE_APP_TOKEN_GOES_HERE", "_type" : "mytype" } }
{ "@timestamp": "'$NOW'", "alternate_message": "fields can be added and removed at will" }' > req

curl -XPOST https://logsene-receiver.sematext.com/_bulk --data-binary @req; echo
```

## Default Log Index Mapping

A [mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/glossary.html#mapping)
is a way to define how your logs are indexed - which fields are in each log event and how each field is indexed. There is no "default" index mapping.  Sematext automatically creates the mapping in each Logs App when your first ship your logs.  Each App can have its own mapping and it can be changed at any time from within Sematext.  There are some [special fields](special-fields) though.

  - the **@timestamp** field is an
    [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) date.  See [Supported Date Formats](supported-date-formats).
  - the **geoip** field is an object that contains a **location** [geo point](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html)
    field (this works well if you're using [Logstash](logstash))
  - **host**, **facility**, **severity**, **syslog-tag**, **source**, and **tags** are
    [Special Fields] that are not analyzed, which enables only exact matches (you can still use wildcards, for example
    to search for **web-server\*** and get **web-server01**)
  - all string fields are analyzed by whitespace and lowercased by
    default, thus makign it possible to search for **message:hello** and match events
    with **Hello World** in the **message** field

## Custom Log Index Mapping

If the default log index fields (also known as index mapping) don't fit
your needs you can create completely custom index mapping. See [Custom Logsene Mapping Template How-To](http://blog.sematext.com/2015/01/20/custom-elasticsearch-index-templates-in-logsene/).

Note that if you have N different log structures, the best way to
handle that is by creating N Logs Management Apps, each with its own index
mapping. For example, you may have web server logs, your system logs in
/var/log/messages, and your custom application logs. Each of these 3
types of logs has a different structure.

The web server logs probably use Apache Common Log format, the logs in /var/log/messages have syslog
structure, and your own application's logs can be in any format your
application happens to use.

To handle all 3 log formats elegantly simply create 3 separate Logs Management apps and use a different format for
each of them. See [Custom Logsene Mapping Template How-To](http://blog.sematext.com/2015/01/20/custom-elasticsearch-index-templates-in-logsene/) for details.
