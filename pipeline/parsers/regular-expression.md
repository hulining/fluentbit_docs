# Regular Expression

**regex** 正则表达式解析器使用自定义的 Ruby 正则表达式，使用名称捕获来定义哪些内容属于哪个字段名。

Fluent Bit 使用 Ruby 模式的 [Onigmo](https://github.com/k-takata/Onigmo) 正则表达式库，您可以使用以下 Web 编辑器来测试您的表达式:

[http://rubular.com/](http://rubular.com/)

{% hint style="info" %}
重要提示: 如果您使用的是 [Tail](../inputs/tail.md) 输入插件，请不要尝试在正则表达式中添加多行支持，因为每行都作为单独的实例进行处理。而是使用 [Tail Multiline](../inputs/tail.md#multiline) 支持的多行功能。
{% endhint %}

> 注意: 了解正则表达式的工作原理超出了本内容的范围

从配置的角度来看，当 `format` 配置项设置为 **regex** 时，`Regex` 配置项也必须存在。

以下解析器配置示例提供了可应用于 Apache HTTP Server 日志记录的规则:

```python
[PARSER]
    Name   apache
    Format regex
    Regex  ^(?<host>[^ ]*) [^ ]* (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^\"]*?)(?: +\S*)?)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)")?$
    Time_Key time
    Time_Format %d/%b/%Y:%H:%M:%S %z
```

例如，使用以下 Apache HTTP Server 日志记录:

```text
192.168.2.20 - - [29/Jul/2015:10:27:10 -0300] "GET /cgi-bin/try/ HTTP/1.0" 200 3395
```

以上内容没有为 Fluent Bit 提供结构化消息，但是启用适当的解析器后，我们可以对其进行结构化表示:

```text
[1154104030, {"host"=>"192.168.2.20",
              "user"=>"-",
              "method"=>"GET",
              "path"=>"/cgi-bin/try/",
              "code"=>"200",
              "size"=>"3395",
              "referer"=>"",
              "agent"=>""
              }
]
```

一个常见的陷阱是，字段名中不能使用字母，数字和下划线以外的字符。例如，像 `(?<user-name>.*)` 之类的字段名将由于包含无效字符\(`-`\)而导致错误。

为了像上述示例一样理解，学习和测试正则表达式，建议您尝试如下 Ruby 正则表达式编辑器: [http://rubular.com/r/X7BH0M4Ivm](http://rubular.com/r/X7BH0M4Ivm)

