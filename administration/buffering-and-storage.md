# 缓冲与存储

[Fluent Bit](https://fluentbit.io/) 的终极目标是收集、解析、过滤和传送日志到中心位置。在此工作流程中，有很多阶段，而关键之一就是_缓冲_: 一种将处理后的数据放置到临时位置直到准备好传送的机制。

默认情况下，Fluent Bit 处理数据时，它使用内容作为存储记录日志的主要和临时位置，但是在某些情况下，理想的情况是在文件系统中具有持久性缓冲机制以提供聚合和数据安全能力。

从 Fluent Bit v1.0 开始，我们引入了一个新的_存储层_，它可以在内存或文件系统中工作。输入插件可配置为在启动时根据需要使用其中一个。

## 配置 <a id="configuration"></a>

存储层相关内容可以在如下两个配置段中配置:

* Service 配置段
* Input 配置段

Service 配置段为存储层配置一个全局环境，然后在 Input 配置段中定义要使用的机制。

### Service 配置段配置 <a id="service-section-configuration"></a>

Service 配置段指在主[配置文件](configuring-fluent-bit/configuration-file.md):

| 配置项 | 描述 | 默认值 |
| :--- | :--- | :--- |
| storage.path | 设置一个可选位置以存储流和数据块。如果未设置此参数，则输入插件只能使用内存缓冲 |  |
| storage.sync | 数据存储到文件系统中的同步模式。_`normal`_ 或 _`full`_ | normal |
| storage.checksum | 从文件系统写入和读取数据时，启用数据完整性检查。存储层使用 CRC32 算法 | Off |
| storage.backlog.mem\_limit | 如果设置了 _`storage.path`_，Fluent Bit 将查找未传送且仍在存储层中的数据块，这些数据块称为 _backlog_ 数据。此选项配置在处理这些记录时使用的最大内存 | 5M |

Service 配置段配置示例如下:

```text
[SERVICE]
    flush                     1
    log_Level                 info
    storage.path              /var/log/flb-storage/
    storage.sync              normal
    storage.checksum          off
    storage.backlog.mem_limit 5M
```

该配置配置了一个可选的缓冲机制，其中数据的根目录为 _/var/log/flb-storage/_，它将使用 _normal_ 同步模式，没有校验和，并在处理积压数据时最多可使用 5MB 的内存。

### Input 配置段配置 <a id="input-section-configuration"></a>

可选的，任何输入插件都可以配置其存储选项，下表描述了可用的选项:

| 配置项 | 描述 | 默认值 |
| :--- | :--- | :--- |
| storage.type | 指定要使用的缓冲机制。可选值为 _memory_ 或 _filesystem_ | memory |

以下示例配置了一个提供文件系统缓冲功能的服务，以及两个输入插件，第一个基于内存作为缓冲，第二个基于文件系统作为缓冲:

```text
[SERVICE]
    flush                     1
    log_Level                 info
    storage.path              /var/log/flb-storage/
    storage.sync              normal
    storage.checksum          off
    storage.backlog.mem_limit 5M

[INPUT]
    name          cpu
    storage.type  filesystem

[INPUT]
    name          mem
    storage.type  memory
```

