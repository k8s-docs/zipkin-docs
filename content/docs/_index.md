---
title: "Zipkin 文档"
linkTitle: "文档"
weight: 10
menu:
  main:
    weight: 10
---

Zipkin 是一个分布式追踪系统.
它可以帮助收集时序数据 需要在服务架构解决延迟问题.
功能包括收集和这些数据的查找.

如果你有一个日志文件跟踪 ID, 您可以直接跳转到它.
除此以外, 您可以基于属性查询如服务, 操作名称，标记和持续时间.
一些有趣的数据将为你被归纳, 如时间的服务花费的时间百分比, 不论是否操作失败.

![跟踪查看截图](https://zipkin.io/public/img/web-screenshot.png)

Zipkin UI 还提出了一种依赖关系图显示有多少通过每个应用程序追踪请求.
这可以用于确定总行为是有帮助 包括弃用服务错误的路径或电话.

![依赖图截图](https://zipkin.io/public/img/dependency-graph.png)

应用程序需要被装备跟踪数据报告给 Zipkin.
这通常意味着示踪剂或仪表库的配置.
通过 HTTP 或 Kafka 最流行的方式来报告数据到 Zipkin, 虽然许多其他选项存在, 如 Apache ActiveMQ, gRPC 和 RabbitMQ.
投放到 UI 的数据存储在内存中, 或用支持的后端持久保存如 Apache Cassandra 或 Elasticsearch.

## 下一步去哪里？

- 尝试 Zipkin, 看看我们的快速入门指南
- 看看你的平台有示踪剂或仪器库
- 查看服务器是否延长或替代是相关的网站.
- 加入 Zipkin Gitter 聊天频道
- 源代码在 GitHub 如 openzipkin/zipkin
- 问题也在 GitHub 上跟踪
