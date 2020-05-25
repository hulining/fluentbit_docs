# 上游服务负载均衡

Fluent Bit [输出插件](pipeline/outputs)通过网络连接外部服务交付日志是很常见的，例如 [HTTP](pipeline/outputs/http.md), [Elasticsearch](pipeline/outputs/elasticsearch.md) 和 [Forward](pipeline/outputs/forward.md) 等其他插件都是这种情况。比较常见的是连接到一个节点\(主机\)，可以满足许多应用场景，但是在某些情况下，则需要在不同节点进行负载均衡。 _Upstream_ 特性提供了这种能力。

_`Upstream`_ 定义了作为输出插件目标一组节点，根据实现的性质，输出插件**必须**支持 _Upstream_ 功能。以下插件支持 _Upstream_:

* [Forward](pipeline/outputs/forward.md)

当前实现的负载均衡模式是 _round-robin\(轮询\)_。

## 配置 <a id="configuration"></a>

要定义 _Upstream_，需要创建一个特定的配置文件，其中包含 UPSTREAM 和一个或多个 NODE 配置段。下表描述了与每个配置段关联的属性。请注意，所有这些都是必须的:

| 配置段 | 配置项/键 | 描述 |
| :--- | :--- | :--- |
| UPSTREAM | name | 定义 _Upstream_ 名称 |
| NODE | name | 定义 _Node_ 名称,包含一个节点 |
|  | host | 目标主机的 IP 地址或主机名 |
|  | port | 目标服务的 TCP 端口 |

### NODE 和特定插件配置 <a id="nodes-and-specific-plugin-configuration"></a>

_NODE_ 可能包含其他插件所需的配置项，这样我们为输出插件提供了足够的灵活性。一个常见的用例是 Forward 输出插件如果启用了TLS，则它需要一些共享的配置项\(下面示例中包含更多详细信息\)。

### Nodes and TLS \(Transport Layer Security\)

除了上表中定义的属性之外，可以通过使用 TLS 加密和证书完成对一个节点的网络进行加密。

可用的 TLS 选项在 [TLS/SSL](administration/security.md) 部分中进行了描述，且可以添加到任何 _Node_ 配置段中。

### 配置文件示例 <a id="configuration-file-example"></a>

如下示例定义了一个名称为 `forward-balancing` 的_Upstream_，供 Forward 输出插件使用，它注册了三个 _Nodes_:

* node-1: 连接到 127.0.0.1:43000
* node-2: 连接到 127.0.0.1:44000
* node-3: 连接到 127.0.0.1:45000，使用 TLS 而无需验证。它还定义了名称 _`shared_key`_ 的 Forward 输出插件所需的特定配置项。

```python
[UPSTREAM]
    name       forward-balancing

[NODE]
    name       node-1
    host       127.0.0.1
    port       43000

[NODE]
    name       node-2
    host       127.0.0.1
    port       44000

[NODE]
    name       node-3
    host       127.0.0.1
    port       45000
    tls        on
    tls.verify off
    shared_key secret
```

请注意，每个 _Upstream_ **必须**定义在自己的配置文件中。不允许在同一文件或不同文件中添加多个 _Upstreams_。

