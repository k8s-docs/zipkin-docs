---
title: "架构"
linkTitle: ""
weight: 2
type: "docs"
description: >
  跟踪器存活于你的应用程序并记录有关操作所发生的定时和元数据.
---

## 架构概述

他们往往仪器库, 所以它们的使用对用户透明.
例如, 仪表化网络服务器记录 当它接收到一个请求和当它发送的响应.
收集的跟踪数据被称为 Span.

仪表在生产里被写入安全并有小的开销.
为此原因, 它们仅传播 IDs in-band, 告诉接收机这有正在进行中的追踪.
已完成 spans 被反馈给 Zipkin out-of-band, 类似应用程序如何以异步方式报告度量.

例如, 当操作被跟踪它需要进行传出 HTTP 请求, 几个头加入到传播 IDs.
头不用于发送详细信息，如操作名称.

在装有仪表的应用中的部件 将数据发送到 Zipkin 被称为记者.
记者通过几种传输到一个发送跟踪数据到 Zipkin 收集器, 其使跟踪数据用久存储.
后来, 存储由所述 API 查询以提供数据到 UI.

下面是描述此流的图:

![Zipkin architecture](https://zipkin.io/public/img/architecture-1.png)

要查看是否有跟踪器或仪表库已经存在于您的平台, 看到我们的名单.

## 示例流程

正如概览中所提到, 标识发 in-band 和细节被发送 out-of-band 至 Zipkin.
在这两种情况下, 跟踪检测负责创建有效的跟踪和正确使它们.
例如, 跟踪器确保奇偶 它发送的数据 in-band (downstream) 和 out-of-band 之间 (异步到 Zipkin).

在此处，用户代码调用资源的 HTTP 跟踪的例子序列 /foo.
这导致单 span, 用户代码接收 HTTP 响应后异步发送到 Zipkin.

```
┌─────────────┐ ┌───────────────────────┐  ┌─────────────┐  ┌──────────────────┐
│ User Code   │ │ Trace Instrumentation │  │ Http Client │  │ Zipkin Collector │
└─────────────┘ └───────────────────────┘  └─────────────┘  └──────────────────┘
       │                 │                         │                 │
           ┌─────────┐
       │ ──┤GET /foo ├─▶ │ ────┐                   │                 │
           └─────────┘         │ record tags
       │                 │ ◀───┘                   │                 │
                           ────┐
       │                 │     │ add trace headers │                 │
                           ◀───┘
       │                 │ ────┐                   │                 │
                               │ record timestamp
       │                 │ ◀───┘                   │                 │
                             ┌─────────────────┐
       │                 │ ──┤GET /foo         ├─▶ │                 │
                             │X-B3-TraceId: aa │     ────┐
       │                 │   │X-B3-SpanId: 6b  │   │     │           │
                             └─────────────────┘         │ invoke
       │                 │                         │     │ request   │
                                                         │
       │                 │                         │     │           │
                                 ┌────────┐          ◀───┘
       │                 │ ◀─────┤200 OK  ├─────── │                 │
                           ────┐ └────────┘
       │                 │     │ record duration   │                 │
            ┌────────┐     ◀───┘
       │ ◀──┤200 OK  ├── │                         │                 │
            └────────┘       ┌────────────────────────────────┐
       │                 │ ──┤ asynchronously report span     ├────▶ │
                             │                                │
                             │{                               │
                             │  "traceId": "aa",              │
                             │  "id": "6b",                   │
                             │  "name": "get",                │
                             │  "timestamp": 1483945573944000,│
                             │  "duration": 386000,           │
                             │  "annotations": [              │
                             │--snip--                        │
                             └────────────────────────────────┘
```

跟踪检测报告跨度异步防止延迟或破坏用户代码与跟踪系统的延迟或失败.

## 传输

Spans sent by the instrumented library must be transported from the services being traced to Zipkin collectors.
There are three primary transports: HTTP, Kafka and Scribe.

## 组件

有 4 个组件，使基普金:

- collector
- storage
- search
- web UI

### Zipkin Collector

Once the trace data arrives at the Zipkin collector daemon, it is validated, stored, and indexed for lookups by the Zipkin collector.

### 存储

Zipkin was initially built to store data on Cassandra since Cassandra is scalable, has a flexible schema, and is heavily used within Twitter.
However, we made this component pluggable.
In addition to Cassandra, we natively support ElasticSearch and MySQL.
Other back-ends might be offered as third party extensions.

### Zipkin 查询服务

Once the data is stored and indexed, we need a way to extract it.
The query daemon provides a simple JSON API for finding and retrieving traces.
The primary consumer of this API is the Web UI.

### Web UI

We created a GUI that presents a nice interface for viewing traces.
The web UI provides a method for viewing traces based on service, time, and annotations.
Note: there is no built-in authentication in the UI!
