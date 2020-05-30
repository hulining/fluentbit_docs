# File

**file** 输出插件可以将通过输入插件接收的数据写入到文件中。

## Configuration Parameters

该插件支持以下配置参数:

| Key | Description |
| :--- | :--- |
| Path | 输出文件路径。如果未设置，则输出文件名为标签名称 |
| Format | 文件内容的格式。另请参阅 [Format](file.md#format) 部分。默认值: `out_file` |

## Format

### out\_file format

`out_file` 格式输出 `time(时间)`,`tag(标签)` 及 JSON 格式的记录。`out_file` 格式没有配置参数。

```javascript
tag: [time, {"key1":"value1", "key2":"value2", "key3":"value3"}]
```

### plain format

`plain` 格式输出 JSON 格式的记录\(没有额外的 `tag(标签)` and `timestamp(时间戳)` 字段\)。`plain` 格式没有配置参数。

```javascript
{"key1":"value1", "key2":"value2", "key3":"value3"}
```

### csv format

`csv` 格式输出 csv 格式的记录。`csv` 格式支持一个额外的配置参数:

| Key | Description |
| :--- | :--- |
| Delimiter | 数据的分隔符。默认值为 `,` |

```python
time[delimiter]"value1"[delimiter]"value2"[delimiter]"value3"
```

### ltsv format

`ltsv` 格式将记录输出为 LTSV 格式。LTSV 支持额外的配置参数:

| Key | Description |
| :--- | :--- |
| Delimiter | 数据的分隔符。默认为 `\t`\(TAB\) |
| Label\_Delimiter | 标签与值的分隔符。默认为 `:` |

```python
field1[label_delimiter]value1[delimiter]field2[label_delimiter]value2\n
```

### template format

`template` 使用自定义格式输出记录。

| Key | Description |
| :--- | :--- |
| Template | 格式化的字符串。默认值为 `{time} {message}` |

`template` 接受格式模板，并使用记录中的值填充相应的占位符。

例如，如果您按如下所示内容进行配置:

```text
[INPUT]
  Name mem

[OUTPUT]
  Name file
  Format template
  Template {time} used={Mem.used} free={Mem.free} total={Mem.total}
```

你将得到如下输出:

```text
1564462620.000254 used=1045448 free=31760160 total=32805608
```

## Getting Started

您可以从命令行或通过配置文件运行插件:

### Command Line

在命令行中，您可以使用以下内容让 Fluent Bit 对数据进行收集:

```bash
$ fluent-bit -i cpu -o file -p path=output.txt
```

### Configuration File

在您的主配置文件中，添加以下 _Input_ 和 _Output_ 配置段:

```python
[INPUT]
    Name cpu
    Tag  cpu

[OUTPUT]
    Name file
    Match *
    Path output.txt
```

