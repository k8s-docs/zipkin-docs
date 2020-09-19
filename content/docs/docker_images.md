---
title: "zipkin Docker 镜像"
linkTitle: "Docker 镜像"
weight: 8
type: "docs"
description: >
  该目录包含用于构建和发布Zipkin 的 Docker镜像资产.
---

## 生产镜像

唯一 zipkin 生产的镜像在这里建:

- openzipkin/zipkin: 承载 zipkin UI 核心服务器映像, API 和收集功能.
- openzipkin/zipkin-slim: 剥离服务器映像承载 zipkin UI 和 API 功能, 但只支持使用 HTTP 或 GRPC 跨度收集器在存储器或存储 Elasticsearch .

## 测试镜像

我们还提供了不适合生产了一批镜像, 而以简化演示和集成测试.
我们设计这些是小，容易启动.
我们通过这样做重复使用相同的基础层 `openzipkin/zipkin`, 和设置模式，其中有关.

- [openzipkin/zipkin-cassandra](storage/cassandra/README.md) - 运行 cassandra 与 zipkin 的模式初始化
- [openzipkin/zipkin-elasticsearch6](storage/elasticsearch6/README.md) - 运行 Elasticsearch 6.x 的
- [openzipkin/zipkin-elasticsearch7](storage/elasticsearch7/README.md) - 运行 Elasticsearch 7.x 版
- [openzipkin/zipkin-kafka](collector/kafka/README.md) - 同时运行 kafka+ZooKeeper
- [openzipkin/zipkin-mysql](storage/mysql/README.md) - MySQL 的运行与 zipkin 的模式初始化
- [openzipkin/zipkin-ui](lens/README.md) - 直接与 NGINX 服务于（镜片）UI

## 入门

zipkin 没有依赖, 例如，你可以像下面这样运行在内存中的 zipkin 服务器:

```sh
docker run -d -p 9411:9411 openzipkin/zipkin-slim
```

见在 (docker IP):9411

在用户界面 - 点击 zipkin-server, 然后点击 "Find Traces".

我们还提供[例如撰写文件](examples/README.md) 集成了收藏家和存储,如 kafka 或 Elasticsearch.

## 配置

配置是通过环境变量, 通过[zipkin-server](https://github.com/openzipkin/zipkin/blob/master/zipkin-server/README.md)定义.
值得注意的是, 你要看看`STORAGE_TYPE`环境变量, 其中包括 "cassandra", "mysql" 和 "elasticsearch".

注意: `openzipkin/zipkin-slim` 图片仅支持 "elasticsearch" 存储.
要使用其他的存储类型, 您必须使用主镜像 `openzipkin/zipkin`.

当在 docker 里, 以下环境变量也适用

- `JAVA_OPTS`: 使用设置 Java 参数, 如堆大小或信任存储位置.
- `STORAGE_PORT_9042_TCP_ADDR` -- 一个 Cassandra 节点侦听端口 9042.
  当您启动容器此环境变量通常通过连接容器的运行设置 `zipkin-cassandra` 如 "storage".
- `STORAGE_PORT_3306_TCP_ADDR` -- 一个 MySQL 节点监听 3306 端口.
  当您启动容器此环境变量通常通过连接容器的运行设置 `zipkin-mysql` 如 "storage".
- `STORAGE_PORT_9200_TCP_ADDR` -- 一个 Elasticsearch 节点侦听端口 9200.
  当您启动容器此环境变量通常通过连接容器的运行设置 `zipkin-elasticsearch` 如 "storage" .
  当 `ES_HOSTS` 或 `ES_AWS_DOMAIN` 设置这被忽略 .
- `KAFKA_PORT_2181_TCP_ADDR` -- 一个 zookeeper 节点侦听端口 2181.
  当您启动容器此环境变量通常通过连接容器的运行设置 `zipkin-kafka` 如 "kafka".

例如, 添加调试日志记录, 组 JAVA_OPTS 如图我们的[docker-compose](docker-compose.yml)文件:

```yaml
- JAVA_OPTS=-Dlogging.level.zipkin=DEBUG -Dlogging.level.zipkin2=DEBUG
```

## 运行时用户

`openzipkin/zipkin` 和 `openzipkin/zipkin-slim` 镜像在没有登录名为 'zipkin' 带有 '/zipkin' 主目录的用户下运行.
由于这是一个 distroless 镜像, 你不会找到安装许多实用工具, 但你可以浏览内容中包含一个 shell:

```bash
$ docker run -it --rm --entrypoint /bin/sh openzipkin/zipkin
/zipkin $ ls
BOOT-INF  META-INF  org       run.sh
```

## 注意

如果使用外部 MySQL 服务器或镜像, 确保模式和其他参数匹配[文档](https://github.com/openzipkin/zipkin/tree/master/zipkin-storage/mysql-v1#applying-the-schema).

## 构建映像

打造 `openzipkin/zipkin`, 从资源库中的顶层, 运行:

```bash
$ docker build -t openzipkin/zipkin:test -f docker/Dockerfile .
```

如果你想 slim 分发替代, 运行:

```bash
$ docker build -t openzipkin/zipkin-slim:test -f docker/Dockerfile . --target zipkin-slim
```
