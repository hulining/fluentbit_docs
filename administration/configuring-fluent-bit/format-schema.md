# 格式与模式

Fluent Bit 可以选择使用配置文件来定义服务的行为，在继续之前，我们需要了解配置模式的工作方式。

该配置模式由三个概念定义:

* Sections\(配置段\)
* Entries\(配置项\): Key/Value\(键/值\)
* Indented Configuration Mode\(缩进配置模式\)

配置文件的简单示例如下:

```python
[SERVICE]
    # 注释行
    Daemon    off
    log_level debug
```

## Sections <a id="sections"></a>

配置段由方括号内的名称或标题定义。例如上面的示例，使用 **`[SERVICE]`** 定义设置了 Service 配置段。配置段规则如下:

* 所有节内容必须缩进\(理想情况下为 4 个空格\)
* 同一文件中可存在多个配置段
* 配置段应该包含注释和配置项目，不能为空
* 配置段下的任何注释行也必须缩进

## Entries: Key/Value <a id="entries_kv"></a>

配置段中包含**配置项**，一个配置项是由一行包含**键**和**值**的文本定义的，在上面的示例中，`[SERVICE]` 配置段包含两个配置项，一个是键 **`Daemon`**，值 **`off`**，另一个是键 **`Log_Level`**，值 **`debug`**。配置项的规则如下:

* 配置项由键值对定义
* 键必须缩进
* 键必须包含以换行符结尾的值
* 可存在多个具有相同名称的键

注释行为 **`#`** 字符的前缀，这些行不做任何处理，但也必须缩进。

## Indented Configuration Mode <a id="indented_mode"></a>

Fluent Bit 配置文件遵循严格的**缩进模式**，这意味着每个配置文件在编写文本时必须遵循从左到右的相同对齐方式。 默认情况下，一个缩进建议包含 4 个空格。例如:

```python
[FIRST_SECTION]
    # This is a commented line
    Key1  some value
    Key2  another value
    # more comments

[SECOND_SECTION]
    KeyN  3.14
```

如您所见，有两个包含多个配置项和注释的配置段，还请注意，配置中允许空行，且不需要缩进。

