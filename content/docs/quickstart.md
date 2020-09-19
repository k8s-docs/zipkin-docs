---
title: "快速开始"
linkTitle: ""
weight: 1
type: "docs"
description: >
  在这个部分我们将通过建立和启动一个 Zipkin 实例对检出 Zipkin 本地.
  有三个选项: 运用 Java, Docker 要么 从源代码运行.
---

If you are familiar with Docker, this is the preferred method to start. If you are unfamiliar with Docker, try running via Java or from source.

Regardless of how you start Zipkin, browse to http://your_host:9411 to find traces!

## Docker

[Docker Zipkin](https://github.com/openzipkin/docker-zipkin) 项目可构建 Docker 镜像, 提供脚本和一个 [docker-compose.yml](https://github.com/openzipkin/docker-zipkin/blob/master/docker-compose.yml) 用于启动预建的镜像.
最快的启动是直接运行最新的镜像:

```sh
docker run -d -p 9411:9411 openzipkin/zipkin
```

## Java

If you have Java 8 or higher installed, the quickest way to get started is to fetch the latest release as a self-contained executable jar:

```sh
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar
```

## 从源代码运行

Zipkin can be run from source if you are developing new features. To achieve this, you’ll need to get Zipkin’s source and build it.

```sh
# get the latest source

git clone https://github.com/openzipkin/zipkin
cd zipkin

# Build the server and also make its dependencies

./mvnw -DskipTests --also-make -pl zipkin-server clean install

# Run the server

java -jar ./zipkin-server/target/zipkin-server-\*exec.jar
```

在 gitter 上挺住和和我们社交, 如果你最终做一些有趣的事情!
