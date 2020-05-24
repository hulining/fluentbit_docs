---
description: Kubernetes 生产级日志处理器
---

# Kubernetes

![](../.gitbook/assets/fluentbit_kube_logging.png)

[Fluent Bit](http://fluentbit.io) 是一个轻量级且可扩展的**日志处理器**，它完全支持 Kubernetes:

* 从文件系统或 Systemd/Journaled 处理 Kubernetes 容器日志
* 使用 Kubernetes 元数据丰富日志
* 将您的日志汇聚到第三方存储服务中，例如 Elasticsearch，InfluxDB，HTTP 等

## 相关概念 <a id="concepts"></a>

在开始之前，重要的是要了解如何部署 Fluent Bit。Kubernetes 管理 _nodes_ 集群，因此我们的日志代理工具需要在每个节点上运行以从每个 _POD_ 收集日志，因此Fluent Bit 被部署为 DaemonSet\(在集群的每个 _node_ 上运行的 POD\)。

当 Fluent Bit 运行时，它将读取，解析和过滤每个 POD 的日志，并将使用以下信息\(元数据\)丰富每条数据:

* Pod Name
* Pod ID
* Container Name
* Container ID
* Labels
* Annotations

为了获得这些信息，名为 _`kubernetes`_ 的内置过滤器插件与 Kubernetes API Server 进行通信以检索相关信息，例如 _`pod_id`_，_`labels`_ 和 _`annotations`_，其他字段，例如 _`pod_name`_，_`container_id`_ 和 _`container _name`_ 是从本地日志文件名检索。所有这些都是自动处理的，从配置方面来说不需要干预。

> Kubernetes 过滤器插件完全受 [Jimmi Dyson](https://github.com/jimmidyson) 编写的 [Fluentd Kubernetes 元数据过滤器](https://github.com/fabric8io/fluent-plugin-kubernetes_metadata_filter)的启发。

## 部署 <a id="installation"></a>

[Fluent Bit](http://fluentbit.io) 必须作为 DaemonSet 部署，这样就可以在 Kubernetes 集群的每个节点上使用它。首先，请使用以下命令来创建名称空间，服务帐号和角色设置\(namespace, serviceaccount, role\):

```text
$ kubectl create namespace logging
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-service-account.yaml
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-role.yaml
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-role-binding.yaml
```

然后创建由 Fluent Bit DaemonSet 使用的 ConfigMap:

```text
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-configmap.yaml
```

### Kubernetes &lt; v1.16 注意事项 <a id="note-for-kubernetes-less-than-v-1-16"></a>

对于 v1.16 之前版本的 Kubernetes，DaemonSet 资源对象在 `apiVersion: apps/v1` 上不可用，该资源对象在 `apiVersion: extensions/v1beta1` 上可用。我们当前的 Daemonset Yaml 文件使用新版本的 `apiVersion`。

如果您使用的是较旧版本的 Kubernetes，请手动获取 Daemonset Yaml 文件，并替换 `apiVersion` 的值，从:

```yaml
apiVersion: apps/v1
```

替换为

```yaml
apiVersion: extensions/v1beta1
```

您可以在 Kubernetes v1.14 Changelog 上阅读有关此变更/弃用的更多信息: [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.14.md\#deprecations](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.14.md#deprecations)

### Fluent Bit to Elasticsearch

Fluent Bit DaemonSet 可以与 Kubernetes 集群上的 Elasticsearch 一起使用:

```text
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-ds.yaml
```

### Fluent Bit to Elasticsearch on Minikube

如果您使用 Minikube 进行测试，请使用以下 DaemonSet 清单:

```text
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-ds-minikube.yaml
```

## 详情 <a id="details"></a>

Fluent Bit 的默认配置可确保以下内容:

* 使用运行节点上的所有容器日志
* [Tail 输入插件](../pipeline/inputs/tail) 不会在引擎中添加超过 **5MB** 的内容，直到将他们刷新到 Elasticsearch。此限制旨在为[积压](../administration/backpressure)方案提供解决方法
* Kubernetes 过滤器插件将使用 Kubernetes 元数据\(特别是 _labels_ 和 _annotations_\)丰富日志数据。过滤器仅在找不到缓存信息时才访问 API 服务器，否则将使用缓存
* 配置中的默认后端是由 [Elasticsearch 输出插件](../pipeline/inputs/elasticsearch)设置的 Elasticsearch。它使用 Logstash 格式处理日志。如果您需要其他索引和类型，请参考插件选项并自行进行调整
* 选项 **`Retry_Limit`** 设置为 False，这意味着如果 Fluent Bit 无法将记录刷新到 Elasticsearch，它将无限期地重试直到成功

