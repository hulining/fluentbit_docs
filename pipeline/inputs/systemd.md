# Systemd

**Systemd** 输入插件允许在 Linux 环境中从 Journald 守护进程收集日志消息。

## 配置参数 <a id="configuration-parameters"></a>

该输入插件支持以下配置参数:

| 键 | 描述 | 默认值 |
| :--- | :--- | :--- |
| Path | Systemd 日志目录的可选路径，如果未设置，该插件将使用默认路径读取仅本地日志 |  |
| Max\_Fields | 设置每个记录允许的最大字段 | 8000 |
| Max\_Entries | 当 Fluent Bit 启动时，日志队列中可能有大量日志。为避免延迟并减少内存使用量，此选项允许指定每次可以处理的最大日志条目数。一旦达到该限制，一旦 Journald 有新的通知，Fluent Bit 将继续处理剩余的日志条目. | 5000 |
| Systemd\_Filter | 查找包含特定日志键/值对日志，如 \_SYSTEMD\_UNIT=UNIT.可以在输入配置段中多次指定该选项，以根据需要应用多个过滤器 |  |
| Systemd\_Filter\_Type | 指定多个 Systemd\_Filter 时，定义过滤器类型。允许的值为 _And_ 和 _Or_ | Or |
| Tag | 该标签用于路由消息，但在 Systemd 插件上还有一个附加功能: 如果标签包含星号/通配符，它将使用 Systemd 单元文件\(如 host.\* =&gt; host.UNIT\_NAME\) |  |
| DB | 指定数据库的文件的路径以跟踪日志记录的游标 |  |
| Read\_From\_Tail | 是否仅读取新纪录，跳过已存储在 Journald 中的日志 | Off |
| Strip\_Underscores | 是否删除 Journald 字段\(key\)的前下划线.例如 _\_PID_ 变为 _PID_. | Off |

## 快速开始 <a id="getting-started"></a>

为了接收 Systemd 日志消息，您可以从命令行或通过配置文件运行插件:

### 命令行 <a id="command-line"></a>

在命令行中，您可以使用以下选项让 Fluent Bit 监听 _Systemd_ 消息:

```bash
$ fluent-bit -i systemd \
             -p systemd_filter=_SYSTEMD_UNIT=docker.service \
             -p tag='host.*' -o stdout
```

> 在上面的示例中，我们正在收集来自 Docker 服务的所有消息

### 配置文件 <a id="configuration-file"></a>

在您的主配置文件中，添加以下 _Input_ 和 _Output_ 配置段:

```text
[SERVICE]
    Flush        1
    Log_Level    info
    Parsers_File parsers.conf

[INPUT]
    Name            systemd
    Tag             host.*
    Systemd_Filter  _SYSTEMD_UNIT=docker.service

[OUTPUT]
    Name   stdout
    Match  *
```

