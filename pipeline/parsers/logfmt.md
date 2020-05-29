# Logfmt

**logfmt** 解析器允许解析 [https://brandur.org/logfmt](https://brandur.org/logfmt) 中描述的 logfmt 格式。更好的描述在 [https://godoc.org/github.com/kr/logfmt](https://godoc.org/github.com/kr/logfmt) 中。

如下是一个配置的示例:

```python
[PARSER]
    Name        logfmt
    Format      logfmt
```

对于上述定义的解析器来说，以下日志是有效的:

```text
key1=val1 key2=val2
```

处理后，在内部表示将为:

```text
[1540936693, {"key1"=>"val1",
              "key2"=>"val2"}]
```

