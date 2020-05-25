# 变量

使用配置文件时，Fluent Bit 支持在与键关联的任何值中使用环境变量。

变量区分大小写，可以按以下格式使用:

```text
${MY_VARIABLE}
```

当Fluent Bit启动时，配置读取器将检测对 `${MY_VARIABLE}` 的任何请求，并将尝试解析其值。

## 示例 <a id="example"></a>

创建如下配置文件\(`fluent-bit.conf`\):

```text
[SERVICE]
    Flush        1
    Daemon       Off
    Log_Level    info

[INPUT]
    Name cpu
    Tag  cpu.local

[OUTPUT]
    Name  ${MY_OUTPUT}
    Match *
```

打开终端并设置环境变量:

```bash
$ export MY_OUTPUT=stdout
```

> 上面的命令将变量 `MY_OUTPUT` 的值设置为 'stdout'。

使用最近创建的配置文件运行 Fluent Bit:

```text
$ bin/fluent-bit -c fluent-bit.conf
Fluent Bit v1.4.0
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2020/03/03 12:25:25] [ info] [engine] started
[0] cpu.local: [1491243925, {"cpu_p"=>1.750000, "user_p"=>1.750000, "system_p"=>0.000000, "cpu0.p_cpu"=>3.000000, "cpu0.p_user"=>2.000000, "cpu0.p_system"=>1.000000, "cpu1.p_cpu"=>0.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>4.000000, "cpu2.p_user"=>4.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>1.000000, "cpu3.p_user"=>1.000000, "cpu3.p_system"=>0.000000}]
```

您可以看到，配置生效，服务可以正常运行。

