# Elasticsearch

**es** 输出插件可将您的记录输出到 [Elasticsearch](http://www.elastic.co) 数据库中。以下说明假定您在您的环境中运行了具有完全可操作的 Elasticsearch 服务。

## Configuration Parameters

| Key | Description | default |
| :--- | :--- | :--- |
| Host | 目标 ES 实例 IP 地址或主机名 | 127.0.0.1 |
| Port | 目标 ES 实例的 TCP 端口 | 9200 |
| Path | ES 在 HTTP 查询路径 `/_bulk` 上接受新数据。但也可以在反向代理的子路径提供 ES 服务。该选项在 Bluent Bit 侧定义了这种路径。它在索引 HTTP POST URI 中添加路径前缀 | 空字符串 |
| Buffer\_Size | 指定用于从 Elasticsearch HTTP 服务读取响应的缓冲区大小。此选项对于需要读取完整响应的调试目的很有用，请注意响应大小取决于插入的记录数而增加。要设置无限数量的内存，请将此值设置为**False**，否则该值必须符合[单位](../../administration/configuring-fluent-bit/unit-sizes.md)中定义的规范 | 4KB |
| Pipeline | 较新版本的 Elasticsearch 允许设置称为 pipeline 的过滤器。出于性能原因，强烈建议在 Fluent Bit 端进行解析和过滤，避免使用 pipeline |  |
| AWS\_Auth | Enable AWS Sigv4 Authentication for Amazon ElasticSearch Service | Off |
| AWS\_Region | Specify the AWS region for Amazon ElasticSearch Service |  |
| HTTP\_User | 可选的用于 Elastic X-pack 访问的用户 |  |
| HTTP\_Passwd | `HTTP_User` 中定义用户的密码 |  |
| Index | Index 索引名称 | fluentbit |
| Type | Type 类型名称 | flb\_type |
| Logstash\_Format | 启用 Logstash 格式兼容性。此选项采用布尔值: True/False，On/Off | Off |
| Logstash\_Prefix | 启用 `Logstash_Format` 后，索引名称由前缀和日期组成。例如: 如果 `Logstash_Prefix` 等于 "mydata"，则索引将变为 "mydata-YYYY.MM.DD"。附加的最后一个字符串是生成数据的日期 | logstash |
| Logstash\_DateFormat | 指定时间格式\(基于 [strftime](http://man7.org/linux/man-pages/man3/strftime.3.html)\) 以生成索引名称的第二部分 | %Y.%m.%d |
| Time\_Key | 启用 `Logstash_Format` 后，每条记录将获得一个新的 timestamp 字段。`Time_Key` 属性定义该字段的名称 | @timestamp |
| Time\_Key\_Format | 启用 `Logstash_Format` 后，此属性定义时间戳格式 | %Y-%m-%dT%H:%M:%S |
| Include\_Tag\_Key | 是否将记录的标签附加到记录中 | Off |
| Tag\_Key | 启用 `Include_Tag_Key` 后，此属性定义附加到记录中的标签的键名 | \_flb-key |
| Generate\_ID | 启用后，自动为记录生成 `_id`。这样可以防止向 ES  输出时出现重复记录 | Off |
| Replace\_Dots | 启用后，用下划线替换字段的点，在 Elasticsearch 2.0-2.3 需要此配置 | Off |
| Trace\_Output | 启用后，打印 ES API 调用到标准输出\(仅用于 diag\) | Off |
| Current\_Time\_Index | 使用当前时间来生成索引而不使用消息记录 | Off |
| Logstash\_Prefix\_Key | 当配置此选线时: 将查找属于该键的记录中的值，并覆盖 `Logstash_Prefix` 以生成索引。如果在记录中未找到键/值，则 `Logstash_Prefix` 选项将用作备选。不支持嵌套键\(如果需要，您可以使用 Nest 过滤器插件删除嵌套\) |  |

> 如果不是很了解 Elastic，`index`\(索引\) 和 `type`\(类型\) 参数可能会令人困惑，如果您以前使用过关系型数据库，则它们可以比作 `database`\(数据库\) 和 `table`\(表\) 概念。另请参见下面的 [FAQ](elasticsearch.md#faq)。

### TLS / SSL

Elasticsearch 输出插件支持 TLS/SSL，有关可用属性和配置的详细信息，请参阅 [TLS/SSL](tcp-and-tls.md) 部分。

## Getting Started

要将记录输出到 Elasticsearch 服务中，您可以从命令行或通过配置文件运行插件:

### Command Line

**es** 插件可以通过两种方式从命令行读取参数，通过 **-p** 参数或直接通过服务 URI 进行设置。URI 格式如下:

```text
es://host:port/index/type
```

使用指定的格式，您可以通过以下方式启动 Fluent Bit:

```text
$ fluent-bit -i cpu -t cpu -o es://192.168.2.3:9200/my_index/my_type \
    -o stdout -m '*'
```

这与如下命令行是类似的:

```text
$ fluent-bit -i cpu -t cpu -o es -p Host=192.168.2.3 -p Port=9200 \
    -p Index=my_index -p Type=my_type -o stdout -m '*'
```

### Configuration File

在您的主配置文件中，添加如下 _Input_ 和 _Output_ 配置段:

```python
[INPUT]
    Name  cpu
    Tag   cpu

[OUTPUT]
    Name  es
    Match *
    Host  192.168.2.3
    Port  9200
    Index my_index
    Type  my_type
```

## About Elasticsearch field names

某些输入插件可能会生成字段名称中包含点的消息，从 Elasticsearch 2.0 不再允许这样做，因此当前 **es** 插件将其替换为下划线，例如:

```text
{"cpu0.p_cpu"=>17.000000}
```

变为

```text
{"cpu0_p_cpu"=>17.000000}
```

## FAQ

### ES 拒绝请求并提示 "the final mapping would have more than 1 type" <a id="faq-multiple-types"></a>

从 Elasticsearch 6.0 开始，您不能在单个索引中创建多个类型。这意味着您无法再按以下方式设置配置:

```text
[OUTPUT]
    Name  es
    Match foo.*
    Index search
    Type  type1

[OUTPUT]
    Name  es
    Match bar.*
    Index search
    Type  type2
```

如果您看到类似如下的错误消息，则需要修改配置，从而在每个索引上使用单一的类型\(Type\)

> Rejecting mapping update to \[search\] as the final mapping would have more than 1 type

有关详细信息，请阅读[关于该问题的官方博客文章](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/removal-of-types.html).

### Fluent Bit + Amazon Elasticsearch

Amazon ElasticSearch Service 添加了必须使用 AWS Sigv4 对 HTTP 请求进行签名的安全层。Fluent Bit v1.4 引入了对Amazon ElasticSearch Service 的实验性支持。

要使用 Amazon ElasticSearch Service，您**必须**将凭证指定为环境变量:

```text
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_SESSION_TOKEN="your-session-token"
```

虽然通常将 AWS 相关凭据设置为环境变量是安全的，但最佳实践是从标准 AWS 来源之一获取凭据\(如 [Amazon EKS IAM Role for a Service Account](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)\)。因此，此功能可能不适用于生产环境。Fluent Bit 和 AWS 正在共同努力以在 Fluent Bit v1.5 中提供对所有标准 AWS 凭证来源的全面支持。

配置示例:

```text
[OUTPUT]
    Name  es
    Match *
    Host  vpc-test-domain-ke7thhzoo7jawsrhmm6mb7ite7y.us-west-2.es.amazonaws.com
    Port  443
    Index my_index
    Type  my_type
    AWS_Auth On
    AWS_Region us-west-2
    tls     On
```

注意，`Port` 设置为 `443`，并启用了 `tls`。

如果此功能尚未满足您的需求，则可以使用如下代理作为替代解决方法:

* [https://github.com/abutaha/aws-es-proxy](https://github.com/abutaha/aws-es-proxy)

可以在以下位置找到有关 AWS Sigv4 和 ElasticSearch 的更多详细信息:

* [https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-request-signing.html](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-request-signing.html)

