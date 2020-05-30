# Standard Output

**stdout** 输出插件允许将通过输入插件接收的数据打印到标准输出。它的用法非常简单，如下所示:

## Configuration Parameters

| Key | Description | default |
| :--- | :--- | :--- |
| Format | 指定要输出的数据格式。支持的格式为 `msgpack`,`json`,`json_lines`,`json_stream` | msgpack |
| json\_date\_key | 在输出中指定日期字段名称 | date |
| json\_date\_format | 指定日期格式。支持的格式为 `double`,`iso8601`\(如: _2018-05-30T09:39:52.000681Z_\),`epoch`. | double |

### Command Line

```bash
$ bin/fluent-bit -i cpu -o stdout -v
```

我们使用 [CPU](../../pipleline/inputs/cpu.md) 输入插件收集 CPU 使用的数据指标，并以易读的方式将它们打印到标准输出中:

```bash
$ bin/fluent-bit -i cpu -o stdout -p format=msgpack -v
Fluent Bit v1.x.x
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2016/10/07 21:52:01] [ info] [engine] started
[0] cpu.0: [1475898721, {"cpu_p"=>0.500000, "user_p"=>0.250000, "system_p"=>0.250000, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>0.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>0.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>1.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>1.000000}]
[1] cpu.0: [1475898722, {"cpu_p"=>0.250000, "user_p"=>0.250000, "system_p"=>0.000000, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>1.000000, "cpu1.p_user"=>1.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>0.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>0.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>0.000000}]
[2] cpu.0: [1475898723, {"cpu_p"=>0.750000, "user_p"=>0.250000, "system_p"=>0.500000, "cpu0.p_cpu"=>2.000000, "cpu0.p_user"=>1.000000, "cpu0.p_system"=>1.000000, "cpu1.p_cpu"=>0.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>1.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>1.000000, "cpu3.p_cpu"=>0.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>0.000000}]
[3] cpu.0: [1475898724, {"cpu_p"=>1.000000, "user_p"=>0.750000, "system_p"=>0.250000, "cpu0.p_cpu"=>1.000000, "cpu0.p_user"=>1.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>2.000000, "cpu1.p_user"=>1.000000, "cpu1.p_system"=>1.000000, "cpu2.p_cpu"=>1.000000, "cpu2.p_user"=>1.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>1.000000, "cpu3.p_user"=>1.000000, "cpu3.p_system"=>0.000000}]
```

No more, no less, it just works.

