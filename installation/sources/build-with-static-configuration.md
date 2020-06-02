# 以静态配置构建

通常，[Fluent Bit](https://fluentbit.io) 允许通过[文本文件](../../administration/configuring-fluent-bit/configuration-file.md)或在命令行中使用特定参数进行配置，虽然这是理想的部署情况，但在某些情况下需要进行更严格的配置: 静态配置模式。

静态配置模式旨在在 Fluent Bit 的最终二进制文件中包含内置配置，从而在运行时禁用外部文件或标志的使用。

## 开始 <a id="getting-started"></a>

### 依赖 <a id="requirements"></a>

以下步骤假定您熟悉使用文本文件配置 Fluent Bit 的经验，并且具有[构建和安装](build-and-install.md)部分描述的从头开始构建它的经验。

#### 配置文件路径 <a id="configuration-directory"></a>

在文件系统中准备一个用作构建系统查找和解析配置文件的入口的目录。该目录必须至少包含一个名为 _`fluent-bit.conf`_ 的配置文件，其中包含必需的 [SERVICE](../../administration/configuring-fluent-bit/configuration-file.md#config_section), [INPUT](../../administration/configuring-fluent-bit/configuration-file.md#config_input) 和 [OUTPUT](../../administration/configuring-fluent-bit/configuration-file.md#config_output) 部分。例如，创建一个新的 _`fluent-bit.conf`_ 文件，内容如下:

```text
[SERVICE]
    Flush     1
    Daemon    off
    Log_Level info

[INPUT]
    Name      cpu

[OUTPUT]
    Name      stdout
    Match     *
```

上面提供的配置将根据正在运行的系统计算 CPU 指标并将其打印到标准输出。

#### 基于自定义配置构建 <a id="build-with-custom-configuration"></a>

在 Fluent Bit 源代码中，进入 _`build/`_ 目录，运行`cmake` 命令并追加 `FLB_STATIC _CONF` 选项，指向最近创建的配置目录，例如

```bash
$ cd fluent-bit/build/
$ cmake -DFLB_STATIC_CONF=/path/to/my/confdir/
```

然后进行构建

```bash
$ make
```

此时生成的 fluent-bit 二进制文件可以直接运行而无需进一步配置:

```bash
$ bin/fluent-bit
Fluent-Bit v0.15.0
Copyright (C) Treasure Data

[2018/10/19 15:32:31] [ info] [engine] started (pid=15186)
[0] cpu.local: [1539984752.000347547, {"cpu_p"=>0.750000, "user_p"=>0.500000, "system_p"=>0.250000, "cpu0.p_cpu"=>1.000000, "cpu0.p_user"=>1.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>0.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>0.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>1.000000, "cpu3.p_user"=>1.000000, "cpu3.p_system"=>0.000000}]
```

