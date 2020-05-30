# Modify

**modify** 过滤器插件允许您指定规则和条件更改记录。

## Example usage

以 JSON 格式的数据作为示例，执行如下操作

* 将 `Key2` 重命名为 `RenamedKey`
* 如果 `OtherKey` 不存在，则添加值为 `Value3` 的 `OtherKey` 记录映射

输入示例

```text
{
  "Key1"     : "Value1",
  "Key2"     : "Value2"
}
```

输出示例:

```text
{
  "Key1"       : "Value1",
  "RenamedKey" : "Value2",
  "OtherKey"   : "Value3"
}
```

## Configuration Parameters

### Rules

该插件支持以下规则:

| 操作 | 参数 1 | 参数 2 | 描述 |
| :--- | :--- | :--- | :--- |
| Set | STRING:KEY | STRING:VALUE | 添加带有指定 `KEY` 和 `VALUE` 的记录键值对映射。如果 `KEY` 已经存在，则**覆盖** |
| Add | STRING:KEY | STRING:VALUE | 如果 `KEY` 不存在，则添加带有指定 `KEY` 和 `VALUE` 的记录键值对映射 |
| Remove | STRING:KEY | NONE | 删除记录键值对中指定的 `KEY` |
| Remove\_wildcard | WILDCARD:KEY | NONE | 删除记录键值对中 `KEY` 通配符匹配的键值对 |
| Remove\_regex | REGEXP:KEY | NONE | 删除记录键值对中 `KEY` 通正则匹配的键值对 |
| Rename | STRING:KEY | STRING:RENAMED\_KEY | 如果 `KEY` 存在且 `RENAMED_KEY` 不存在，则将 `KEY` 重命名为 `RENAMED_KEY` |
| Hard\_rename | STRING:KEY | STRING:RENAMED\_KEY | 重命名 `KEY` 为 `RENAMED_KEY`。如果 `RENAMED_KEY` 已经存在，则**覆盖** |
| Copy | STRING:KEY | STRING:COPIED\_KEY | 如果 `KEY` 存在且 `COPIED_KEY` 不存在，则将 `KEY` 复制为 `COPIED_KEY` |
| Hard\_copy | STRING:KEY | STRING:COPIED\_KEY | 复制 `KEY` 为 `COPIED_KEY`。如果 `COPIED_KEY` 已经存在，则**覆盖** |

* 规则不区分大小写，参数区分大小写
* 过滤器实例中可以设置任意数量的规则
* 规则按照它们出现的顺序应用，每个规则都根据前一个规则的结果进行操作

### Conditions

该插件支持以下条件:

| 条件 | 参数 1 | 参数 2 | 描述 |
| :--- | :--- | :--- | :--- |
| Key\_exists | STRING:KEY | NONE | 指定 `KEY` 存在则为 `true` |
| Key\_does\_not\_exist | STRING:KEY | STRING:VALUE | 指定 `KEY` 不存在则为 `true` |
| A\_key\_matches | REGEXP:KEY | NONE | 匹配指定正则表达式 `KEY` 则为 `true` |
| No\_key\_matches | REGEXP:KEY | NONE | 不匹配指定正则表达式 `KEY` 则为 `true` |
| Key\_value\_equals | STRING:KEY | STRING:VALUE | 指定 `KEY` 存在且值为 `VALUE`,则为 `true` |
| Key\_value\_does\_not\_equal | STRING:KEY | STRING:VALUE | 指定 `KEY` 存在且值不为 `VALUE`,则为 `true` |
| Key\_value\_matches | STRING:KEY | REGEXP:VALUE | 指定 `KEY` 存在且值匹配正则表达式 `VALUE`,则为 `true` |
| Key\_value\_does\_not\_match | STRING:KEY | REGEXP:VALUE | 指定 `KEY` 存在且值不匹配正则表达式 `VALUE`,则为 `true` |
| Matching\_keys\_have\_matching\_values | REGEXP:KEY | REGEXP:VALUE | 所有匹配 `KEY` 的键都具有匹配 `VALUE` 的值,则为 `true` |
| Matching\_keys\_do\_not\_have\_matching\_values | REGEXP:KEY | REGEXP:VALUE | 所有匹配 `KEY` 的键都具有不匹配 `VALUE` 的值,则为 `true` |

* 条件不区分大小写，参数区分大小写
* 过滤器实例中可以设置任意数量的条件
* 条件适用于整个过滤器实例及其所有规则。而不是单个规则
* 要应用规则，所有条件都必须为 `true`\(条件间关系为与\)

## Example \#1 - Add and Rename

要开始记录过滤，您可以从命令行或通过配置文件运行过滤器。以下调用 [mem](../../pipleline/inputs/mem.md)\(内存数据指标\)输入插件，它输出如下示例记录:

```text
[0] memory: [1488543156, {"Mem.total"=>1016044, "Mem.used"=>841388, "Mem.free"=>174656, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
[1] memory: [1488543157, {"Mem.total"=>1016044, "Mem.used"=>841420, "Mem.free"=>174624, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
[2] memory: [1488543158, {"Mem.total"=>1016044, "Mem.used"=>841420, "Mem.free"=>174624, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
[3] memory: [1488543159, {"Mem.total"=>1016044, "Mem.used"=>841420, "Mem.free"=>174624, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
```

### Using command Line

> 注意: 使用命令行模式要求引号正确解析通配符。建议使用配置文件。

```text
bin/fluent-bit -i mem \
  -p 'tag=mem.local' \
  -F modify \
  -p 'Add=Service1 SOMEVALUE' \
  -p 'Add=Service2 SOMEVALUE3' \
  -p 'Add=Mem.total2 TOTALMEM2' \
  -p 'Rename=Mem.free MEMFREE' \
  -p 'Rename=Mem.used MEMUSED' \
  -p 'Rename=Swap.total SWAPTOTAL' \
  -p 'Add=Mem.total TOTALMEM' \
  -m '*' \
  -o stdout
```

