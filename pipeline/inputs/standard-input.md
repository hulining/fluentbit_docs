# 标准输入

**stdin** 标准输入插件允许通过标准输入接口\(stdin\)检索有效的 JSON 文本消息。要使用它，请指定插件名称作为输入，例如:

```bash
$ fluent-bit -i stdin -o stdout
```

_stdin_ 插件可以识别以下 JSON 数据格式的输入数据:

```bash
1. { map => val, map => val, map => val }
2. [ time, { map => val, map => val, map => val } ]
```

如下是一个更好的示例演示它是如何工作的，通过一个 Bash 脚本生成消息并将其写入 [Fluent Bit](http://fluentbit.io)。 将以下内容写入名为 _test.sh_ 的文件中:

```bash
#!/bin/sh

while :; do
  echo -n "{\"key\": \"some value\"}"
  sleep 1
done
```

授予脚本执行权限:

```bash
$ chmod 755 test.sh
```

现在，通过以下方式启动脚本和 [Fluent Bit](http://fluentbit.io):

```bash
$ ./test.sh | fluent-bit -i stdin -o stdout
Fluent Bit v1.x.x
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2016/10/07 21:44:46] [ info] [engine] started
[0] stdin.0: [1475898286, {"key"=>"some value"}]
[1] stdin.0: [1475898287, {"key"=>"some value"}]
[2] stdin.0: [1475898288, {"key"=>"some value"}]
[3] stdin.0: [1475898289, {"key"=>"some value"}]
[4] stdin.0: [1475898290, {"key"=>"some value"}]
```

