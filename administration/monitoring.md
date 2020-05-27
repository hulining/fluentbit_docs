---
description: 从 Fluent Bit 收集指标
---

# 监控

Fluent Bit 带有内置的 HTTP 服务，可用于查询内部信息并监控每个正在运行的插件的指标。

监控接口可以轻松地与 Prometheus 集成，因为我们支持 Prometheus 原生格式。

## 快速开始 <a id="getting_started"></a>

首先，第一步是修改配置文件以启用 HTTP Server:

```text
[SERVICE]
    HTTP_Server  On
    HTTP_Listen  0.0.0.0
    HTTP_PORT    2020

[INPUT]
    Name cpu

[OUTPUT]
    Name  stdout
    Match *
```

上面的配置代码片段将指示 Fluent Bit 在 TCP/2020 端口上启动 HTTP Server 并在所有网络接口上进行监听:

```text
$ bin/fluent-bit -c fluent-bit.conf
Fluent Bit v1.4.0
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2020/03/10 19:08:24] [ info] [engine] started
[2020/03/10 19:08:24] [ info] [http_server] listen iface=0.0.0.0 tcp_port=2020
```

现在使用简单 **curl** 命令就可以收集一些信息:

```text
$ curl -s http://127.0.0.1:2020 | jq
{
  "fluent-bit": {
    "version": "0.13.0",
    "edition": "Community",
    "flags": [
      "FLB_HAVE_TLS",
      "FLB_HAVE_METRICS",
      "FLB_HAVE_SQLDB",
      "FLB_HAVE_TRACE",
      "FLB_HAVE_HTTP_SERVER",
      "FLB_HAVE_FLUSH_LIBCO",
      "FLB_HAVE_SYSTEMD",
      "FLB_HAVE_VALGRIND",
      "FLB_HAVE_FORK",
      "FLB_HAVE_PROXY_GO",
      "FLB_HAVE_REGEX",
      "FLB_HAVE_C_TLS",
      "FLB_HAVE_SETJMP",
      "FLB_HAVE_ACCEPT4",
      "FLB_HAVE_INOTIFY"
    ]
  }
}
```

请注意，我们正在将 _curl_ 命令输出发送到 _jq_ 程序，这有助于从终端读取 JSON 数据。Fluent Bit 并非旨在进行 JSON 优雅打印。

## REST API Interface <a id="rest_api"></a>

Fluent Bit 公开了许多有用的监控接口，从 Fluent Bit v0.14 开始，以下端点可用:

| URI | Description | Data Format |
| :--- | :--- | :--- |
| / | Fluent Bit 构建信息 | JSON |
| /api/v1/uptime | 以秒和可读的格式获取正常运行的时间 | JSON |
| /api/v1/metrics | 已加载插件的内部指标 | JSON |
| /api/v1/metrics/prometheus | 可以被 Prometheus 服务获取已加载插件的内部指标 | Prometheus Text 0.0.4 |

## Uptime Example

使用以下命令查询服务正常运行时间:

```text
$ curl -s http://127.0.0.1:2020/api/v1/uptime | jq
```

它应该打印类似如下的输出:

```javascript
{
  "uptime_sec": 8950000,
  "uptime_hr": "Fluent Bit has been running:  103 days, 14 hours, 6 minutes and 40 seconds"
}
```

## Metrics Examples

使用以下命令查询 JSON 格式的内部指标:

```bash
$ curl -s http://127.0.0.1:2020/api/v1/metrics | jq
```

it should print a similar output like this:

```javascript
{
  "input": {
    "cpu.0": {
      "records": 8,
      "bytes": 2536
    }
  },
  "output": {
    "stdout.0": {
      "proc_records": 5,
      "proc_bytes": 1585,
      "errors": 0,
      "retries": 0,
      "retries_failed": 0
    }
  }
}
```

#### Metrics in Prometheus format

查询 Prometheus Text 0.0.4 格式的内部指标:

```bash
$ curl -s http://127.0.0.1:2020/api/v1/metrics/prometheus
```

这次，相同的指标将以 Prometheus 格式输出，而不是 JSON

```text
fluentbit_input_records_total{name="cpu.0"} 57 1509150350542
fluentbit_input_bytes_total{name="cpu.0"} 18069 1509150350542
fluentbit_output_proc_records_total{name="stdout.0"} 54 1509150350542
fluentbit_output_proc_bytes_total{name="stdout.0"} 17118 1509150350542
fluentbit_output_errors_total{name="stdout.0"} 0 1509150350542
fluentbit_output_retries_total{name="stdout.0"} 0 1509150350542
fluentbit_output_retries_failed_total{name="stdout.0"} 0 1509150350542
```

### 配置别名 <a id="configuring-aliases"></a>

默认情况下，在运行时配置的插件会以 _`plugin_name.ID`_ 的格式获取内部名称。出于监控目的，如果配置了多个相同类型的插件，可能会造成混淆。为了区别，每个配置的输入或输出部分都可以使用_别名_，它将用作指标的父名称。

下面的示例使用 [CPU](../pipline/inputs/cpu.md) 输入插件的 INPUT 部分设置别名:

```text
[SERVICE]
    HTTP_Server  On
    HTTP_Listen  0.0.0.0
    HTTP_PORT    2020

[INPUT]
    Name  cpu
    Alias server1_cpu

[OUTPUT]
    Name  stdout
    Alias raw_output
    Match *
```

当查询指标时，我们获取到了设置的别名而不是插件的名称:

```javascript
{
  "input": {
    "server1_cpu": {
      "records": 8,
      "bytes": 2536
    }
  },
  "output": {
    "raw_output": {
      "proc_records": 5,
      "proc_bytes": 1585,
      "errors": 0,
      "retries": 0,
      "retries_failed": 0
    }
  }
}
```

