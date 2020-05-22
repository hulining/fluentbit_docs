# 升级说明

本文涵盖了有关从之前 Fluent Bit 版本升级的相关说明，旨在包含您必须知道的兼容性更改。

The following article cover the relevant notes for users upgrading from previous Fluent Bit versions. We aim to cover compatibility changes that you must be aware of.

有关每个发行版更改的更多详细信息，请参阅[官方发行说明](https://fluentbit.io/announcements/)

## Fluent Bit v1.4

如果从 Fluent Bit v1.3 进行升级，不需要做出改动。只需享受令人兴奋的新功能即可 :)

## Fluent Bit v1.3

如果您从 Fluent Bit v1.2 升级到 v1.3，不需要做出改动。如果要从旧版本升级，请查看下面的增量更改。

## Fluent Bit v1.2

### Docker, JSON, Parsers and Decoders

在 Fluent Bit v1.2，我们修复了 JSON 编码和解码相关的问题，因此当解析 Docker 日志时**不再需要**使用 decoders。新的 Docker 解析器如下所示:

```text
[PARSER]
    Name         docker
    Format       json
    Time_Key     time
    Time_Format  %Y-%m-%dT%H:%M:%S.%L
    Time_Keep    On
```

> 注意: 再次说明，不要再使用 decoders

### Kubernetes Filter

我们还对 Kubernetes 过滤器如何处理字符串化的 _`log`_ 消息进行了改进。如果启用了 _`Merge_Log`_ 选项，它将尝试将日志内容作为 JSON 映射进行处理，并将添加键到根映射中。

另外，我们修复并改进了名为 _`Merge_Log_Key`_ 的选项。如果成功合并日志，则所有的键都将打包在此选项指定的键下，说明配置如下:

```text
[FILTER]
    Name             Kubernetes
    Match            kube.*
    Kube_Tag_Prefix  kube.var.log.containers.
    Merge_Log        On
    Merge_Log_Key    log_processed
```

例如，如果原始日志内容为以下映射:

```javascript
{"key1": "val1", "key2": "val2"}
```

最终记录将组合如下:

```javascript
{
    "log": "{\"key1\": \"val1\", \"key2\": \"val2\"}",
    "log_processed": {
        "key1": "val1",
        "key2": "val2"
    }
}
```

## Fluent Bit v1.1

如果您是从 **Fluent Bit &lt;= 1.0.x** 升级，在升级到 **Fluent Bit v1.1** 系列版本时，您应考虑以下相关更改:

### Kubernetes Filter

我们引入了一个名为 _`Kube_Tag_Prefix`_ 的新配置属性，以帮助标记前缀解析并解决以前版本中出现的意外行为。

在 1.0.x 的版本迭代中，Tail 输入插件的一次提交改变了关于使用通配符进行扩展时生成标签的默认方式，从而破坏与其他服务的兼容性。参考如下配置示例:

```text
[INPUT]
    Name  tail
    Path  /var/log/containers/*.log
    Tag   kube.*
```

我们预期 Tag 将被扩展为:

```text
kube.var.log.containers.apache.log
```

但 1.0 系列中引入的更改仅从绝对路径切换为基本文件名:

```text
kube.apache.log
```

在 Fluent Bit v1.1 发行版中，我们恢复了这个默认行为，现在标签是由受监控文件的绝对路径组成的。

> 标签带有绝对路径与路由和灵活的配置有关，还有助于保持与 Fluentd 的兼容性。

Tail 输入插件中的行为切换影响 Kubernetes 过滤器的运行方式。正如您所知道的那样，当使用 Tail 输入作为数据源时，使用过滤器时需要执行来自文件名的本地元数据查找。现在，使用新增的 _`Kube_Tag_Prefix`_ 选项，您可以指定 Tail 输入插件中使用的前缀是什么。对于上面的配置示例，新配置将如下所示：

```text
[INPUT]
    Name  tail
    Path  /var/log/containers/*.log
    Tag   kube.*

[FILTER]
    Name             kubernetes
    Match            *
    Kube_Tag_Prefix  kube.var.log.containers.
```

正确的 _`Kube_Tag_Prefix`_ 值一定由在 Tail 输入插件中设置的标签前缀加上转换后的受监控目录(用点代替斜杠)组成。