### Configuration File

```python
[INPUT]
    Name mem
    Tag  mem.local

[OUTPUT]
    Name  stdout
    Match *

[FILTER]
    Name modify
    Match *
    Add Service1 SOMEVALUE
    Add Service3 SOMEVALUE3
    Add Mem.total2 TOTALMEM2
    Rename Mem.free MEMFREE
    Rename Mem.used MEMUSED
    Rename Swap.total SWAPTOTAL
    Add Mem.total TOTALMEM
```

### Result

命令行和配置调用的输出应相同，输出如下:

```text
[2018/04/06 01:35:13] [ info] [engine] started
[0] mem.local: [1522980610.006892802, {"Mem.total"=>4050908, "MEMUSED"=>738100, "MEMFREE"=>3312808, "SWAPTOTAL"=>1046524, "Swap.used"=>0, "Swap.free"=>1046524, "Service1"=>"SOMEVALUE", "Service3"=>"SOMEVALUE3", "Mem.total2"=>"TOTALMEM2"}]
[1] mem.local: [1522980611.000658288, {"Mem.total"=>4050908, "MEMUSED"=>738068, "MEMFREE"=>3312840, "SWAPTOTAL"=>1046524, "Swap.used"=>0, "Swap.free"=>1046524, "Service1"=>"SOMEVALUE", "Service3"=>"SOMEVALUE3", "Mem.total2"=>"TOTALMEM2"}]
[2] mem.local: [1522980612.000307652, {"Mem.total"=>4050908, "MEMUSED"=>738068, "MEMFREE"=>3312840, "SWAPTOTAL"=>1046524, "Swap.used"=>0, "Swap.free"=>1046524, "Service1"=>"SOMEVALUE", "Service3"=>"SOMEVALUE3", "Mem.total2"=>"TOTALMEM2"}]
[3] mem.local: [1522980613.000122671, {"Mem.total"=>4050908, "MEMUSED"=>738068, "MEMFREE"=>3312840, "SWAPTOTAL"=>1046524, "Swap.used"=>0, "Swap.free"=>1046524, "Service1"=>"SOMEVALUE", "Service3"=>"SOMEVALUE3", "Mem.total2"=>"TOTALMEM2"}]
```

## Example \#2 - Conditionally Add and Remove

### Configuration File

```python
[INPUT]
    Name mem
    Tag  mem.local
    Interval_Sec 1

[FILTER]
    Name    modify
    Match   mem.*

    Condition Key_Does_Not_Exist cpustats
    Condition Key_Exists Mem.used

    Set cpustats UNKNOWN

[FILTER]
    Name    modify
    Match   mem.*

    Condition Key_Value_Does_Not_Equal cpustats KNOWN

    Add sourcetype memstats

[FILTER]
    Name    modify
    Match   mem.*

    Condition Key_Value_Equals cpustats UNKNOWN

    Remove_wildcard Mem
    Remove_wildcard Swap
    Add cpustats_more STILL_UNKNOWN

[OUTPUT]
    Name           stdout
    Match          *
```

### Result

```text
[2018/06/14 07:37:34] [ info] [engine] started (pid=1493)
[0] mem.local: [1528925855.000223110, {"cpustats"=>"UNKNOWN", "sourcetype"=>"memstats", "cpustats_more"=>"STILL_UNKNOWN"}]
[1] mem.local: [1528925856.000064516, {"cpustats"=>"UNKNOWN", "sourcetype"=>"memstats", "cpustats_more"=>"STILL_UNKNOWN"}]
[2] mem.local: [1528925857.000165965, {"cpustats"=>"UNKNOWN", "sourcetype"=>"memstats", "cpustats_more"=>"STILL_UNKNOWN"}]
[3] mem.local: [1528925858.000152319, {"cpustats"=>"UNKNOWN", "sourcetype"=>"memstats", "cpustats_more"=>"STILL_UNKNOWN"}]
```

## Example \#3 - Emoji

### Configuration File

```python
[INPUT]
    Name mem
    Tag  mem.local

[OUTPUT]
    Name  stdout
    Match *

[FILTER]
    Name modify
    Match *

    Remove_Wildcard Mem
    Remove_Wildcard Swap
    Set This_plugin_is_on 🔥
    Set 🔥 is_hot
    Copy 🔥 💦
    Rename  💦 ❄️
    Set ❄️ is_cold
    Set 💦 is_wet
```

### Result

```text
[2018/06/14 07:46:11] [ info] [engine] started (pid=21875)
[0] mem.local: [1528926372.000197916, {"This_plugin_is_on"=>"🔥", "🔥"=>"is_hot", "❄️"=>"is_cold", "💦"=>"is_wet"}]
[1] mem.local: [1528926373.000107868, {"This_plugin_is_on"=>"🔥", "🔥"=>"is_hot", "❄️"=>"is_cold", "💦"=>"is_wet"}]
[2] mem.local: [1528926374.000181042, {"This_plugin_is_on"=>"🔥", "🔥"=>"is_hot", "❄️"=>"is_cold", "💦"=>"is_wet"}]
[3] mem.local: [1528926375.000090841, {"This_plugin_is_on"=>"🔥", "🔥"=>"is_hot", "❄️"=>"is_cold", "💦"=>"is_wet"}]
[0] mem.local: [1528926376.000610974, {"This_plugin_is_on"=>"🔥", "🔥"=>"is_hot", "❄️"=>"is_cold", "💦"=>"is_wet"}]
```

