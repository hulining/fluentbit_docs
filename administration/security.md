# 安全性

Fluent Bit 提供了对_传输层安全_\(TLS\)及_安全套接字层_\(SSL\)的集成支持。在本节中，我们仅将两种实现都称为 TLS。

每个需要执行网络 I/O 的输出插件都可以选择启用 TLS 并配置其行为。下表描述了可用的属性:

Each output plugin that requires to perform Network I/O can optionally enable TLS and configure the behavior. The following table describes the properties available:

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| tls | 是否启用 TLS | Off |
| tls.verify | 强制证书验证 | On |
| tls.debug | 设置 TLS 调试级别。它接受如下值: 0\(No debug\), 1 \(Error\), 2 \(State change\), 3 \(Informational\), 4 Verbose | 1 |
| tls.ca\_file | CA 证书文件的绝对路径 |  |
| tls.ca\_path | 扫描 CA 证书文件的绝对路径 |  |
| tls.crt\_file | 证书文件的绝对路径 |  |
| tls.key\_file | 私钥文件的绝对路径 |  |
| tls.key\_passwd | 私钥文件的可选密码 |  |
| tls.vhost | 用于 TLS SNI 的主机名 |  |

可以在配置文件中启用上表列出的属性，特别是在每个输出插件部分，或者直接通过命令行启用。

以下输出插件可以使用 TLS 功能：

* [Azure](../pipeline/outputs/azure.md)
* [BigQuery](../pipeline/outputs/bigquery.md)
* [Datadog](../pipeline/outputs/datadog.md)
* [Elasticsearch](../pipeline/outputs/elasticsearch.md)
* [Forward](../pipeline/inputs/forward.md)
* [GELF](../pipeline/outputs/gelf.md)
* [HTTP](../pipeline/outputs/http.md)
* [InfluxDB](../pipeline/outputs/influxdb.md)
* [Kafka REST Proxy](../pipeline/outputs/kafka-rest-proxy.md)
* Slack
* [Splunk](../pipeline/outputs/splunk.md)
* [Stackdriver](../pipeline/outputs/stackdriver.md)
* [TCP & TLS](../pipeline/outputs/tcp-and-tls.md)
* [Treasure Data](../pipeline/outputs/treasure-data.md)

此外，另一些插件实现了部分支持 TLS，这意味着包含受限制的配置。

* [Kubernetes Filter](../pipeline/filters/kubernetes.md)

## 示例: 在 HTTP 输出插件上启用 TLS <a id="example-enable-tls-on-http-output"></a>

默认情况下，HTTP 输出插件使用普通的 TCP，可以通过以下命令从命令行启用 TLS:

```text
$ fluent-bit -i cpu -t cpu -o http://192.168.2.3:80/something \
    -p tls=on         \
    -p tls.verify=off \
    -m '*'
```

在如上示例的命令行参数中，启用了 _`tls`_ 和 _`tls.verify`_ 两个属性\(强烈建议始终将是否验证保持为 ON\)。

使用如下配置文件可以实现相同的效果:

```text
[INPUT]
    Name  cpu
    Tag   cpu

[OUTPUT]
    Name       http
    Match      *
    Host       192.168.2.3
    Port       80
    URI        /something
    tls        On
    tls.verify Off
```

## 技巧和窍门 <a id="tips-and-tricks"></a>

### 使用 TLS 连接到虚拟服务器 <a id="connect-to-virtual-servers-using-tls"></a>

Fluent Bit 支持 [TLS 服务名称指示](https://en.wikipedia.org/wiki/Server_Name_Indication)。如果您在单个 IP 地址\(也称为虚拟主机\)上提供多个主机名，则可以使用 `tls.vhost` 连接到指定的主机名。

```text
[INPUT]
    Name  cpu
    Tag   cpu

[OUTPUT]
    Name        forward
    Match       *
    Host        192.168.10.100
    Port        24224
    tls         On
    tls.verify  On
    tls.ca_file /etc/certs/fluent.crt
    tls.vhost   fluent.example.com
```

