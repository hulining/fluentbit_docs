# 内部状态导出/信号

服务运行时，我们可以公开[数据指标](monitoring.md)查看服务数据流的整体状态。但是在一些应用场景中，我们想知道当前服务内部的状态，特别是像_内部缓冲区的当前状态是什么？_之类问题的答案，内部状态导出功能会帮我们回答。

Fluent Bit v1.4 引入了内部状态导出功能，可以从命令行发送 `CONT` Unix 信号来轻松触发。

{% hint style="info" %}
注意: 此功能仅在 Linux 和 BSD 系列操作系统上可用
{% endhint %}

## Usage

运行以下 `kill` 命令来向 Fluent Bit 发信号：

```text
kill -CONT `pidof fluent-bit`
```

> 命令中的 `pidof` 表示 Fluent Bit 进程的 ID。你可以替换它

Fluent Bit 会将以下信息发送到标准输出\(stdout\):

```text
[engine] caught signal (SIGCONT)
[2020/03/23 17:39:02] Fluent Bit Dump

===== Input =====
syslog_debug (syslog)
│
├─ status
│  └─ overlimit     : no
│     ├─ mem size   : 60.8M (63752145 bytes)
│     └─ mem limit  : 61.0M (64000000 bytes)
│
├─ tasks
│  ├─ total tasks   : 92
│  ├─ new           : 0
│  ├─ running       : 92
│  └─ size          : 171.1M (179391504 bytes)
│
└─ chunks
   └─ total chunks  : 92
      ├─ up chunks  : 35
      ├─ down chunks: 57
      └─ busy chunks: 92
         ├─ size    : 60.8M (63752145 bytes)
         └─ size err: 0

===== Storage Layer =====
total chunks     : 92
├─ mem chunks    : 0
└─ fs chunks     : 92
   ├─ up         : 35
   └─ down       : 57
```

## 输出插件状态导出 <a id="input-plugins-dump"></a>

状态导出为每个配置的输入实例提供情报\(内部信息\)。

### Status

插件的总体状态。

| 项 | 子项 | 描述 |
| :--- | :--- | :--- |
| overlimit |  | 如果插件已启用 [`Mem_Buf_Limit`](backpressure.md) 配置，则此项将报告插件在状态导出时是否超过其限制。如果超过，则显示 `yes`，否则显示`no` |
|  | mem\_size | 输入插件正在使用的当前内存大小. |
|  | mem\_limit | Limit set by Mem\_Buf\_Limit设置的内存大小 |

### Tasks

当输入插件将数据提取到引擎中时，将创建一个 Chunk\(数据块\)。数据块可以包含多个记录。在达到刷新时间后，引擎将创建一个 Task\(任务\)，其中包含有相关数据块的路由。

Task 状态导出描述了与输入插件相关联的 Task 信息：

| 项 | 描述 |
| :--- | :--- |
| total\_tasks | 与输入插件生成的数据相关联的活动任务总数. |
| new | 尚未分配给输出插件的任务数。Task 在很短的时间内处于 `new` 新建状态\(大多数情况下，该值非常低或为零\) |
| running | 输出插件正在处理的活动任务数. |
| size | 正在处理的数据块使用的内存大小\(所有数据块的大小\). |

### Chunks

数据块状态导出提供了有关输入插件已生成并且仍在处理中的所有数据块的详细信息。

根据缓冲策略和配置所设置的参数限制，某些数据块可能处于 `up`\(内存中\)或 `down`\(文件系统中\)的状态

| 项 | 子项 | 描述 |
| :--- | :--- | :--- |
| total\_chunks |  | 由引擎处理的输入插件生成的数据块总数 |
| up\_chunks |  | 内存中加载的数据块总数 |
| down\_chunks |  | 存储在文件系统中，尚未加载到内存中的数据块总数 |
| busy\_chunks |  | 标记为 **busy**\(正在被刷入输出\)或已锁定的块。处于 **busy** 状态的数据块是不可变的，可能已经准备好\(或正在\)处理 |
|  | size | 数据块使用的字节数. |
|  | size err | 处于错误状态无法获取其大小的数据块的数量 |

## 存储层状态导出 <a id="storage-layer-dump"></a>

Fluent Bit 依赖为混合缓冲区设计的自定义存储层接口。`Storage Layer` 的项包含 Fluent Bit 注册的块的总体信息:

| 项 | 子项 | 描述 |
| :--- | :--- | :--- |
| total chunks |  | 数据块总数 |
| mem chunks |  | 基于内存的数据块总数 |
| fs chunks |  | 基于文件系统的数据块总数 |
|  | up | 由文件系统装入到内存中的数据块总数 |
|  | down | 文件系统未装入内存的数据块总数 |

