# Nest

**nest** 过滤器插件允许您对嵌套数据进行操作。它的操作方式是:

* `nest` - 获取一组记录并将其放置在映射中
* `lift` - 通过映射的键获取其值并将其提取为记录

## Example usage \(nest\)

以使用 JSON 形式的数据为例，要将带有通配符的键 `Key*` 嵌套在新键 `NestKey` 下，转换前后如下:

输入示例:

```text
{
  "Key1"     : "Value1",
  "Key2"     : "Value2",
  "OtherKey" : "Value3"
}
```

输出示例

```text
{
  "OtherKey" : "Value3"
  "NestKey"  : {
    "Key1"     : "Value1",
    "Key2"     : "Value2",
  }
}
```

## Example usage \(lift\)

以使用 JSON 形式的数据为例，要提取带有嵌套数据的键 `NestKey*` 中的值\(提取为记录\)，转换前后如下: 输入示例:

```text
{
  "OtherKey" : "Value3"
  "NestKey"  : {
    "Key1"     : "Value1",
    "Key2"     : "Value2",
  }
}
```

输出示例:

```text
{
  "Key1"     : "Value1",
  "Key2"     : "Value2",
  "OtherKey" : "Value3"
}
```

## Configuration Parameters

该插件支持如下配置参数:

| Key | Value Format | Operation | Description | 中文 |
| :--- | :--- | :--- | :--- | :--- |
| Operation | ENUM \[`nest` or `lift`\] |  | Select the operation `nest` or `lift` | 可选的操作为 `nest` 或 `lift`\(嵌套或提取\) |
| Wildcard | FIELD WILDCARD | `nest` | Nest records which field matches the wildcard | 带有通配符的字符串,用于筛选记录映射字段用于嵌套或提取 |
| Nest\_under | FIELD STRING | `nest` | Nest records matching the `Wildcard` under this key | 将 `Wildcard` 匹配到的键映射嵌套在 `Nest_under` 指定的键下 |
| Nested\_under | FIELD STRING | `lift` | Lift records nested under the `Nested_under` key | 提取 `Nested_under` 指定的键下的键值映射作为记录的键值映射 |
| Add\_prefix | FIELD STRING | ANY | Prefix affected keys with this string | 为受影响的键添加前缀 |
| Remove\_prefix | FIELD STRING | ANY | Remove prefix from affected keys if it matches this string | 从受影响的键中删除匹配的前缀 |

## Getting Started

要开始记录过滤，您可以从命令行或通过配置文件运行过滤器。以下调用 [mem](../../pipleline/inputs/mem.md)\(内存数据指标\)输入插件，它输出如下示例记录:

```text
[0] memory: [1488543156, {"Mem.total"=>1016044, "Mem.used"=>841388, "Mem.free"=>174656, "Swap.total"=>2064380, "Swap.used"=>139888, "Swap.free"=>1924492}]
```

## Example \#1 - nest

### Command Line

> 注意：使用命令行模式要求引号正确解析通配符。建议使用配置文件。

以下命令将加载 _mem_ 输入插件。然后 _nest_ 过滤器会使用通配符规则与关键字匹配，并将与 `Mem.*` 匹配的关键字嵌套在新关键字 `NEST` 下:

```text
$ bin/fluent-bit -i mem -p 'tag=mem.local' -F nest -p 'Operation=nest' -p 'Wildcard=Mem.*' -p 'Nest_under=Memstats' -p 'Remove_prefix=Mem.' -m '*' -o stdout
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
    Name nest
    Match *
    Operation nest
    Wildcard Mem.*
    Nest_under Memstats
    Remove_prefix Mem.
```

### Result

命令行和配置调用的输出应相同，输出如下:

```text
[2018/04/06 01:35:13] [ info] [engine] started
[0] mem.local: [1522978514.007359767, {"Swap.total"=>1046524, "Swap.used"=>0, "Swap.free"=>1046524, "Memstats"=>{"total"=>4050908, "used"=>714984, "free"=>3335924}}]
```

## Example \#1 - nest and lift undo

本示例将所有 `Mem.*` 和 `Swap.*` 键嵌套在 `Stats` 键下，然后通过 `lift` 操作撤消这些操作。输出显示不变。

### Configuration File

