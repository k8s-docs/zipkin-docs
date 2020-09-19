---
title: "zipkin-server"
linkTitle: ""
weight: 8
type: "docs"
description: >
  Zipkin 服务器是Java 1.8+ 服务, 打包为一个可执行的jar.
---

Span storage and collectors are [configurable](#configuration). By default, storage is in-memory,
the HTTP collector (POST /api/v2/spans endpoint) is enabled, and the server listens on port 9411.

Zipkin Server is implemented with [Armeria](https://github.com/line/armeria). While it uses [Spring Boot](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
internally, Zipkin Server should not be considered a normal Spring Boot application.

## 自定义服务器不支持

By Custom servers we mean trying to use/embed `zipkin` as part of _an application you package_ (e.g. adding `zipkin-server` dependency to a Spring-boot application) instead of the packaged application we release.

For proper usage, see the guides below.

## 快速开始

The quickest way to get started is to fetch the [latest released server](https://search.maven.org/remote_content?g=io.zipkin&a=zipkin-server&v=LATEST&c=exec) as a self-contained executable jar. Note that the Zipkin server requires minimum JRE 8. For example:

```bash
$ curl -sSL https://zipkin.io/quickstart.sh | bash -s
$ java -jar zipkin.jar
```

Once you've started, browse to http://your_host:9411 to find traces!

## 端点

The following endpoints are defined under the base url http://your_host:9411

- / - [UI](../zipkin-ui)
- /config.json - [Configuration for the UI](#configuration-for-the-ui)
- /api/v2 - [API](https://zipkin.io/zipkin-api/#/)
- /health - Returns 200 status if OK
- /info - Provides the version of the running instance
- /metrics - Includes collector metrics broken down by transport type
- /prometheus - Prometheus scrape endpoint

The [legacy /api/v1 API](https://zipkin.io/zipkin-api/#/) is still supported. Backends are decoupled from the
HTTP API via data conversion. This means you can still accept legacy data on new backends and visa versa. Enter
`https://zipkin.io/zipkin-api/zipkin-api.yaml` into the explore box of the Swagger UI to view the old definition

### CORS (跨来源资源共享)

By default, all endpoints under `/api/v2` are configured to **allow** cross-origin requests.

This can be changed by modifying the property `zipkin.query.allowed-origins`.

For example, to allow CORS requests from `http://foo.bar.com`:

```
ZIPKIN_QUERY_ALLOWED_ORIGINS=http://foo.bar.com
```

See [Configuration](#configuration) for more about how Zipkin is configured.

### 服务和 span 名称查询

The [Zipkin API](https://zipkin.io/zipkin-api/#/default/get_services) does not include
a parameter for how far back to look for service or span names. In order
to prevent excessive load, service and span name queries are limited by
`QUERY_LOOKBACK`, which defaults to 24hrs (two daily buckets: one for
today and one for yesterday)

## 日志

By default, zipkin writes log messages to the console at INFO level and above. You can adjust
categories using the `logging.level.XXX` property.

For example, if you want to enable debug logging for all zipkin categories, you can start the server like so:

```bash
$ java -jar zipkin.jar --logging.level.zipkin2=DEBUG
```

See [Configuration](#configuration) for more about how Zipkin is configured.

### 高级日志记录配置

Under the covers, the server uses [Spring Boot - Logback integration](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-logback-for-logging).
For example, you can add `--logging.exception-conversion-word=%wEx{full}` to dump full stack traces
instead of truncated ones.

## 度量

Collector Metrics are exported to the path `/metrics`. These and additional metrics are exported
to the path `/prometheus`.

### 例 Prometheus 配置

Here's an example `/prometheus` configuration, using the Prometheus
exposition [text format version 0.0.4](https://prometheus.io/docs/instrumenting/exposition_formats/)

```yaml
- job_name: "zipkin"
  scrape_interval: 5s
  metrics_path: "/prometheus"
  static_configs:
    - targets: ["localhost:9411"]
  metric_relabel_configs:
    # Response code count
    - source_labels: [__name__]
      regex: '^status_(\d+)_(.*)$'
      replacement: "${1}"
      target_label: status
    - source_labels: [__name__]
      regex: '^status_(\d+)_(.*)$'
      replacement: "${2}"
      target_label: path
    - source_labels: [__name__]
      regex: '^status_(\d+)_(.*)$'
      replacement: "http_requests_total"
      target_label: __name__
```

### 收集器

Collector metrics are broken down by transport. The following are exported to the "/metrics" endpoint:

| Metric                                                | Description                                                                           |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------- |
| counter.zipkin_collector.messages.\$transport         | cumulative messages received; should relate to messages reported by instrumented apps |
| counter.zipkin_collector.messages_dropped.\$transport | cumulative messages dropped; reasons include client disconnects or malformed content  |
| counter.zipkin_collector.bytes.\$transport            | cumulative message bytes                                                              |
| counter.zipkin_collector.spans.\$transport            | cumulative spans read; should relate to messages reported by instrumented apps        |
| counter.zipkin_collector.spans_dropped.\$transport    | cumulative spans dropped; reasons include sampling or storage failures                |
| gauge.zipkin_collector.message_spans.\$transport      | last count of spans in a message                                                      |
| gauge.zipkin_collector.message_bytes.\$transport      | last count of bytes in a message                                                      |

## 配置

我们支持 ENV 变量配置, 如 `STORAGE_TYPE=cassandra3`, 因为他们所熟悉的管理员和容易在运行时环境中使用如 Docker.

以下是 zipkin 的顶级配置:

- `QUERY_PORT`: 监听端口的 HTTP API 和 Web UI; 默认为 9411
- `QUERY_ENABLED`: `false` 禁用 HTTP 读 '/api/v2' 下端点. 这也将禁用 UI, 因为它依赖于 API. 如果你唯一的目标就是限制搜索, 用 `SEARCH_ENABLED` 代替. 默认为 true
- `SEARCH_ENABLED`: `false` 禁止查询 API 和任何索引或后处理中搜索 在收集到的支持搜索.
  这不会禁用整个 UI, 通过 ID 和依赖查询仍然痕迹进行操作.
  禁用此当你使用其他服务 (如日志) 找到踪迹标识.默认为 true
- `QUERY_TIMEOUT`: 设置查询请求的硬超时. 接受任何时间串 (例如, 100ms). 0 值将完全禁用超时. 默认为 11s.
- `QUERY_LOG_LEVEL`: 日志级别写入到控制台; 默认为 INFO
- `QUERY_NAMES_MAX_AGE`: Controls the value of the `max-age` header zipkin-server responds with on
  http requests for autocompleted values in the UI (service names for example). Defaults to 300 seconds.
- `QUERY_LOOKBACK`: 多少毫秒查询可以从 endTs 回头; 默认值 24 小时 (每天两个水桶: 一个今天和一个昨天)
- `STORAGE_TYPE`: SpanStore 履行: 之一 `mem`, `mysql`, `cassandra`, `elasticsearch`
- `COLLECTOR_SAMPLE_RATE`: Percentage of traces to retain, defaults to always sample (1.0).
- `AUTOCOMPLETE_KEYS`: span 标签键的名单将由 `/api/v2/autocompleteTags` 端点返回 ; 代码键应该是逗号分隔 例如 "instance_id,user_id,env"
- `AUTOCOMPLETE_TTL`: How long in milliseconds to suppress calls to write the same autocomplete key/value pair. Default 3600000 (1 hr)

### 配置文件覆盖

Under the scenes, all configuration are managed by Spring Boot. This means that properties may also
be overridden by system properties or any other alternative [supported by Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html).

We use [yaml configuration](src/main/resources/zipkin-server-shared.yml) to bind shorter or more
idiomatic ENV variables to the Spring properties ultimately in use. While most users should only use
environment variables, some may desire a properties file approach to override settings. For example,
knowing we set `spring.config.name=zipkin-server`, Spring Boot will automatically look for a file
named `zipkin-server.properties` in the current directory, and the same properties we set in yaml
can be overridden that way.

If you choose to use property-based configuration instead of ENV variables, you are choosing to
self-support your configuration. This means you'll use [Spring Boot documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)
or [StackOverflow](https://stackoverflow.com/questions/tagged/spring-boot) to resolve concerns
related to property resolution as opposed to raising issues or using our chat support. We have to
mention this because configuration of Spring implies vast responsibility and our resources must be
conserved for Zipkin related tasks.

## UI

Zipkin has a web UI, automatically included in the exec jar, and is hosted by default on port 9411.

When the UI loads, it reads default configuration from the `/config.json` endpoint.

| Attribute                | Property                             | Description                                                                                                                                                                                                                                      |
| ------------------------ | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| environment              | zipkin.ui.environment                | The value here becomes a label in the top-right corner. Not required.                                                                                                                                                                            |
| defaultLookback          | zipkin.ui.default-lookback           | Default duration in millis to look back when finding traces. Affects the "Start time" element in the UI. Defaults to 900000 (15 minutes in millis).                                                                                              |
| searchEnabled            | zipkin.ui.search-enabled             | If the Discover screen is enabled. Defaults to true.                                                                                                                                                                                             |
| queryLimit               | zipkin.ui.query-limit                | Default limit for Find Traces. Defaults to 10.                                                                                                                                                                                                   |
| instrumented             | zipkin.ui.instrumented               | Which sites this Zipkin UI covers. Regex syntax. e.g. `http:\/\/example.com\/.*` Defaults to match all websites (`.*`).                                                                                                                          |
| logsUrl                  | zipkin.ui.logs-url                   | Logs query service url pattern. If specified, a button will appear on the trace page and will replace {traceId} in the url by the traceId. Not required.                                                                                         |
| supportUrl               | zipkin.ui.support-url                | A URL where a user can ask for support. If specified, a link will be placed in the side menu to this URL, for example a page to file support tickets. Not required.                                                                              |
| archivePostUrl           | zipkin.ui.archive-post-url           | Url to POST the current trace in Zipkin v2 json format. e.g. 'https://longterm/api/v2/spans'. If specified, a button will appear on the trace page accordingly. Not required.                                                                    |
| archiveUrl               | zipkin.ui.archive-url                | Url to a web application serving an archived trace, templated by '{traceId}'. e.g. https://longterm/zipkin/trace/{traceId}'. This is shown in a confirmation message after a trace is successfully POSTed to the `archivePostUrl`. Not required. |
| dependency.enabled       | zipkin.ui.dependency.enabled         | If the Dependencies screen is enabled. Defaults to true.                                                                                                                                                                                         |
| dependency.lowErrorRate  | zipkin.ui.dependency.low-error-rate  | The rate of error calls on a dependency link that turns it yellow. Defaults to 0.5 (50%) set to >1 to disable.                                                                                                                                   |
| dependency.highErrorRate | zipkin.ui.dependency.high-error-rate | The rate of error calls on a dependency link that turns it red. Defaults to 0.75 (75%) set to >1 to disable.                                                                                                                                     |
| basePath                 | zipkin.ui.basepath                   | path prefix placed into the <base> tag in the UI HTML; useful when running behind a reverse proxy. Default "/zipkin"                                                                                                                             |

To map properties to environment variables, change them to upper-underscore case format. For
example, if using docker you can set `ZIPKIN_UI_QUERY_LIMIT=100` to affect `$.queryLimit` in `/config.json`.

### 跟踪档案

Most production Zipkin clusters store traces with a limited TTL. This makes it a bit inconvenient to
share a trace, as the link to it will expire after a few days.

The "archive a trace" feature helps with this. Launch a second zipkin server pointing to a storage with a longer
TTL than the regular one and set the archivePostUrl and archiveUrl UI configs pointing to this second server.
Once archivePostUrl is set, a new "Archive Trace" button will appear on the trace view page.

## 存储

### 内存存储

Zipkin's [In-Memory Storage](../zipkin/src/main/java/zipkin2/storage/InMemoryStorage.java) holds all
data in memory, purging older data upon a span limit. It applies when `STORAGE_TYPE` is unset or
set to the value `mem`.

    * `MEM_MAX_SPANS`: Oldest traces (and their spans) will be purged first when this limit is exceeded. Default 500000

Example usage:

```bash
$ java -jar zipkin.jar
```

Note: this storage component was primarily developed for testing and as a means to get Zipkin server
up and running quickly without external dependencies. It is not viable for high work loads. That
said, if you encounter out-of-memory errors, try decreasing `MEM_MAX_SPANS` or increasing the heap
size (-Xmx).

Exampled of doubling the amount of spans held in memory:

```bash
$ MEM_MAX_SPANS=1000000 java -Xmx1G -jar zipkin.jar
```

### Cassandra 存储

Zipkin's [Cassandra storage component](../zipkin-storage/cassandra)
supports version 3.11+ and applies when `STORAGE_TYPE` is set to `cassandra3`:

    * `CASSANDRA_KEYSPACE`: The keyspace to use. Defaults to "zipkin2"
    * `CASSANDRA_CONTACT_POINTS`: Comma separated list of host addresses part of Cassandra cluster. You can also specify a custom port with 'host:port'. Defaults to localhost on port 9042.
    * `CASSANDRA_LOCAL_DC`: Name of the datacenter that will be considered "local" for latency load balancing. When unset, load-balancing is round-robin.
    * `CASSANDRA_ENSURE_SCHEMA`: Ensuring cassandra has the latest schema. If enabled tries to execute scripts in the classpath prefixed with `cassandra-schema-cql3`. Defaults to true
    * `CASSANDRA_USERNAME` and `CASSANDRA_PASSWORD`: Cassandra authentication. Will throw an exception on startup if authentication fails. No default
    * `CASSANDRA_USE_SSL`: Requires `javax.net.ssl.trustStore` and `javax.net.ssl.trustStorePassword`, defaults to false.

The following are tuning parameters which may not concern all users:

    * `CASSANDRA_MAX_CONNECTIONS`: Max pooled connections per datacenter-local host. Defaults to 8
    * `CASSANDRA_INDEX_CACHE_MAX`: Maximum trace index metadata entries to cache. Zero disables caching. Defaults to 100000.
    * `CASSANDRA_INDEX_CACHE_TTL`: How many seconds to cache index metadata about a trace. Defaults to 60.
    * `CASSANDRA_INDEX_FETCH_MULTIPLIER`: How many more index rows to fetch than the user-supplied query limit. Defaults to 3.

Example usage with Cassandra connection and query logging:

```bash
$ STORAGE_TYPE=cassandra3 java -jar zipkin.jar \
 --logging.level.com.datastax.driver.core.Connection=debug \
 --logging.level.com.datastax.driver.core.QueryLogger.NORMAL=trace
```

### Elasticsearch 存储

Zipkin's [Elasticsearch storage component](../zipkin-storage/elasticsearch)
supports versions 5-7.x and applies when `STORAGE_TYPE` is set to `elasticsearch`

The following apply when `STORAGE_TYPE` is set to `elasticsearch`:

    * `ES_HOSTS`: A comma separated list of elasticsearch base urls to connect to ex. http://host:9200.
                  Defaults to "http://localhost:9200".
    * `ES_PIPELINE`: Indicates the ingest pipeline used before spans are indexed. No default.
    * `ES_TIMEOUT`: Controls the connect, read and write socket timeouts (in milliseconds) for
                    Elasticsearch API. Defaults to 10000 (10 seconds)
    * `ES_INDEX`: The index prefix to use when generating daily index names. Defaults to zipkin.
    * `ES_DATE_SEPARATOR`: The date separator to use when generating daily index names. Defaults to '-'.
    * `ES_INDEX_SHARDS`: The number of shards to split the index into. Each shard and its replicas
                         are assigned to a machine in the cluster. Increasing the number of shards
                         and machines in the cluster will improve read and write performance. Number
                         of shards cannot be changed for existing indices, but new daily indices
                         will pick up changes to the setting. Defaults to 5.
    * `ES_INDEX_REPLICAS`: The number of replica copies of each shard in the index. Each shard and
                           its replicas are assigned to a machine in the cluster. Increasing the
                           number of replicas and machines in the cluster will improve read
                           performance, but not write performance. Number of replicas can be changed
                           for existing indices. Defaults to 1. It is highly discouraged to set this
                           to 0 as it would mean a machine failure results in data loss.
    * `ES_ENSURE_TEMPLATES`: Installs Zipkin index templates when missing. Setting this to false can
                             lead to corrupted data when index templates mismatch expectations. If
                             you set this to false, you choose to troubleshoot your own data or
                             migration problems as opposed to relying on the community for this.
                             Defaults to true.
    * `ES_USERNAME` and `ES_PASSWORD`: Elasticsearch basic authentication, which defaults to empty string.
                                       Use when X-Pack security (formerly Shield) is in place.
    * `ES_CREDENTIALS_FILE`: The location of a file containing Elasticsearch basic authentication
                             credentials, as properties. The username property is
                             `zipkin.storage.elasticsearch.username`, password `zipkin.storage.elasticsearch.password`.
                             This file is reloaded periodically, using `ES_CREDENTIALS_REFRESH_INTERVAL`
                             as the interval. This parameter takes precedence over ES_USERNAME and
                              ES_PASSWORD when specified.
    * `ES_CREDENTIALS_REFRESH_INTERVAL`: Credentials refresh interval in seconds, which defaults to
                                         1 second. This is the maximum amount of time spans will drop due to stale
                                         credentials. Any errors reading the credentials file occur in logs at this rate.
    * `ES_HTTP_LOGGING`: When set, controls the volume of HTTP logging of the Elasticsearch API.
                         Options are BASIC, HEADERS, BODY
    * `ES_SSL_NO_VERIFY`: When true, disables the verification of server's key certificate chain.
                          This is not appropriate for production. Defaults to false.
    * `ES_TEMPLATE_PRIORITY`: The priority value of the composable index templates. This is only applicable
                              for ES version 7.8 or above. Must be set, even to 0, to use composable template

Example usage:

To connect normally:

```bash
$ STORAGE_TYPE=elasticsearch ES_HOSTS=http://myhost:9200 java -jar zipkin.jar
```

To log Elasticsearch API requests:

```bash
$ STORAGE_TYPE=elasticsearch ES_HTTP_LOGGING=BASIC java -jar zipkin.jar
```

#### 使用自定义密钥存储或信托存储 (SSL)

If your Elasticsearch endpoint customized SSL configuration (for example self-signed) certificates,
you can use any of the following [subset of JSSE properties](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#T6) to connect.

- javax.net.ssl.keyStore
- javax.net.ssl.keyStorePassword
- javax.net.ssl.keyStoreType
- javax.net.ssl.trustStore
- javax.net.ssl.trustStorePassword
- javax.net.ssl.trustStoreType

Usage example:

```bash
$ JAVA_OPTS='-Djavax.net.ssl.keyStore=keystore.p12 -Djavax.net.ssl.keyStorePassword=keypassword -Djavax.net.ssl.keyStoreType=PKCS12 -Djavax.net.ssl.trustStore=truststore.p12 -Djavax.net.ssl.trustStorePassword=trustpassword -Djavax.net.ssl.trustStoreType=PKCS12'
$ STORAGE_TYPE=elasticsearch java $JAVA_OPTS -jar zipkin.jar
```

Under the scenes, these map to properties prefixed `zipkin.storage.elasticsearch.ssl.`, which affect
the Armeria client used to connect to Elasticsearch.

The above properties allow the most common SSL setup to work out of box. If you need more
customization, please make a comment in [this issue](https://github.com/openzipkin/zipkin/issues/2774).

### 传统(v1)存储组件

The following components are no longer encouraged, but exist to help aid
transition to supported ones. These are indicated as "v1" as they use
data layouts based on Zipkin's V1 Thrift model, as opposed to the
simpler v2 data model currently used.

#### MySQL 存储

Zipkin's [MySQL component](../zipkin-storage/mysql-v1) is tested against MySQL
5.7 and applies when `STORAGE_TYPE` is set to `mysql`:

    * `MYSQL_DB`: The database to use. Defaults to "zipkin".
    * `MYSQL_USER` and `MYSQL_PASS`: MySQL authentication, which defaults to empty string.
    * `MYSQL_HOST`: Defaults to localhost
    * `MYSQL_TCP_PORT`: Defaults to 3306
    * `MYSQL_MAX_CONNECTIONS`: Maximum concurrent connections, defaults to 10
    * `MYSQL_USE_SSL`: Requires `javax.net.ssl.trustStore` and `javax.net.ssl.trustStorePassword`, defaults to false.

Note: This module is not recommended for production usage. Before using this,
you must [apply the schema](../zipkin-storage/mysql-v1#applying-the-schema).

Alternatively you can use `MYSQL_JDBC_URL` and specify the complete JDBC url yourself. Note that the URL constructed by
using the separate settings above will also include the following parameters:
`?autoReconnect=true&useSSL=false&useUnicode=yes&characterEncoding=UTF-8`. If you specify the JDBC url yourself, add
these parameters as well.

Example usage:

```bash
$ STORAGE_TYPE=mysql MYSQL_USER=root java -jar zipkin.jar
```

#### Cassandra 存储

Zipkin's [Legacy (v1) Cassandra storage component](../zipkin-storage/cassandra-v1)
supports version 2.2+ and applies when `STORAGE_TYPE` is set to `cassandra`:

The environment variables are the same as `STORAGE_TYPE=cassandra3`,
except the default keyspace name is "zipkin".

Example usage:

```bash
$ STORAGE_TYPE=cassandra java -jar zipkin.jar
```

### 节流的存储 (Experimental)

These settings can be used to help tune the rate at which Zipkin flushes data to another, underlying
`StorageComponent` (such as Elasticsearch):

    * `STORAGE_THROTTLE_ENABLED`: Enables throttling
    * `STORAGE_THROTTLE_MIN_CONCURRENCY`: Minimum number of Threads to use for writing to storage.
    * `STORAGE_THROTTLE_MAX_CONCURRENCY`: Maximum number of Threads to use for writing to storage.
    * `STORAGE_THROTTLE_MAX_QUEUE_SIZE`: How many messages to buffer while all Threads are writing data before abandoning a message (0 = no buffering).

As this feature is experimental, it is not recommended to run this in production environments.

## 收集器

### HTTP 收集器

The HTTP collector is enabled by default. It accepts spans via `POST /api/v1/spans` and `POST /api/v2/spans`.
The HTTP collector supports the following configuration:

| Property                        | Environment Variable     | Description                                              |
| ------------------------------- | ------------------------ | -------------------------------------------------------- |
| `zipkin.collector.http.enabled` | `COLLECTOR_HTTP_ENABLED` | `false` disables the HTTP collector. Defaults to `true`. |

### Scribe (Legacy) 收集器

A collector supporting Scribe is enabled when `COLLECTOR_SCRIBE_ENABLED=true`. New
sites are discouraged from using this collector as Scribe is an archived
technology.

| Environment Variable | Property                           | Description                                                         |
| -------------------- | ---------------------------------- | ------------------------------------------------------------------- |
| `COLLECTOR_PORT`     | `zipkin.collector.scribe.port`     | The port to listen for thrift RPC scribe requests. Defaults to 9410 |
| `SCRIBE_CATEGORY`    | `zipkin.collector.scribe.category` | Category zipkin spans will be consumed from. Defaults to `zipkin`   |

### ActiveMQ 收集器

The [ActiveMQ Collector](../zipkin-collector/activemq) is enabled when `ACTIVEMQ_URL` is set to a v5.x broker. The following settings apply in this case.

| Environment Variable         | Property                                     | Description                                                                                                                                                                  |
| ---------------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `COLLECTOR_ACTIVEMQ_ENABLED` | `zipkin.collector.activemq.enabled`          | `false` disables the ActiveMQ collector. Defaults to `true`.                                                                                                                 |
| `ACTIVEMQ_URL`               | `zipkin.collector.activemq.url`              | [Connection URL](https://activemq.apache.org/uri-protocols) to the ActiveMQ broker, ex. `tcp://localhost:61616` or `failover:(tcp://localhost:61616,tcp://remotehost:61616)` |
| `ACTIVEMQ_QUEUE`             | `zipkin.collector.activemq.queue`            | Queue from which to collect span messages. Defaults to `zipkin`                                                                                                              |
| `ACTIVEMQ_CLIENT_ID_PREFIX`  | `zipkin.collector.activemq.client-id-prefix` | Client ID prefix for queue consumers. Defaults to `zipkin`                                                                                                                   |
| `ACTIVEMQ_CONCURRENCY`       | `zipkin.collector.activemq.concurrency`      | Number of concurrent span consumers. Defaults to `1`                                                                                                                         |
| `ACTIVEMQ_USERNAME`          | `zipkin.collector.activemq.username`         | Optional username to connect to the broker                                                                                                                                   |
| `ACTIVEMQ_PASSWORD`          | `zipkin.collector.activemq.password`         | Optional password to connect to the broker                                                                                                                                   |

Example usage:

```bash
$ ACTIVEMQ_URL=tcp://localhost:61616 java -jar zipkin.jar
```

### Kafka 收集器

The Kafka collector is enabled when `KAFKA_BOOTSTRAP_SERVERS` is set to
a v0.10+ server. The following settings apply in this case. Some settings
correspond to "New Consumer Configs" in [Kafka documentation](https://kafka.apache.org/documentation/#newconsumerconfigs).

| Variable                  | New Consumer Config | Description                                                                                  |
| ------------------------- | ------------------- | -------------------------------------------------------------------------------------------- |
| `COLLECTOR_KAFKA_ENABLED` | N/A                 | `false` disables the Kafka collector. Defaults to `true`.                                    |
| `KAFKA_BOOTSTRAP_SERVERS` | bootstrap.servers   | Comma-separated list of brokers, ex. 127.0.0.1:9092. No default                              |
| `KAFKA_GROUP_ID`          | group.id            | The consumer group this process is consuming on behalf of. Defaults to `zipkin`              |
| `KAFKA_TOPIC`             | N/A                 | Comma-separated list of topics that zipkin spans will be consumed from. Defaults to `zipkin` |
| `KAFKA_STREAMS`           | N/A                 | Count of threads consuming the topic. Defaults to `1`                                        |

Example usage:

```bash
$ KAFKA_BOOTSTRAP_SERVERS=127.0.0.1:9092 \
    java -jar zipkin.jar
```

#### 其它 kafka 的消费特性

You may need to set other
[Kafka consumer properties](https://kafka.apache.org/documentation/#newconsumerconfigs), in
addition to the ones with explicit properties defined by the collector. In this case, you need to
prefix that property name with `zipkin.collector.kafka.overrides` and pass it as a system property
argument.

For example, to override `auto.offset.reset`, you can set a system property named
`zipkin.collector.kafka.overrides.auto.offset.reset`:

```bash
$ KAFKA_BOOTSTRAP_SERVERS=127.0.0.1:9092 \
    java -Dzipkin.collector.kafka.overrides.auto.offset.reset=latest -jar zipkin.jar
```

#### 详细的例子

Example targeting Kafka running in Docker:

```bash
$ export KAFKA_BOOTSTRAP_SERVERS=$(docker-machine ip `docker-machine active`)
# Run Kafka in the background
$ docker run -d -p 9092:9092 \
    --env ADVERTISED_HOST=$KAFKA_BOOTSTRAP_SERVERS \
    --env AUTO_CREATE_TOPICS=true \
    spotify/kafka
# Start the zipkin server, which reads $KAFKA_BOOTSTRAP_SERVERS
$ java -jar zipkin.jar
```

Multiple bootstrap servers:

```bash
$ KAFKA_BOOTSTRAP_SERVERS=broker1.local:9092,broker2.local:9092 \
    java -jar zipkin.jar
```

Alternate topic name(s):

```bash
$ KAFKA_BOOTSTRAP_SERVERS=127.0.0.1:9092 \
    java -Dzipkin.collector.kafka.topic=zapkin,zipken -jar zipkin.jar
```

Specifying bootstrap servers as a system property, instead of an environment variable:

```bash
$ java -Dzipkin.collector.kafka.bootstrap-servers=127.0.0.1:9092 \
    -jar zipkin.jar
```

### RabbitMQ 收集器

The [RabbitMQ collector](../zipkin-collector/rabbitmq) will be enabled when the `addresses` or `uri` for the RabbitMQ server(s) is set.

Example usage:

```bash
$ RABBIT_ADDRESSES=localhost java -jar zipkin.jar
```

### gRPC 收集器 (试验)

You can enable a gRPC span collector endpoint by setting `COLLECTOR_GRPC_ENABLED=true`. The
`zipkin.proto3.SpanService/Report` endpoint will run on the same port as normal HTTP (9411).

Example usage:

```bash
$ COLLECTOR_GRPC_ENABLED=true java -jar zipkin.jar
```

As this service is experimental, it is not recommended to run this in production environments.

## 自跟踪

Self tracing exists to help troubleshoot performance of the zipkin-server. Production deployments
who enable self-tracing should lower the sample rate from 1.0 (100%) to a much smaller rate, like
0.001 (0.1% or 1 out of 1000).

When `zipkin.self-tracing.enabled=true`, Zipkin will self-trace calls to the API under the service
name "zipkin-server".

| Variable                    | Property                           | Description                                                              |
| --------------------------- | ---------------------------------- | ------------------------------------------------------------------------ |
| SELF_TRACING_ENABLED        | zipkin.self-tracing.enabled        | Set to true to enable self-tracing. Defaults to false                    |
| SELF_TRACING_SAMPLE_RATE    | zipkin.self-tracing.sample-rate    | Percentage of self-traces to retain, defaults to always sample (1.0).    |
| SELF_TRACING_FLUSH_INTERVAL | zipkin.self-tracing.flush-interval | Interval in seconds to flush self-tracing data to storage. Defaults to 1 |

### 128 位跟踪标识

Zipkin supports 64 and 128-bit trace identifiers, typically serialized
as 16 or 32 character hex strings. By default, spans reported to zipkin
with the same trace ID will be considered in the same trace.

For example, `463ac35c9f6413ad48485a3953bb6124` is a 128-bit trace ID,
while `48485a3953bb6124` is a 64-bit one.

Note: Span (or parent) IDs within a trace are 64-bit regardless of the
length or value of their trace ID.

#### 迁移从 64 到 128 位跟踪标识

Unless you only issue 128-bit traces when all applications support them,
the process of updating applications from 64 to 128-bit trace IDs results
in a mixed state. This mixed state is mitigated by the setting
`STRICT_TRACE_ID=false`, explained below. Once a migration is complete,
remove the setting `STRICT_TRACE_ID=false` or set it to true.

Here are a few trace IDs the help what happens during this setting.

- Trace ID A: 463ac35c9f6413ad48485a3953bb6124
- Trace ID B: 48485a3953bb6124
- Trace ID C: 463ac35c9f6413adf1a48a8cff464e0e
- Trace ID D: 463ac35c9f6413ad

In a 64-bit environment, trace IDs will look like B or D above. When an
application upgrades to 128-bit instrumentation and decides to create a
128-bit trace, its trace IDs will look like A or C above.

Applications who aren't yet 128-bit capable typically only retain the
right-most 16 characters of the trace ID. When this happens, the same
trace could be reported as trace ID A or trace ID B.

By default, Zipkin will think these are different trace IDs, as they are
different strings. During a transition from 64-128 bit trace IDs, spans
would appear split across two IDs. For example, it might start as trace
ID A, but the next hop might truncate it to trace ID B. This would render
the system unusable for applications performing upgrades.

One way to address this problem is to not use 128-bit trace IDs until
all applications support them. This prevents a mixed scenario at the cost
of coordination. Another way is to set `STRICT_TRACE_ID=false`.

When `STRICT_TRACE_ID=false`, only the right-most 16 of a 32 character
trace ID are considered when grouping or retrieving traces. This setting
should only be applied when transitioning from 64 to 128-bit trace IDs
and removed once the transition is complete.

See https://github.com/openzipkin/b3-propagation/issues/6 for the status
of known open source libraries on 128-bit trace identifiers.

See `zipkin2.storage.StorageComponent.Builder` for even more details!

## TLS/SSL

Zipkin-server can be made to run with TLS if needed:

```bash
# assuming you generate the key like this
keytool -genkeypair -alias mysite -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore zipkin.p12 -validity 3650

java -jar zipkin.jar --armeria.ssl.key-store=zipkin.p12 --armeria.ssl.key-store-type=PKCS12 --armeria.ssl.key-store-password=123123 --armeria.ssl.key-alias=mysite  --armeria.ssl.enabled=true --armeria.ports[0].port=9411 --armeria.ports[0].protocols[0]=https
```

## 与 docker 运行

Released versions of zipkin-server are published to Docker Hub as `openzipkin/zipkin`.
See [docker-zipkin](https://github.com/openzipkin/docker-zipkin) for details.

## 本地构建

To build and run the server from the currently checked out source, enter the following.

```bash
# Build the server and also make its dependencies
$ ./mvnw -q --batch-mode -DskipTests --also-make -pl zipkin-server clean install
# Run the server
$ java -jar ./zipkin-server/target/zipkin-server-*exec.jar
# or Run the slim server
$ java -jar ./zipkin-server/target/zipkin-server-*slim.jar
```
