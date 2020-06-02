# Grep

**grep** 过滤插件允许根据正则表达式模式匹配或排除特定记录。

## 配置参数 <a id="configuration-parameters"></a>

该插件支持如下配置参数:

| Key | Value Format | Description |
| :--- | :--- | :--- |
| Regex | FIELD REGEX | 保留与正则表达式匹配的字段的记录 |
| Exclude | FIELD REGEX | 排除与正则表达式匹配的字段的记录 |

## 快速开始 <a id="getting-started"></a>

要开始过滤数据记录，您可以从命令行或通过配置文件运行过滤器。如下示例假定您有一个名为 **lines.txt** 的文件，其中包含以下内容:

```text
aaa
aab
bbb
ccc
ddd
eee
fff
ggg
```

### 命令行 <a id="command-line"></a>

> 注意: 使用命令行模式时，需要特别注意正则表达式的正确性。建议使用配置文件。

如下命令将加载 **tail** 插件并读取 **lines.txt** 文件的内容。然后 **grep** 过滤器会将正则表达式规则应用于 **log** 字段\(由 tail 插件创建\)，并且仅保留以 **aa** 开头的记录:

```text
$ bin/fluent-bit -i tail -p 'path=lines.txt' -F grep -p 'regex=log aa' -m '*' -o stdout
```

### 配置文件 <a id="configuration-file"></a>

```python
[INPUT]
    Name   tail
    Path   lines.txt

[FILTER]
    Name   grep
    Match  *
    Regex  log aa

[OUTPUT]
    Name   stdout
    Match  *
```

该过滤器允许按顺序应用多个规则，您可以根据需要设置多个 `Regex` 和 `Exclude` 配置项。

### 嵌套字段示例 <a id="nested-fields-example"></a>

到目前为止，不支持嵌套字段。如果您有类似以下格式的记录:

```javascript
{
    "kubernetes": {
        "pod_name": "myapp-0",
        "namespace_name": "default",
        "pod_id": "216cd7ae-1c7e-11e8-bb40-000c298df552",
        "labels": {
            "app": "myapp"
        },
        "host": "minikube",
        "container_name": "myapp",
        "docker_id": "370face382c7603fdd309d8c6aaaf434fd98b92421ce7c7c8aafe7697d4aa362"
    }
}
```

如果您想排除与给定嵌套字段\(如 `kubernetes.labels.app`\)，则可以与 [nest](nest.md) 过滤插件一起使用。如下是一个排除与 `kubernetes.labels.app: myapp` 匹配的记录的示例:

```python
[FILTER]
    Name         nest
    Match        *
    Operation    lift
    Nested_under kubernetes

[FILTER]
    Name         nest
    Match        *
    Operation    lift
    Nested_under labels

[FILTER]
    Name    grep
    Match   *
    Exclude app myapp
```

