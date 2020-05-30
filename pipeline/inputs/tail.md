# tail

**tail** 输入插件允许监测一个或多个文本文件。它具有类似于 `tail -f` 的 shell 命令行功能。

该插件读取 _Path_ 模式中的每个匹配文件，并为每个新行\(分隔符为`\n`\)生成一条新纪录。作为可选的，可以使用数据库文件，以便插件可以跟踪文件的历史记录和偏移状态，这对于重启服务时的状态恢复非常有用。

## 配置参数 <a id="config"></a>

该插件支持以下配置参数:

| 键 | 描述 | 默认值 |
| :--- | :--- | :--- |
| Buffer\_Chunk\_Size | 设置读取文件数据的初始缓冲区大小。该值也用于增加缓冲区大小。该值必须符合[单位](../../administration/configuring-fluent-bit/unit-sizes.md)章节中的规范 | 32k |
| Buffer\_Max\_Size | 设置每个监控文件的缓冲区大小。当需要增加缓冲区时，此值用于限制内存缓冲区可以增加多少。如果超过此限制，则将从监控文件列表中删除该文件。该值必须符合[单位](../../administration/configuring-fluent-bit/unit-sizes.md)章节中的规范 | Buffer\_Chunk\_Size |
| Path | 通过使用通配符指定一个或多个日志文件的 |  |
| Path\_Key | 如果启用，它将附加监控文件的名称作为记录的一部分。指定的值成为映射中的键 |  |
| Exclude\_Path | 设置一个或多个用逗号分隔的模式，以排除符合特定条件的文件。例如 exclude\_path=\*.gz,\*.zip |  |
| Refresh\_Interval | 刷新监控文件列表的时间间隔\(秒\) | 60 |
| Rotate\_Wait | 指定监控文件的额外时间，以防止日志文件滚动丢失某些数据 | 5 |
| Ignore\_Older | 忽略比该时间旧的记录\(秒\)。支持 m,h,d\(分钟,小时,天\)。默认行为是从指定文件中读取所有记录。 仅在指定了解析器并且可以解析记录时间时才可用 |  |
| Skip\_Long\_Lines | 当监控的文件由于行\(Buffer\_Max\_Size\)很长而达到缓冲区容量时，默认行为是停止监视该文件。Skip\_Long\_Lines 会更改该行为，并指示 Fluent Bit 跳过长行并继续处理适合缓冲区大小的其他行 | Off |
| DB | 指定跟踪监控文件的偏移量的数据库文件 |  |
| DB.Sync | 设置默认的同步方法。可选值: Extra, Full, Normal, Off.此标志影响内部 SQLite 引擎与磁盘同步的方式，有关选项的更多详细信息，请参阅 [sqlite 文档](https://www.sqlite.org/pragma.html#pragma_synchronous). | Full |
| Mem\_Buf\_Limit | 设置将数据追加到引擎时的内存限制。如果达到此限制，它将被暂停；刷新数据后，它将恢复. |  |
| Parser | 指定解析器的名称，将记录转化为结构化消息 |  |
| Key | 当消息是非结构化数据时\(未应用解析器\)，消息将以字符串形式作为 _`log`_ 键的值。此选项允许为该键指定名称 | log |
| Tag | 为读取的行设置标签\(带有正则表达式字段\)。如 `kube.<namespace_name>.<pod_name>.<container_name>`.请注意支持如下"标签扩展"规则: 如果标签包含星号\(\*\)，则星号\(\*\)将被替换为文件的绝对路径\(请参阅 [Workflow of Tail + Kubernetes Filter](../../pipeline/filters/kubernetes.md) |  |
| Tag\_Regex | 设置正则表达式以从文件中提取字段.如 `(?<pod_name>[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)_(?<namespace_name>[^_]+)_(?<container_name>.+)-` |  |

请注意，如果未指定数据库参数 _db_，默认情况下，插件将从头开始读取每个目标文件。

### 多行配置参数 <a id="multiline"></a>

此外，还存在以下选项来配置多行文件的处理:

| 键 | 描述 | 默认值 |
| :--- | :--- | :--- |
| Multiline | 如果启用，该插件将尝试发现多行消息并使用适当的解析器组成输出消息。请注意，启用此选项后，将不使用 Parser 选项 | Off |
| Multiline\_Flush | 处理队列中多行消息的等待时间 | 4 |
| Parser\_Firstline | 多行消息的开头匹配的解析器的名称。请注意，解析器中定义的正则表达式必须包含组名\(名为capture \) |  |
| Parser\_N | 可选的额外解析器，用于解析并结构化多行条目。此选项可用于定义多个解析器，例如：Parser\_1 ab1，Parser\_2 ab2，Parser\_N abN |  |

### Docker 模式配置参数 <a id="docker_mode"></a>

还包含 Docker 模式，用于重组由于其行长限制被 Docker 守护程序分割的 JSON 日志行。要使用此功能，请配置 tail 插件使用相应的解析器，然后启用 Docker 模式:

| Key | Description | Default |
| :--- | :--- | :--- |
| Docker\_Mode | 如果启用，该插件将重新组合已分割的 Docker 日志行，然后将其传递至如上配置的任何解析器。此模式不能与多行模式同时使用 | Off |
| Docker\_Mode\_Flush | 刷新队列中未完成的分割行的等待时间 | 4 |

## 快速开始 <a id="getting_started"></a>

为了实时读取文本或日志文件，您可以从命令行或通过配置文件运行插件:

### 命令行 <a id="command-line"></a>

在命令行中，您可以使用以下选项让 Fluent Bit 解析文本文件

```bash
$ fluent-bit -i tail -p path=/var/log/syslog -o stdout
```

### 配置文件 <a id="configuration-file"></a>

在您的主配置文件中，添加以下 _Input_ 和 _Output_ 配置段:

```python
[INPUT]
    Name        tail
    Path        /var/log/syslog

[OUTPUT]
    Name   stdout
    Match  *
```

## 文件状态跟踪 <a id="keep_state"></a>

强烈建议您启用 _tail_  输入插件可保存跟踪文件的状态的功能。为此，可以使用 **db** 属性,如:

```bash
$ fluent-bit -i tail -p path=/var/log/syslog -p db=/path/to/logs.db -o stdout
```

运行时，数据库文件_`/path/to/logs.db`_ 将被创建，该数据库由 SQLite3 支持，因此，如果您有兴趣探索内容，则可以使用 SQLite 客户端工具将其打开，例如:

```text
$ sqlite3 tail.db
-- Loading resources from /home/edsiper/.sqliterc

SQLite version 3.14.1 2016-08-11 18:53:32
Enter ".help" for usage hints.
sqlite> SELECT * FROM in_tail_files;
id     name                              offset        inode         created
-----  --------------------------------  ------------  ------------  ----------
1      /var/log/syslog                   73453145      23462108      1480371857
sqlite>
```

> 确保 Fluent Bit 没有依赖该数据库再浏览数据库文件，否则您会看到一些 _Error: database is locked_ 错误消息。

#### 格式化 SQLite

默认情况下，SQLite 客户端工具不会以人类易读的方式格式化列，因此要浏览 _in\_tail\_files_ 表，您可以在 **`~/.sqliterc`** 中创建具有以下内容的配置文件

```text
.headers on
.mode column
.width 5 32 12 12 10
```

## 文件滚动 <a id="files-rotation"></a>

文件滚动可以被正确的处理，包括日志滚动 _copytruncate\(复制截断\)_ 模式。

