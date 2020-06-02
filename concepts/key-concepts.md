---
description: 以下关键概念对于理解 Fluent Bit 的工作方式非常重要
---

# 核心概念

在开始使用 [Fluent Bit](https://fluentbit.io/) 之前，最好先了解一些服务的关键概念。本文档对这些概念和通用 [Fluent Bit](https://fluentbit.io/) 术语进行了简要介绍。我们列出了所有术语的清单，但建议您从头到尾阅读本文档，以更全面地了解日志或数据流处理器。

* Event or Record\(事件或记录\)
* Filtering\(过滤\)
* Tag\(标签\)
* Timestamp\(时间戳\)
* Match\(匹配\)
* Structured Message\(结构化消息\)

## 事件或记录 <a id="event-or-record"></a>

传入 Fluent Bit 的每条日志或指标数据都被视为时间或记录。

例如，一个 Syslog 文件的内容如下:

```text
Jan 18 12:52:16 flb systemd[2222]: Starting GNOME Terminal Server
Jan 18 12:52:16 flb dbus-daemon[2243]: [session uid=1000 pid=2243] Successfully activated service 'org.gnome.Terminal'
Jan 18 12:52:16 flb systemd[2222]: Started GNOME Terminal Server.
Jan 18 12:52:16 flb gsd-media-keys[2640]: # watch_fast: "/org/gnome/terminal/legacy/" (establishing: 0, active: 0)
```

它包含四行，并且它们代表**四个**独立的事件。

在内部，事件通常由两部分组成\(以数组形式表示\):

```javascript
[TIMESTAMP, MESSAGE]
```

## 过滤 <a id="filtering"></a>

在某些情况下，我们需要对事件内容进行修改。修改，丰富或删除事件的过程被称为过滤

过滤过程由需要应用场景，例如:

* 将特定的信息\(如 IP 地址或元数据\)添加到事件
* 选择特定的事件内容
* 删除与某些模式匹配的事件

## 标签 <a id="tag"></a>

每个进入 Fluent Bit 的事件都会分配一个标签。此标签是一个内部字符串，它由路由器在以后的阶段中使用，以确定该事件必须经过哪个过滤器或路由到哪个输出。

大多数的标签是在配置中手动指定的。如果未指定标签，Fluent Bit 将自动分配为生成该事件的输入插件的名称

{% hint style="info" %}
唯一**不**分配的标签的输入插件是 [Forward](../pipeline/inputs/forward.md) 输出插件。该插件使用被称为 "Forward"\(转发\) 的 Fluentd 网络协议，其中每个事件都已带有相关联的标签。Fluent Bit 将始终使用客户端设置的传入标签。
{% endhint %}

## 时间戳 <a id="timestamp"></a>

时间戳表示事件创建的 _时刻_。每个事件都包含一个时间戳。时间戳是数字形式的小数，格式如下:

```javascript
SECONDS.NANOSECONDS
```

### Seconds

自 Unix 纪元以来经过的秒数

### Nanoseconds

小数秒或纳秒

{% hint style="info" %}
时间戳始终存在，可以通过输入插件设置，也可以通过数据解析过程发现。
{% endhint %}

## \(标签\)匹配 <a id="match"></a>

Fluent Bit 允许将收集和处理的事件传递到一个或多个目的地，这是通过路由阶段完成的。匹配表示一个简单的规则，用于选择标签与定义规则相匹配的事件

要了解有关标签和匹配的更多信息，请查看[路由](data-pipeline/router.md)部分.

## 结构化消息 <a id="structured-messages"></a>

源事件可以有或没有结构。结构在事件消息中定义了一组 _keys_ 和 _values_\(键值对\)。作为示例，请参考如下两条消息:

### 非结构化消息 <a id="no-structured-message"></a>

```javascript
"Project Fluent Bit created on 1398289291"
```

### 结构化消息 <a id="structured-message"></a>

```javascript
{"project": "Fluent Bit", "created": 1398289291}
```

在底层结构上，它们都是一个字节数组，但是结构化消息定义了键值对。结构有助于实现对数据修改的更快操作。

{% hint style="info" %}
Fluent Bit **总是**将每个消息作为结构化消息处理。出于高性能性能的考虑，我们使用被称为 [MessagePack](https://msgpack.org/) 的二进制序列化数据格式。

可以将 [MessagePack](https://msgpack.org/) 视为 JSON 的二进制版本
{% endhint %}

