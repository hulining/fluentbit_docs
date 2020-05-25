# 命令

配置文件必须足够灵活以适应任何部署需求，它们必须保持干净且易读的格式。

Fluent Bit _Commands_ 扩展了具有特定内置功能的配置文件。从Fluent Bit 0.12 系列开始可用的命令列表为:

| 命令 | 模版 | 描述 |
| :--- | :--- | :--- |
| [@INCLUDE](commands.md#cmd_include) | @INCLUDE FILE | 包含一个配置文件 |
| [@SET](commands.md#cmd_set) | @SET KEY=VAL | 配置配置变量 |

## @INCLUDE 命令 <a id="cmd_include"></a>

配置日志记录管道可能会有大量配置文件。为了保持易于理解的配置，建议将配置分成多个文件。

`@INCLUDE` 命令允许配置读取器包含一个外部配置文件，例如:

```text
[SERVICE]
    Flush 1

@INCLUDE inputs.conf
@INCLUDE outputs.conf
```

上面的示例定义了主服务配置文件，还包括两个文件以延续配置:

### inputs.conf

```text
[INPUT]
    Name cpu
    Tag  mycpu

[INPUT]
    Name tail
    Path /var/log/*.log
    Tag  varlog.*
```

### outputs.conf

```text
[OUTPUT]
    Name   stdout
    Match  mycpu

[OUTPUT]
    Name            es
    Match           varlog.*
    Host            127.0.0.1
    Port            9200
    Logstash_Format On
```

请注意，不管包含的顺序如何，Fluent Bit **始终**遵循以下顺序:

* Service
* Inputs
* Filters
* Outputs

## @SET 命令 <a id="cmd_set"></a>

Fluent Bit 支持[配置变量](variables.md),向 Fluent Bit 设置变量的一种方法是通过设置 Shell 环境变量，另一种是通过 _`@SET`_ 命令

`@SET` 命令只能在每行的根目录级别使用，这意味着它不能在配置段内使用，例如：

```text
@SET my_input=cpu
@SET my_output=stdout

[SERVICE]
    Flush 1

[INPUT]
    Name ${my_input}

[OUTPUT]
    Name ${my_output}
```

