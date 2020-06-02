# HTTP

**http** 输出插件允许将您的记录输出到 HTTP 端点。目前，该功能非常基础，它以 [MessagePack](http://msgpack.org)\(或JSON\)格式的数据记录向 HTTP 端点发出 POST 请求。

## Configuration Parameters

| Key | Description | default |
| :--- | :--- | :--- |
| Host | 目标 HTTP 服务的主机名或 IP 地址 | 127.0.0.1 |
| HTTP\_User | 基本身份认证用户名 |  |
| HTTP\_Passwd | 基本身份认证密码 |  |
| Port | HTTP 服务的端口 | 80 |
| Proxy | 指定 HTTP 代理。该值的预期格式为 `http://host:port`。_https_ 协议的代理还不支持 |  |
| URI | 指定目标 Web 服务器 HTTP URI，如: /something | / |
| Format | 指定要在 HTTP 请求正文中使用的数据格式，默认使用 `msgpack`。其它支持的格式是 `json`,`json_stream`,`json_lines`,`gelf` | msgpack |
| header\_tag | 为原始的消息标签指定一个可选的 HTTP 请求头 |  |
| Header | 添加 HTTP 请求头的键值对。可以设置多个 |  |
| json\_date\_key | 在输出中指定日期字段的名称 | date |
| json\_date\_format | 指定日期格式。支持的格式为 `double`,`iso8601`\(如: _2018-05-30T09:39:52.000681Z_\) | double |
| gelf\_timestamp\_key | 为 `gelf` 格式的数据格式指定 `timestamp` 的键 |  |
| gelf\_host\_key | 为 `gelf` 格式的数据格式指定 `host` 的键 |  |
| gelf\_short\_messge\_key | 为 `gelf` 格式的数据格式指定 `short`messge 的键 |  |
| gelf\_full\_message\_key | 为 `gelf` 格式的数据格式指定 `full`messge 的键 |  |
| gelf\_level\_key | 为 `gelf` 格式的数据格式指定 `level` 的键 |  |

### TLS / SSL

**http** 输出插件支持 TLS/SSL，有关可用属性和配置的详细信息，请参阅 [TLS/SSL](tcp-and-tls.md#tls-configuration-parameters) 部分。

## Getting Started

要将记录输出到 HTTP 服务器，您可以从命令行或通过配置文件运行插件:

### Command Line

**http** 插件可以通过两种方式从命令行读取参数，通过 **-p** 参数或直接通过服务 URI 进行设置。URI 格式如下:

```text
http://host:port/something
```

使用指定的格式，您可以通过以下方式启动 Fluent Bit:

```text
$ fluent-bit -i cpu -t cpu -o http://192.168.2.3:80/something -m '*'
```

### Configuration File

在您的主配置文件中，添加如下 _Input_ 和 _Output_ 配置段:

```python
[INPUT]
    Name  cpu
    Tag   cpu

[OUTPUT]
    Name  http
    Match *
    Host  192.168.2.3
    Port  80
    URI   /something
```

默认情况下，URI 会成为输出记录的标签，原始标签值将被忽略。要保留标签，必须基于多个配置段并将其输出到不同的 URI。

我们还支持的另一种方式时在可配置的请求头中发送原始标签。接收方可以根据请求头字段执行所需的操作: 解析它并将其用作消息的标签。示例如下:

要配置此行为，请添加以下配置:

```text
[OUTPUT]
    Name  http
    Match *
    Host  192.168.2.3
    Port  80
    URI   /something
    Format json
    header_tag  FLUENT-TAG
```

如果您使用 Fluentd 作为数据接收器，则可以使用 `in_http` 和 `out_rewrite_tag_filter` 来使用此 HTTP 请求头。

```text
<source>
  @type http
  add_http_headers true
</source>

<match something>
  @type rewrite_tag_filter
  <rule>
    key HTTP_FLUENT_TAG
    pattern /^(.*)$/
    tag $1
  </rule>
</match>
```

请注意，我们使用自定义请求头覆盖了值为 URI 路径的标签。

#### Example : Add a header

```text
[OUTPUT]
    Name           http
    Match          *
    Host           127.0.0.1
    Port           9000
    Header         X-Key-A Value_A
    Header         X-Key-B Value_B
    URI            /something
```

#### Example : Sumo Logic HTTP Collector

Sumo Logic 建议的配置是使用 `iso8601` 格式时间戳的 `json_lines` 格式记录。`PrivateKey` 指定用于配置的 HTTP 收集器。

```text
[OUTPUT]
    Name             http
    Match            *
    Host             collectors.au.sumologic.com
    Port             443
    URI              /receiver/v1/http/[PrivateKey]
    Format           json_lines
    Json_date_key    timestamp
    Json_date_format iso8601
```

Sumo Logic 查询 [CPU](../../pipleline/inputs/cpu.md) 输入插件的示例\(需要 `iso8601` 日期格式的 `json_lines` 格式记录\)如下:

```text
_sourcecategory="my_fluent_bit"
| json "cpu_p" as cpu
| timeslice 1m
| max(cpu) as cpu group by _timeslice
```

