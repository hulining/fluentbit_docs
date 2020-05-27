# 调度与重试

[Fluent Bit](https://fluentbit.io) 包含一个引擎，可以协调从输入插件提取数据并调用 _Scheduler\(调度器\)_ 决定何时通过一个或多个输出插件刷新数据。调度器以固定的时间刷新新数据，并进行调度重试。

当调用输出插件刷新某些数据时，在处理完该数据后，它以三种可能的返回状态通知引擎:

* OK
* Retry
* Error

如果返回状态为 **OK**，意味着它能够成功处理和刷新数据，如果返回 **Error** 状态，则表示发生了不可恢复的错误，引擎不会尝试再次刷新该数据。如果请求 **Retry**，_引擎_将要求_调度器_尝试刷新数据，调度器将决定刷新数据的等待时长。

## 配置重试 <a id="configuring-retries"></a>

调度器提供了一个简单的配置项，称为 **Retry\_Limit**，它可以在每个输出配置段中配置。此选项允许禁用重试或尝试重试 N 次后丢弃数据:

|  | Value | Description |
| :--- | :--- | :--- |
| Retry\_Limit | N | 整数值，用于设置允许的最大重试次数。N 必须大于等于 1\(默认为 2\) |
| Retry\_Limit | False | 不限制重试次数 |

### 示例 <a id="example"></a>

以下示例包含两个输出配置，其中 HTTP 输出插件可重试无限次，而 Elasticsearch 插件具有 5 次重试限制:

```text
[OUTPUT]
    Name        http
    Host        192.168.5.6
    Port        8080
    Retry_Limit False

[OUTPUT]
    Name            es
    Host            192.168.5.20
    Port            9200
    Logstash_Format On
    Retry_Limit     5
```

