# JSON

JSON 解析器是最简单的选择: 如果原始日志源是 JSON 格式的字符串，它将采用其结构并将其直接转换为内部二进制表示形式。

在默认解析器配置文件中可以找到一个简单的配置，该记录是解析 Docker 日志文件的记录\(当使用 tail 输入插件时\)：

```python
[PARSER]
    Name        docker
    Format      json
    Time_Key    time
    Time_Format %Y-%m-%dT%H:%M:%S %z
```

对于上述定义的解析器，以下日志记录是的有效内容:

```javascript
{"key1": 12345, "key2": "abc", "time": "2006-07-28T13:22:04Z"}
```

After processing, it internal representation will be:

```text
[1154103724, {"key1"=>12345, "key2"=>"abc"}]
```

time 字段已转换为 Unix 时间戳\(UTC\)，并且映射由原始消息转换为每个组成部分

