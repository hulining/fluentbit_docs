---
description: 功能强大且灵活的路由
---

# 标签重写

Tags\(标签\)使 [routing\(路由\)](../../concepts/data-pipeline/router.md)成为可能。标签是在生成记录的输入定义的配置中设置的。在某些情况下，修改管道中的标签可能有用，我们可以执行更高级，更灵活的路由。

`rewrite_tag` 过滤器允许重新发送带有新标签的记录。记录重新发送后，原始记录可以选择保留或丢弃。

## How it Works

它的工作方式是定义特定记录中指定键内容与正则表达式进行匹配的规则，如果匹配，则发出包含已定义标签的新纪录。可以指定多个规则，并按顺序处理它们，知道其中之一匹配为止:

新标记可以由以下组成:

* 字母和数字
* 原始标签字符串或其中一部分
* 正则表达式组捕获
* 处理后记录的任何键或子键
* 环境变量

## Configuration Parameters

`rewrite_tag` 过滤器支持以下配置参数:

| Key | Description | 翻译 |
| :--- | :--- | :--- |
| Rule | Defines the matching criteria and the format of the Tag for the matching record. The Rule format have four components: `KEY REGEX NEW_TAG KEEP`. For more specific details of the Rule format and it composition read the next section. | 定义重写标签的规则，格式一般为 `KEY REGEX NEW_TAG KEEP` |
| Emitter\_Name | When the filter emits a record under the new Tag, there is an internal emitter plugin that takes care of the job. Since this emitter expose metrics as any other component of the pipeline, you can use this property to configure an optional name for it. | 定义发送新纪录的发射器的名称\(可选\) |
| Emitter\_Storage.type | Define a buffering mechanism for the new records created. Note these records are part of the emitter plugin. This option support the values `memory` \(default\) or `filesystem`. If the destination for the new records generated might face backpressure due to latency or slow network, we strongly recommends enabling the `filesystem` mode. | 为新记录定义缓冲机制，可选值为 `memory`\(默认\)或 `filesystem` |

## Rules

Rules\(规则\)定义记录的匹配条件并指定如何为记录创建新标签。您可以在同一配置部分中定义一个或多个规则。规则为以下格式:

```text
$KEY  REGEX  NEW_TAG  KEEP
```

### Key

KEY \(键\)指定日志记录中存在的键，其值用于正则表达式\(REGEX\)匹配。键名以 `$` 作为前缀。考虑以下结构化记录\(为便于阅读而格式化\):

```javascript
{
  "name": "abc-123",
  "ss": {
    "s1": {
      "s2": "flb"
    }
  }
}
```

如果我们想要键 `name` 的值与正则表达式相匹配，则必须使用 `$name`。键选择器足够灵活，可以匹配结构中具有嵌套关系的子映射。如果我们想检查嵌套键 `s2` 的值，我们可以将 KEY 指定为 `$ss['s1']['s2']`，如下:

* `$name` = "abc-123"
* `$ss['s1']['s2']` = "flb"

请注意，KEY 必须指向包含字符串值的键，对于数字，布尔值，映射或数组**无效**

### Regex

使用一个简单的正则表达式，我们可以指定一个模式，用于匹配指定键的值，也可以利用组捕获来创建自定义占位符值。

如果我们想要匹配所有 `$name` 是 `string-number` 格式的值\(如上例所示\)的记录，我们可以使用如下正则表达式:

```text
^([a-z]+)-([0-9]+)$
```

请注意，在我们的示例中，我们使用括号表示数据的匹配组。如果模式与该值匹配，则将创建一个占位符，可以由 `NEW_TAG` 部分使用

如果 `$name` 值为 `abc-123`，如下**占位符**将被创建:

* `$0` = "abc-123"
* `$1` = "abc"
* `$2` = "123"

如果正则表达式与传入记录不匹配，则将跳过该规则，并使用下一个规则处理\(如果有\)。

### New\_Tag

如果正则表达式与规则中定义的键的值匹配，我们准备为该记录重新标记一个新的标签。标签可以是包含以下任意字符的字符串: `a-z`,`A-Z`,`0-9`,`.-,`。

