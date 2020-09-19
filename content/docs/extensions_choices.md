---
title: "服务器扩展和选择"
linkTitle: ""
weight: 4
type: "docs"
description: >
  Zipkin 服务器为 span 收集和储存捆绑扩展.
  默认 spans 可以收集 HTTP, Kafka 或 RabbitMQ 传输并存储在存储器或在 MySQL, Cassandra 或 Elasticsearch.
---

## 服务器扩展

以下模块添加储存或运输扩展默认的服务器构建.
请参考他们的个人文件进行安装和配置指南.

## OpenZipkin 支持的

The following extensions are supported by the OpenZipkin team and are hosted at the OpenZipkin GitHub group. You can reach out to the team on Zipkin Gitter chat.

Type Module Related product Other notes
Collector zipkin-aws collector-sqs AWS SQS
Collector zipkin-aws collector-kinesis AWS Kinesis
Storage zipkin-aws storage-elasticsearch-aws AWS Elasticsearch Service
Storage zipkin-aws storage-xray AWS X-Ray only supports sending to an XRay UDP daemon
Storage zipkin-gcp storage-stackdriver GCP Stackdriver only supports sending to an Stackdriver

## 社区支持

Type Module Related product Other notes
Storage zipkin-logzio Logz.io Supports both sending and reading back data to and from Logz.io: Secure & Scalable Log Management with Cloud-Based ELK
Storage zipkin-scouter storage-scouter Scouter APM only supports sending to an Scouter
Storage zipkin-storage-kafka Apache kafka Supports aggregation and indexing of tracing data via Kafka Streams State Store.
Reporter spring-cloud-sleuth-haystack-reporter Haystack Supports sending data to Haystack, a resilient, scalable tracing and analysis system.

## 备用服务器

The OpenZipkin team publish apis, data formats, and shared libraries that allow alternate backends to process the same data sent to the default Zipkin server.

### 社区支持

Listed below are alternative backends that accept Zipkin format. Some use the same code as Zipkin on the same endpoints while others are on alternative endpoints or partially support features. In any case, the following aim to allow existing zipkin clients to use backends the OpenZipkin team does not support. Hence, direct questions to their respective communities.

Apache SkyWalking

When zipkin-receiver is enabled, Skywalking exposes the same HTTP POST endpoints Zipkin does
http port 9411 accepts /api/v1/spans (thrift, json) and /api/v2/spans (json, proto) POST requests.
this extension uses the same encoding library and same endpoints as Zipkin does.
Jaeger
When COLLECTOR_ZIPKIN_HTTP_PORT=9411 is set, Jaeger exposes a partial implementation of Zipkin’s HTTP POST endpoints
http port 9411 accepts /api/v1/spans (thrift, json) and /api/v2/spans (json, proto) POST requests.
When SPAN_STORAGE_TYPE=kafka and zipkin-thrift, Jaeger reads Zipkin v1 thrift encoded span messages from a Kafka topic.
Note: The above is a deprecated practice in Zipkin. Most instrumentation bundle multiple spans per message in v2 format.
Pitchfork
Pitchfork exposes the same HTTP POST endpoints Zipkin does
http port 9411 accepts /api/v1/spans (thrift, json) and /api/v2/spans (json, proto) POST requests.
Did we miss a server extension or alternative? Please open a pull-request to openzipkin.github.io.
