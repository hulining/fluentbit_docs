# Kafka

**kafka** 输出插件可以将您的记录输出到 [Apache Kafka](https://kafka.apache.org/) 服务中。该插件使用了官方的 [librdkafka C 库](https://github.com/edenhill/librdkafka)\(内置依赖库\)。

## Configuration Parameters

| Key | Description | default |
| :--- | :--- | :--- |
| Format | 指定数据格式，可用选项为 `json`, `msgpack`. | json |
| Message\_Key | 可选的存储消息的键 |  |
| Message\_Key\_Field | 如果设置，记录中的 `Message_Key_Field` 值将指定消息的键。如果没有设置或没有找到指定的键，则将使用 `Message_Key`\(如果设置了\) 作为消息的键 |  |
| Timestamp\_Key | 设置存储记录时间戳的键 | @timestamp |
| Timestamp\_Format | 'iso8601' 或 'double' | double |
| Brokers | Kafka Brokers 的多个列表中的一个，例如: 192.168.1.3:9092, 192.168.1.4:9092. |  |
| Topics | Fluent Bit 将消息发送到 Kafka 的单个 Topic 或以逗号分割的 Topic 列表，如果仅设置一个T opic，则该 Topic 将用于所有记录。如果存在多个主题，则使用记录中 `Topic_Key` 指定字段值 | fluent-bit |
| Topic\_Key | 如果存在多个主题，则记录中 `Topic_Key` 指定字段值将指定要使用的主题。例如: 如果 `Topic_Key` 设置为 `router`且记录为 `{"key1": 123, "router": "route\_2"}`， Fluent Bit 将使用 `route_2` 作为 kafka 的 topic.请注意，如果`Topics` 列表中没有 `Topic_Key` 的值，则默认情况下，将使用 `Topics` 列表中的第一个值 |  |
| rdkafka.{property} | `{property}` 可以是 [librdkafka properties](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md) 任何属性 |  |

> 将 `rdkafka.log.connection.close` 设置为 `false` 且将 `rdkafka.request.required.acks` 设置为 1 是 librdfkafka properties 推荐设置的示例。

## Getting Started

要将记录输出到 Apache Kafka，您可以从命令行或通过配置文件运行插件:

### Command Line

**kafka** 插件可以通过 **-p** 参数从命令行读取参数:

```text
$ fluent-bit -i cpu -o kafka -p brokers=192.168.1.3:9092 -p topics=test
```

### Configuration File

在您的主配置文件中，添加如下 _Input_ 和 _Output_ 配置段:

```text
[INPUT]
    Name  cpu

[OUTPUT]
    Name        kafka
    Match       *
    Brokers     192.168.1.3:9092
    Topics      test
```

