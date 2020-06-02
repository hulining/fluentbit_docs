# Kubernetes

**kubernetes** 过滤器插件允许使用元数据丰富您的日志文件。

当 Fluent Bit 作为 DaemonSet 部署在 Kubernetes 并配置为从容器\(使用 tail 或 systemd 输入插件\)读取日志时，此过滤器可以执行以下操作:

* 分析 Tag 标签并提取以下元数据:
  * Pod Name
  * Namespace
  * Container Name
  * Container ID
* 查询 Kubernetes API Server 以获取有关 Pod 的额外元数据:
  * Pod ID
  * Labels
  * Annotations

数据缓存在本地内存中，并附加到每个日志记录上。

## 配置参数 <a id="configuration-parameters"></a>

该插件支持以下配置参数:

| 配置项 | 描述 | 默认值 |
| :--- | :--- | :--- |
| Buffer\_Size | 从Kubernetes API Server 读取响应时，设置 HTTP 客户端的缓冲区大小。该值必须符合[单位](../../administration/configuring-fluent-bit/unit-sizes.md)规范 | 32k |
| Kube\_URL | API Server 服务端点 | https://kubernetes.default.svc:443 |
| Kube\_CA\_File | CA 证书文件 | /var/run/secrets/kubernetes.io/serviceaccount/ca.crt |
| Kube\_CA\_Path | 扫描证书文件的绝对路径 |  |
| Kube\_Token\_File | Token 文件 | /var/run/secrets/kubernetes.io/serviceaccount/token |
| Kube\_Tag\_Prefix | 当源记录来自 Tail 输入插件时，此选项指定 Tail 配置中使用的前缀 | kube.var.log.containers. |
| Merge\_Log | 启用后，它会检查 `log` 字段内容是否为 JSON 字符串映射，如果是，则将映射字段附加为日志结构的一部分 | Off |
| Merge\_Log\_Key | 启用 `Merge_Log` 时，设置结构化处理后，映射结构键的名称 |  |
| Merge\_Log\_Trim | 启用 `Merge_Log` 时，删除可能存在的 `\n` 或 `\r` 字段值 | On |
| Merge\_Parser | 可选的解析器名称，用于指定如何解析 `log` 键中包含的数据。一般用于开发或测试 |  |
| Keep\_Log | 当 `Keep_Log` 被禁用，一旦成功合并，`log` 字段将被删除\(`Merge_Log` 必须启用\) | On |
| tls.debug | 调试级别 0\(nothing\) 到 4 \(every detail\). | -1 |
| tls.verify | 启用后，在连接到 Kubernetes API Server 时打开证书验证 | On |
| Use\_Journal | 启用后，过滤器将以 Journald 格式读取日志 | Off |
| Regex\_Parser | 设置备用解析器以处理记录 Tag 并提取 pod\_name, pod\_name, namespace\_name, container\_name 和 docker\_id.解析器必须在 [parsers 文件](https://github.com/fluent/fluent-bit/blob/master/conf/parsers.conf)中注册\(参考 _filter-kube-test_ 解析器示例\). |  |
| K8S-Logging.Parser | 允许 Kubernetes Pod 建议预定义的解析器\(在 Kubernetes Annotations 部分中了解更多信息\) | Off |
| K8S-Logging.Exclude | 允许Kubernetes Pod 从日志处理器中排除其日志\(在 Kubernetes Annotations 部分中了解更多信息\). | Off |
| Labels | 是否在额外的元数据中包含 Kubernetes 资源标签信息 | On |
| Annotations | 是否在额外的元数据中包括 Kubernetes 资源信息 | On |
| Kube\_meta\_preload\_cache\_dir | 如果设置，则可以从该目录中 JSON 格式的文件中缓存/预加载 Kubernetes 元数据，命名为`namespace-pod.meta` |  |
| Dummy\_Meta | 如果设置，使用伪造的元数据\(多用于开发/测试\) | Off |

## 处理 'log' 字段值 <a id="processing-the-log-value"></a>

Kubernetes 过滤器提供了多种方式来处理 _log_ 键中包含的数据。假定您在 `parsers.conf` 中定义的原始 Docker 解析器如下:

```text
[PARSER]
    Name         docker
    Format       json
    Time_Key     time
    Time_Format  %Y-%m-%dT%H:%M:%S.%L
    Time_Keep    On
```

> 从 Fluent Bit v1.2 开始，如果您的输出使用 Elasticsearch 数据库，我们不建议使用 decoders 插件\(Decode\_Field\_As\)，以避免数据类型冲突。

要对 _log_ 键进行处理, 在过滤器插件配置中 **必须启用** the `Merge_Log` 配置属性，然后将执行以下处理顺序:

* 如果 Pod 设置了预定义的解析器，则过滤器使用该解析器处理 _log_ 字段的内容.
* 如果 Pod 没有设置预定义的解析器且设置了 `Merge_Parser` 配置项，则使用在配置中建议的解析器处理 _log_ 字段的内容.
* 如果Pod 没有设置预定义的解析器且未设置 `Merge_Parser`，则尝试将内容作为 JSON 格式处理

如果 _log_ 字段值处理失败，则该值保持不变。上面的顺序不是链式的，这意味着它是互斥的，过滤器将使用上述一个解析器进行解析，而不是全部解析器。

## Kubernetes Annotations

Kubernetes 过滤器的一个灵活的功能是允许 Kubernetes Pods 在处理记录时为日志处理器管道建议某些行为。目前它支持:

* 设置预定义解析器
* 请求将指定日志排除

可以使用以下注解:

| Annotation | Description | Default |
| :--- | :--- | :--- |
| fluentbit.io/parser\[\_stream\]\[-container\] | 设置预定义解析器。该解析器必须已经在 Fluent Bit 中注册。仅当 Fluent Bit 配置启用了 `K8S-Logging.Parser` 选项时，此选项才会生效。如果生效，`stream` 参数将被限定为 `stdout` 或 `stderr`\(表示 Pod 或容器的标准输出或标准错误输出\)，`container` 参数可以指定为 Pod 中指定的容器名 |  |
| fluentbit.io/exclude\[\_stream\]\[-container\] | 请求将指定 Pod 产生的日志排除。当 Fluent Bit 配置启用了 `K8S-Logging.Exclude` 选项时，此选项才会生效 | False |

### Pod 的注解示例 <a id="annotation-examples-in-pod-definition"></a>

#### 设置预定义解析器 <a id="suggest-a-parser"></a>

以下内容定义了一个将 Apache 日志发送到标准输出的 Pod。在 Pod 的注解中，使用名为 _apache_ 的预定义解析器处理数据:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apache-logs
  labels:
    app: apache-logs
  annotations:
    fluentbit.io/parser: apache
spec:
  containers:
  - name: apache
    image: edsiper/apache_logs
```

#### 请求将指定日志排除 <a id="request-to-exclude-logs"></a>

在某些情况下，用户希望请求日志处理器简单地跳过相关 Pod 中的日志:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apache-logs
  labels:
    app: apache-logs
  annotations:
    fluentbit.io/exclude: "true"
spec:
  containers:
  - name: apache
    image: edsiper/apache_logs
```

请注意，注解值是布尔值，可以使用 _true_ 或 _false_，并且必须加引号

## Tail + Kubernetes 过滤器的工作流程 <a id="workflow-of-tail-kubernetes-filter"></a>

Kubernetes 过滤器依赖 [tail](../inputs/tail.md) 或 [systemd](../inputs/systemd.md) 输入插件来处理日志数据，并使用 Kubernetes 元数据丰富日志数据。下面，我们将说明 Tail 的工作流程，以及它如何与 Kubernetes 过滤器相关联。考虑以下配置示例\(仅用于演示，不用于生产\):

```text
[INPUT]
    Name    tail
    Tag     kube.*
    Path    /var/log/containers/*.log
    Parser  docker

[FILTER]
    Name             kubernetes
    Match            kube.*
    Kube_URL         https://kubernetes.default.svc:443
    Kube_CA_File     /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    Kube_Token_File  /var/run/secrets/kubernetes.io/serviceaccount/token
    Kube_Tag_Prefix  kube.var.log.containers.
    Merge_Log        On
    Merge_Log_Key    log_processed
```

在 INPUT 配置段中，[tail](../../pipeline/input/tail.md) 插件将监控路径 `/var/log/containers/` 路径以 `.log` 结尾的所有文件。对于每个文件，它将读取每一行日志记录并应用 docker 解析器。然后，日志记录将被附加标签并发送到下一步。

Tail 插件支持标签扩展，这意味着如果标签带有星号\(\*\)，它将用受监控文件的绝对路径替换该值，因此，如果您的文件名路径为:

```text
/var/log/container/apache-logs-annotated_default_apache-aeeccc7a9f00f6e4e066aeff0434cf80621215071f1b20a51e8340aa7c35eac6.log
```

那么该文件的每个记录的标签将变为:

```text
kube.var.log.containers.apache-logs-annotated_default_apache-aeeccc7a9f00f6e4e066aeff0434cf80621215071f1b20a51e8340aa7c35eac6.log
```

> 请注意，斜杠用点代替

当运行 Kubernetes 过滤器时，它将尝试匹配所有以 `kube.` 开头的记录，因此上述文件中的记录将符合匹配规则，过滤器将尝试丰富日志记录。

Kubernetes 过滤器并不关心日志的来源，它仅关心受监控文件的绝对路径名称，因为该信息包含用于从 Kubernetes Master/API Server 检索正在运行的相关 Pod 的名称和名称空间名称的元数据信息。

如果设置了 **`Kube_Tag_Prefix`** 配置项\(Fluent Bit &gt;=1.1.x 可用\)，它将使用该值删除上一个 INPUT 配置段中添加的标记的前缀的。请注意，配置属性默认为 `kube_var.logs.containers`。因此以前的 Tag 内容从:

```text
kube.var.log.containers.apache-logs-annotated_default_apache-aeeccc7a9f00f6e4e066aeff0434cf80621215071f1b20a51e8340aa7c35eac6.log
```

转换为

```text
apache-logs-annotated_default_apache-aeeccc7a9f00f6e4e066aeff0434cf80621215071f1b20a51e8340aa7c35eac6.log
```

> 上面的转换不会修改原始 Tag，仅仅为过滤器执行元数据检索创建一个新的表示形式。

过滤器使用新值检索容器名称和名称空间，它使用内部正则表达式:

```text
(?<pod_name>[a-z0-9](?:[-a-z0-9]*[a-z0-9])?(?:\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)_(?<namespace_name>[^_]+)_(?<container_name>.+)-(?<docker_id>[a-z0-9]{64})\.log$
```

> 如果您想了解更多详细信息，参见该定义的源代码 [here](https://github.com/fluent/fluent-bit/blob/master/plugins/filter_kubernetes/kube_regex.h#L26>).

您可以在 [Rublar.com](https://rubular.com/r/HZz3tYAahj6JCd) 网站上查看此操作的执行方式。

#### 自定义正则表达式 <a id="custom-regex"></a>

在某些非常见的条件下，用户可能希望更改该硬编码的正则表达式，可以使用 **`Regex_Parser`** 选项。

#### 最终结果 <a id="final-comments"></a>

此时，过滤器可以收集 _pod\_name_ 和 _namespace_ 的值，并使用该信息将检查本地缓存\(内部哈希表\)中是否存在该键的某些元数据，如果存在，它将使用元数据值丰富日志记录，否则它将连接到 Kubernetes Master / API Server 并检索相关信息。

