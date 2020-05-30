# Dummy

**dummy** 输入插件可生成虚拟事件记录。它对于测试，调试，压力测试和 Fluent Bit入 门非常有用。

## 配置参数 <a id="configuration-parameters"></a>

该插件支持以下配置参数:

| Key | Description |
| :--- | :--- |
| Dummy | 模拟的 JSON 事件记录，默认为 `{"message":"dummy"}` |
| Rate | 每秒产生的事件数，默认为 1 |

## 快速开始 <a id="getting-started"></a>

您可以从命令行或通过配置文件运行插件:

### 命令行 <a id="command-line"></a>

```bash
$ fluent-bit -i dummy -o stdout
Fluent Bit v1.x.x
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2017/07/06 21:55:29] [ info] [engine] started
[0] dummy.0: [1499345730.015265366, {"message"=>"dummy"}]
[1] dummy.0: [1499345731.002371371, {"message"=>"dummy"}]
[2] dummy.0: [1499345732.000267932, {"message"=>"dummy"}]
[3] dummy.0: [1499345733.000757746, {"message"=>"dummy"}]
```

### 配置文件 <a id="configuration-file"></a>

在您的主配置文件中，添加以下 _Input_ 和 _Output_ 配置段:

```python
[INPUT]
    Name   dummy
    Tag    dummy.log

[OUTPUT]
    Name   stdout
    Match  *
```

