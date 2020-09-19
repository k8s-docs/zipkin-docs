---
title: "数据模型"
linkTitle: ""
weight: 6
type: "docs"
description: >
  请注意，此页面是出于过期. 请查看 [Zipkin Api](https://zipkin.io/zipkin-api/#/default/post_spans) 文档 直到该页面被更新阐述了模型的字段.
---

为了说明的 Zipkin 显示的跟踪数据, 让我们将它与 Zipkin 数据模型里面等价信息.
通过比较这些, 我们看到

- 入站和出站请求在不同的 span
- 包括 CS 的 spans 可以记录他们去了哪里的 sa 注释
  - 这有助于 当目的地协议不使 Zipkin 仪表, 如 MySQL.

第一, 我们看到一个在 Zipkin 跟踪查看器里面跟踪如图所示 :

![Zipkin Screenshot](https://zipkin.io/public/img/json_zipkin_screenshot.png)

而在 Zipkin 数据模型相同的跟踪:

```json
[
  {
    "traceId": "5982fe77008310cc80f1da5e10147517",
    "name": "get",
    "id": "bd7a977555f6b982",
    "timestamp": 1458702548467000,
    "duration": 386000,
    "localEndpoint": {
      "serviceName": "zipkin-query",
      "ipv4": "192.168.1.2",
      "port": 9411
    },
    "annotations": [
      {
        "timestamp": 1458702548467000,
        "value": "sr"
      },
      {
        "timestamp": 1458702548853000,
        "value": "ss"
      }
    ]
  },
  {
    "traceId": "5982fe77008310cc80f1da5e10147517",
    "name": "get-traces",
    "id": "ebf33e1a81dc6f71",
    "parentId": "bd7a977555f6b982",
    "timestamp": 1458702548478000,
    "duration": 354374,
    "localEndpoint": {
      "serviceName": "zipkin-query",
      "ipv4": "192.168.1.2",
      "port": 9411
    },
    "tags": {
      "lc": "JDBCSpanStore",
      "request": "QueryRequest{serviceName=zipkin-query, spanName=null, annotations=[], binaryAnnotations={}, minDuration=null, maxDuration=null, endTs=1458702548478, lookback=86400000, limit=1}"
    }
  },
  {
    "traceId": "5982fe77008310cc80f1da5e10147517",
    "name": "query",
    "id": "be2d01e33cc78d97",
    "parentId": "ebf33e1a81dc6f71",
    "timestamp": 1458702548786000,
    "duration": 13000,
    "localEndpoint": {
      "serviceName": "zipkin-query",
      "ipv4": "192.168.1.2",
      "port": 9411
    },
    "remoteEndpoint": {
      "serviceName": "spanstore-jdbc",
      "ipv4": "127.0.0.1",
      "port": 3306
    },
    "annotations": [
      {
        "timestamp": 1458702548786000,
        "value": "cs"
      },
      {
        "timestamp": 1458702548799000,
        "value": "cr"
      }
    ],
    "tags": {
      "jdbc.query": "select distinct `zipkin_spans`.`trace_id` from `zipkin_spans` join `zipkin_annotations` on (`zipkin_spans`.`trace_id` = `zipkin_annotations`.`trace_id` and `zipkin_spans`.`id` = `zipkin_annotations`.`span_id`) where (`zipkin_annotations`.`endpoint_service_name` = ? and `zipkin_spans`.`start_ts` between ? and ?) order by `zipkin_spans`.`start_ts` desc limit ?",
      "sa": "true"
    }
  },
  {
    "traceId": "5982fe77008310cc80f1da5e10147517",
    "name": "query",
    "id": "13038c5fee5a2f2e",
    "parentId": "ebf33e1a81dc6f71",
    "timestamp": 1458702548817000,
    "duration": 1000,
    "localEndpoint": {
      "serviceName": "zipkin-query",
      "ipv4": "192.168.1.2",
      "port": 9411
    },
    "remoteEndpoint": {
      "serviceName": "spanstore-jdbc",
      "ipv4": "127.0.0.1",
      "port": 3306
    },
    "annotations": [
      {
        "timestamp": 1458702548817000,
        "value": "cs"
      },
      {
        "timestamp": 1458702548818000,
        "value": "cr"
      }
    ],
    "tags": {
      "jdbc.query": "select `zipkin_spans`.`trace_id`, `zipkin_spans`.`id`, `zipkin_spans`.`name`, `zipkin_spans`.`parent_id`, `zipkin_spans`.`debug`, `zipkin_spans`.`start_ts`, `zipkin_spans`.`duration` from `zipkin_spans` where `zipkin_spans`.`trace_id` in (?)",
      "sa": "true"
    }
  },
  {
    "traceId": "5982fe77008310cc80f1da5e10147517",
    "name": "query",
    "id": "37ee55f3d3a94336",
    "parentId": "ebf33e1a81dc6f71",
    "timestamp": 1458702548827000,
    "duration": 2000,
    "localEndpoint": {
      "serviceName": "zipkin-query",
      "ipv4": "192.168.1.2",
      "port": 9411
    },
    "remoteEndpoint": {
      "serviceName": "spanstore-jdbc",
      "ipv4": "127.0.0.1",
      "port": 3306
    },
    "annotations": [
      {
        "timestamp": 1458702548827000,
        "value": "cs"
      },
      {
        "timestamp": 1458702548829000,
        "value": "cr"
      }
    ],
    "tags": {
      "jdbc.query": "select `zipkin_annotations`.`trace_id`, `zipkin_annotations`.`span_id`, `zipkin_annotations`.`a_key`, `zipkin_annotations`.`a_value`, `zipkin_annotations`.`a_type`, `zipkin_annotations`.`a_timestamp`, `zipkin_annotations`.`endpoint_ipv4`, `zipkin_annotations`.`endpoint_port`, `zipkin_annotations`.`endpoint_service_name` from `zipkin_annotations` where `zipkin_annotations`.`trace_id` in (?) order by `zipkin_annotations`.`a_timestamp` asc, `zipkin_annotations`.`a_key` asc",
      "sa": "true"
    }
  }
]
```
