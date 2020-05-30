# FlowCounter

_FlowCounter_ 是对记录进行计数的协议。**flowcounter** 输出插件允许对记录及其大小进行计数。

## Configuration Parameters

该插件支持以下配置参数:

| Key | Description | Default |
| :--- | :--- | :--- |
| Unit | 持续时间的单位 \(second/minute/hour/day\) | minute |

## Getting Started

您可以通过命令行或通过配置文件运行插件:

### Command Line

在命令行中，您可以使用以下选项让 Fluent Bit 对数据进行计数:

```bash
$ fluent-bit -i cpu -o flowcounter
```

### Configuration File

在您的主配置文件中，添加如下 _Input_ 和 _Output_ 配置段:

```python
[INPUT]
    Name cpu
    Tag  cpu

[OUTPUT]
    Name flowcounter
    Match *
    Unit second
```

## Testing

Fluent Bit 运行后，您将看到类似于如下的输出:

```bash
$ fluent-bit -i cpu -o flowcounter  
Fluent Bit v1.x.x
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2016/12/23 11:01:20] [ info] [engine] started
[out_flowcounter] cpu.0:[1482458540, {"counts":60, "bytes":7560, "counts/minute":1, "bytes/minute":126 }]
```

