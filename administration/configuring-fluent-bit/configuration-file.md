---
description: 本页面描述 Fluent Bit 的主配置文件
---

# 配置文件

配置 Fluent Bit 的方法之一是使用主配置文件。Fluent Bit 允许使用一个配置文件，该配置文件可以在全局范围内工作，并按照之前定义的[格式与模式](format-schema.md)使用。

主配置文件支持 4 种类型的配置段:

* Service
* Input
* Filter
* Output

此外，还可以使用函数将主配置文件拆分为多个文件，以包含外部文件:

* Include File

## Service <a id="config_section"></a>

_`Service`_ 配置段定义了服务的全局属性，下表中介绍了此版本可用的键:

| 键 | 描述 | 中文 | 默认值 |
| :--- | :--- | :--- | :--- |
| Flush | Set the flush time in `seconds.nanoseconds`. The engine loop uses a Flush timeout to define when is required to flush the records ingested by input plugins through the defined output plugins. | 以 `seconds.nanoseconds` 格式设置刷新时间。设置引擎将由输入插件进入的记录何时由输出插件输出 | 5 |
| Daemon | Boolean value to set if Fluent Bit should run as a Daemon \(background\) or not. Allowed values are: yes, no, on and off.  note: If you are using a Systemd based unit as the one we provide in our packages, do not turn on this option. | Fluent Bit 是否应该作为守护\(后台\)进程运行 | Off |
| Log\_File | Absolute path for an optional log file. By default all logs are redirected to the standard output interface \(stdout\). | 可选日志文件的绝对路径。默认情况下，所有日志都输出到标准输出\(stdout\) |  |
| Log\_Level | Set the logging verbosity level. Allowed values are: error, warning, info, debug and trace. Values are accumulative, e.g: if 'debug' is set, it will include error, warning, info and debug.  Note that _trace_ mode is only available if Fluent Bit was built with the _WITH\_TRACE_ option enabled. | 设置日志级别。可选值为: error, warning, info, debug and trace。注意，只有在构建时启用 _`WITH_TRACE`_, _trace_ 级别才可用 | info |
| Parsers\_File | Path for a `parsers` configuration file. Multiple Parsers\_File entries can be defined within the section. | `parsers` 配置文件路径。配置段中可配置多个 Parsers\_File 配置项 |  |
| Plugins\_File | Path for a `plugins` configuration file. A _plugins_ configuration file allows to define paths for external plugins, for an example [see here](https://github.com/fluent/fluent-bit/blob/master/conf/plugins.conf). | `plugins` 配置文件路径。_plugins_ 配置文件中可定义外部插件的路径。参见[示例](https://github.com/fluent/fluent-bit/blob/master/conf/plugins.conf) |  |
| Streams\_File | Path for the Stream Processor configuration file. To learn more about Stream Processing configuration go [here](../../stream-processing/introduction.md). | 流式处理器配置文件路径。有关流式处理配置，参见[这里](../../stream-processing/introduction.md) |  |
| HTTP\_Server | Enable built-in HTTP Server | 是否启用内置 HTTP 服务 | Off |
| HTTP\_Listen | Set listening interface for HTTP Server when it's enabled | HTTP 服务启用时,监听地址 | 0.0.0.0 |
| HTTP\_Port | Set TCP Port for the HTTP Server | HTTP 服务的 TCP 端口 | 2020 |
| Coro\_Stack\_Size | Set the coroutines stack size in bytes. The value must be greater than the page size of the running system. Don't set too small value \(say 4096\), or coroutine threads can overrun the stack buffer. Do not change the default value of this parameter unless you know what  you are doing. | 设置协程栈大小\(单位:字节\)。该值必须大于正在运行的系统页面大小。不要设置太小的值\(如 4096\)，否则协程线程可能会堆栈缓冲区溢出。除非您知道您自己在做什么，否则不要修改此参数默认值 | 24576 |

如下是一个 _SERVICE_ 配置段:

```python
[SERVICE]
    Flush           5
    Daemon          off
    Log_Level       debug
```

## Input <a id="config_input"></a>

_`INPUT`_ 配置段定义数据源\(与输入插件相关联\)，我们将描述 _INPUT_ 配置段的基本配置。请注意，每个输入插件都可以添加自己的配置项:

| 键 | 描述 |
| :--- | :--- |
| Name | 输入插件名称. |
| Tag | 与该插件产生的所有记录关联的标签名称 |

配置项 _`Name`_ 是必需的，它告知 Fluent Bit 应加载哪个输入插件。配置项 _`Tag`_ 除 _forward_ 输入插件\(因为它提供动态标签\)以外其它输入插件都是必需的:

### 示例 <a id="example"></a>

如下是 _INPUT_ 配置段的一个示例:

```python
[INPUT]
    Name cpu
    Tag  my_cpu
```

## Filter <a id="config_filter"></a>

_`FILTER`_ 配置段定义一个过滤器\(与过滤插件相关联\)，我们将描述 _FILTER_ 配置段的基本配置。请注意，每个过滤插件都可以添加自己的配置项:

| 键 | 描述 |  |
| :--- | :--- | :--- |
| Name | 过滤插件名称 |  |
| Match | 与传入记录的标签匹配的模式。它区分大小写并支持星号\(\*\)作为通配符 |  |
| Match\_Regex | 与传入记录的标签匹配的正则表达式。如果要使用完整的正则表达式语法，请使用此选项 |  |

配置项 _`Name`_ 是必需的，它告知 Fluent Bit 应加载哪个过滤插件。配置项 _`Match`_ 或 _`Match_Regex`_ 对于所有插件都是必需的。如果两者都指定，则 _`Match_Regex`_ 优先级更高。

### 示例 <a id="example-1"></a>

如下是 _FILTER_ 配置段的一个示例:

```python
[FILTER]
    Name  stdout
    Match *
```

## Output <a id="config_output"></a>

_`OUTPUT`_ 配置段指定记录标签匹配后的目的地。该配置支持以下配置项：

| 键 | 描述 |  |
| :--- | :--- | :--- |
| Name | 输出插件名称 |  |
| Match | 与传入记录的标签匹配的模式。它区分大小写并支持星号\(\*\)作为通配符 |  |
| Match\_Regex | 与传入记录的标签匹配的正则表达式。如果要使用完整的正则表达式语法，请使用此选项 |  |

### 示例 <a id="example-2"></a>

以下是 _`OUTPUT`_ 配置段的示例：

```python
[OUTPUT]
    Name  stdout
    Match my*cpu
```

### 示例: 收集 CPU 指标 <a id="example-collecting-cpu-metrics"></a>

以下配置文件示例演示了如何收集 CPU 指标并每五秒钟将结果刷新到标准输出:

```python
[SERVICE]
    Flush     5
    Daemon    off
    Log_Level debug

[INPUT]
    Name  cpu
    Tag   my_cpu

[OUTPUT]
    Name  stdout
    Match my*cpu
```

## Include File <a id="config_include_file"></a>

为避免复杂的长配置文件，最好将特定部分拆分为不同的文件，然后从主文件中调用\(include\)它们。

从 Fluent Bit 0.12 开始，添加了新的配置命令 _`@INCLUDE`_，可以通过以下方式使用:

```text
@INCLUDE somefile.conf
```

配置读取器将尝试打开文件 _somefile.conf_，如果找不到，它将假定它是基于主配置文件路径的相对路径，如:

* 主配置文件路径: /tmp/main.conf
* 引入文件: somefile.conf
* Fluent Bit 将尝试打开 somefile.conf，如果失败，则尝试打开 /tmp/somefile.conf.

找到文件后，其内容将替换 `@INCLUDE somefile.conf` 行。这是一个简单的文本包含。您仍然必须遵循前面的[格式与模式](format-schema.md)中的规范。例如，您不能定义多个 `[SERVICE]` 配置段。

_`@INCLUDE`_ 命令仅在配置文件的起始位置起作用，不能在配置段内部使用。

支持通配符\(\*\)包含多个文件，例如:

```text
@INCLUDE input_*.conf
```

