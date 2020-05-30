# TCP & TLS

**tcp** 输出插件将记录发送到远程 TCP 服务器。payload 可以根据需要以不同的方式格式化。

## Configuration Parameters

| Key | Description | default |
| :--- | :--- | :--- |
| Host | Fluent Bit 或 Fluentd 监听转发消息的主机 | 127.0.0.1 |
| Port | 目标主机的 TCP 端口 | 5170 |
| Format | 指定输出的数据格式。支持的格式是 `msgpack`,`json`,`json_lines`,`json\_stream`\_. | msgpack |
| json\_date\_key | 在输出中指定日期字段的名称 | date |
| json\_date\_format | 指定日期格式。支持的格式为 `double`,`iso8601`\(如: _2018-05-30T09:39:52.000681Z_\) | double |

## TLS Configuration Parameters

以下参数可用于配置通过 TLS 加密的安全连接:

| Key | Description | Default |
| :--- | :--- | :--- |
| tls | 是否启用 TLS 连接 | Off |
| tls.verify | 是否强制证书认证 | On |
| tls.debug | 设置 TLS 调试级别。接受如下参数. 0 \(No debug\), 1 \(Error\), 2 \(State change\), 3 \(Informational\), 4 Verbose | 1 |
| tls.ca\_file | CA 证书文件的绝对路径 |  |
| tls.crt\_file | crt 证书文件的绝对路径. |  |
| tls.key\_file | 私钥文件的绝对路径 |  |
| tls.key\_passwd | 可选的私钥文件密码 |  |

### Command Line

```bash
$ bin/fluent-bit -i cpu -o tcp://127.0.0.1:5170 -p format=json_lines -v
```

我们使用 [CPU](../../pipleline/inputs/cpu.md) 输入插件收集 CPU 使用的数据指标，并以 JSON 格式将它们发送到以 netcat 监听的服务端点中，如:

#### Start the TCP listener

在终端中运行以下命令，netcat 将开始在 TCP/5170 端口上监听消息:

```text
$ nc -l 5170
```

启动 Fluent Bit:

```bash
$ bin/fluent-bit -i cpu -o stdout -p format=msgpack -v
Fluent Bit v1.x.x
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2016/10/07 21:52:01] [ info] [engine] started
[0] cpu.0: [1475898721, {"cpu_p"=>0.500000, "user_p"=>0.250000, "system_p"=>0.250000, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>0.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>0.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>1.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>1.000000}]
[1] cpu.0: [1475898722, {"cpu_p"=>0.250000, "user_p"=>0.250000, "system_p"=>0.000000, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>1.000000, "cpu1.p_user"=>1.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>0.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>0.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>0.000000}]
[2] cpu.0: [1475898723, {"cpu_p"=>0.750000, "user_p"=>0.250000, "system_p"=>0.500000, "cpu0.p_cpu"=>2.000000, "cpu0.p_user"=>1.000000, "cpu0.p_system"=>1.000000, "cpu1.p_cpu"=>0.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>1.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>1.000000, "cpu3.p_cpu"=>0.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>0.000000}]
[3] cpu.0: [1475898724, {"cpu_p"=>1.000000, "user_p"=>0.750000, "system_p"=>0.250000, "cpu0.p_cpu"=>1.000000, "cpu0.p_user"=>1.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>2.000000, "cpu1.p_user"=>1.000000, "cpu1.p_system"=>1.000000, "cpu2.p_cpu"=>1.000000, "cpu2.p_user"=>1.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>1.000000, "cpu3.p_user"=>1.000000, "cpu3.p_system"=>0.000000}]
```

No more, no less, it just works.