标签可以是从匹配记录中获取任何字符串值，原始标签本身，环境变量或常规占位符。

考虑以下可以匹配规则的传入数据:

* Tag = aa.bb.cc
* Record =  `{"name": "abc-123", "ss": {"s1": {"s2": "flb"}}}`
* Environment variable $HOSTNAME = fluent

有了这些信息，我们可以为记录创建一个自定义的标签，如下所示:

```text
newtag.$TAG.$TAG[1].$1.$ss['s1']['s2'].out.${HOSTNAME}
```

预期生成的标签将是:

```text
newtag.aa.bb.cc.bb.abc.flb.out.fluent
```

我们可以使用占位符，记录内容和环境变量。

### Keep

如果规则符合条件，则过滤器将发出带有新标签的记录副本。KEEP 属性是一个布尔值，用于定义带有旧 Tag 的原始记录是否继续保留在管道中或被丢弃。

您可以使用 `true` 或 `false` 来决定预期的行为。该参数没有默认值，是规则中的必填字段。

## Configuration Example

如下配置示例将重新发出一个虚拟\(手动创建的\)记录，过滤器将重写标签，丢弃旧记录并将新记录打印到标准输出:

```python
[SERVICE]
    Flush     1
    Log_Level info

[INPUT]
    NAME   dummy
    Dummy  {"tool": "fluent", "sub": {"s1": {"s2": "bit"}}}
    Tag    test_tag

[FILTER]
    Name          rewrite_tag
    Match         test_tag
    Rule          $tool ^(fluent)$  from.$TAG.new.$tool.$sub['s1']['s2'].out false
    Emitter_Name  re_emitted

[OUTPUT]
    Name   stdout
    Match  from.*
```

原始标签 `test_tag` 将被重写为 `from.test_tag.new.fluent.bit.out`:

```bash
$ bin/fluent-bit -c example.conf
Fluent Bit v1.x.x
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

...
[0] from.test_tag.new.fluent.bit.out: [1580436933.000050569, {"tool"=>"fluent", "sub"=>{"s1"=>{"s2"=>"bit"}}}]
```

## Monitoring

如[监控](../../administration/monitoring.md)部分所述，Fluent Bit 管道的每个组件都公开指标。此过滤器公开的基本指标是 `drop_records` 和 `add_record`s，它们记录了传入数据块中已删除记录的总数或新添加的记录数。

由于 `rewrite_tag` 发出流水线开始处的新记录，因此它公开了一个名为 `emit_records` 的附加指标，该指标记录了发出记录的总数。

### Understanding the Metrics

使用上面提供的配置，如果我们查询 HTTP 接口中公开的数据指标，我们将看到以下内容:

执行如下命令:

```bash
$ curl  http://127.0.0.1:2020/api/v1/metrics/ | jq
```

将有如下数据指标输出:

```javascript
{
  "input": {
    "dummy.0": {
      "records": 2,
      "bytes": 80
    },
    "emitter_for_rewrite_tag.0": {
      "records": 1,
      "bytes": 40
    }
  },
  "filter": {
    "rewrite_tag.0": {
      "drop_records": 2,
      "add_records": 0,
      "emit_records": 2
    }
  },
  "output": {
    "stdout.0": {
      "proc_records": 1,
      "proc_bytes": 40,
      "errors": 0,
      "retries": 0,
      "retries_failed": 0
    }
  }
}
```

_`dummy`_ 输入插件生成两个记录，过滤器从数据块中删除两个记录，并发出两个新带有新标签的记录。

生成的记录由内部发射器处理,因此新纪录将在发射器指标中进行汇总,请查看名为 `emitter_for_rewrite_tag.0` 的条目。

### What is the Emitter ?

Emitter\(发射器\)是 Fluent Bit 内部插件，它允许管道的其他组件发出自定义记录。在这种情况下，`rewrite_tag` 创建了一个Emitter 实例，专门用于发射记录，通过这种方式,我们可以精确控制谁发出了什么。

通过设置上述的 `Emitter_Name` 配置属性可以更改数据指标中的发射器名称。

