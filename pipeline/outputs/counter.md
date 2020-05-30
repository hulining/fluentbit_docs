# Counter

**counter** 计数器插件是一个非常简单的插件，可以计算刷新的记录数。插件输出如下:

```text
[TIMESTAMP, NUMBER_OF_RECORDS_NOW] (total = RECORDS_SINCE_IT_STARTED)
```

## Getting Started

您可以从命令行或通过配置文件运行此插件:

### Command Line

在命令行中，您可以使用以下选项让 Fluent Bit 对数据进行计数:

```bash
$ fluent-bit -i cpu -o counter
```

### Configuration File

在主配置文件中，添加以下 _Input_ 和 _Output_ 配置段:

```python
[INPUT]
    Name cpu
    Tag  cpu

[OUTPUT]
    Name  counter
    Match *
```

## Testing

Fluent Bit 运行后，您将看到如下输出:

```bash
$ bin/fluent-bit -i cpu -o counter -f 1
Fluent Bit v1.x.x
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2017/07/19 11:19:02] [ info] [engine] started
1500484743,1 (total = 1)
1500484744,1 (total = 2)
1500484745,1 (total = 3)
1500484746,1 (total = 4)
1500484747,1 (total = 5)
```

