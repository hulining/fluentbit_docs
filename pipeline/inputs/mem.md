# Memory Metics

**mem** 输入插件，每隔一定的时间间隔收集正在运行的系统的内存和 swap 使用情况的信息，并报告内存总量和可用空间。

## 快速开始 <a id="getting-started"></a>

要从系统获取内存和 swap 使用情况，可以从命令行或通过配置文件运行插件:

### 命令行 <a id="commo"></a>

```bash
$ fluent-bit -i mem -t memory -o stdout -m '*'
Fluent Bit v1.x.x
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2017/03/03 21:12:35] [ info] [engine] started
[0] memory: [1488543156, {"Mem.total"=>1016044, "Mem.used"=>841388, "Mem.free"=>174656, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
[1] memory: [1488543157, {"Mem.total"=>1016044, "Mem.used"=>841420, "Mem.free"=>174624, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
[2] memory: [1488543158, {"Mem.total"=>1016044, "Mem.used"=>841420, "Mem.free"=>174624, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
[3] memory: [1488543159, {"Mem.total"=>1016044, "Mem.used"=>841420, "Mem.free"=>174624, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
```

### 配置文件 <a id="configuration-file"></a>

在您的主配置文件中，添加以下 _Input_ 和 _Output_ 配置段:

```python
[INPUT]
    Name   mem
    Tag    memory

[OUTPUT]
    Name   stdout
    Match  *
```