```python
[INPUT]
    Name mem
    Tag  mem.local

[OUTPUT]
    Name  stdout
    Match *

[FILTER]
    Name nest
    Match *
    Operation nest
    Wildcard Mem.*
    Wildcard Swap.*
    Nest_under Stats
    Add_prefix NESTED

[FILTER]
    Name nest
    Match *
    Operation lift
    Nested_under Stats
    Remove_prefix NESTED
```

### Result

```text
[2018/06/21 17:42:37] [ info] [engine] started (pid=17285)
[0] mem.local: [1529566958.000940636, {"Mem.total"=>8053656, "Mem.used"=>6940380, "Mem.free"=>1113276, "Swap.total"=>16532988, "Swap.used"=>1286772, "Swap.free"=>15246216}]
```

## Example \#2 - nest 3 levels deep

本示例将以 `Mem.*` 开头的键嵌套在 `LAYER1` 下，然后将其自身嵌套在 `LAYER2` 下，后者又嵌套在 `LAYER3` 下。

### Configuration File

```python
[INPUT]
    Name mem
    Tag  mem.local

[OUTPUT]
    Name  stdout
    Match *

[FILTER]
    Name nest
    Match *
    Operation nest
    Wildcard Mem.*
    Nest_under LAYER1

[FILTER]
    Name nest
    Match *
    Operation nest
    Wildcard LAYER1*
    Nest_under LAYER2

[FILTER]
    Name nest
    Match *
    Operation nest
    Wildcard LAYER2*
    Nest_under LAYER3
```

### Result

```text
[0] mem.local: [1524795923.009867831, {"Swap.total"=>1046524, "Swap.used"=>0, "Swap.free"=>1046524, "LAYER3"=>{"LAYER2"=>{"LAYER1"=>{"Mem.total"=>4050908, "Mem.used"=>1112036, "Mem.free"=>2938872}}}}]


{
  "Swap.total"=>1046524,
  "Swap.used"=>0,
  "Swap.free"=>1046524,
  "LAYER3"=>{
    "LAYER2"=>{
      "LAYER1"=>{
        "Mem.total"=>4050908,
        "Mem.used"=>1112036,
        "Mem.free"=>2938872
      }
    }
  }
}
```

## Example \#3 - multiple nest and lift filters with prefix

本示例以 [Example 2](nest.md#example-2-nest-3-levels-deep) 的 3 层深度嵌套开始，并应用 3 次 `lift` 过滤器以反转操作。最终结果是所有记录再次位于顶层，没有嵌套层级。且为每个提升的级别添加一个前缀。

### Configuration file

```python
[INPUT]
    Name mem
    Tag  mem.local

[OUTPUT]
    Name  stdout
    Match *

[FILTER]
    Name nest
    Match *
    Operation nest
    Wildcard Mem.*
    Nest_under LAYER1

[FILTER]
    Name nest
    Match *
    Operation nest
    Wildcard LAYER1*
    Nest_under LAYER2

[FILTER]
    Name nest
    Match *
    Operation nest
    Wildcard LAYER2*
    Nest_under LAYER3

[FILTER]
    Name nest
    Match *
    Operation lift
    Nested_under LAYER3
    Add_prefix Lifted3_

[FILTER]
    Name nest
    Match *
    Operation lift
    Nested_under Lifted3_LAYER2
    Add_prefix Lifted3_Lifted2_

[FILTER]
    Name nest
    Match *
    Operation lift
    Nested_under Lifted3_Lifted2_LAYER1
    Add_prefix Lifted3_Lifted2_Lifted1_
```

### Result

```text
[0] mem.local: [1524862951.013414798, {"Swap.total"=>1046524, "Swap.used"=>0, "Swap.free"=>1046524, "Lifted3_Lifted2_Lifted1_Mem.total"=>4050908, "Lifted3_Lifted2_Lifted1_Mem.used"=>1253912, "Lifted3_Lifted2_Lifted1_Mem.free"=>2796996}]


{
  "Swap.total"=>1046524,
  "Swap.used"=>0,
  "Swap.free"=>1046524,
  "Lifted3_Lifted2_Lifted1_Mem.total"=>4050908,
  "Lifted3_Lifted2_Lifted1_Mem.used"=>1253912,
  "Lifted3_Lifted2_Lifted1_Mem.free"=>2796996
}
```

