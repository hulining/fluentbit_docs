# 构建和安装

[Fluent Bit](http://fluentbit.io) 使用 [CMake](http://cmake.org) 作为构建系统。建议的准备构建系统的过程包括以下步骤:

## 环境准备 <a id="prepare-environment"></a>

> 在以下步骤中，您可以找到使用默认选项构建和安装项目的正确命令。如果您已经知道 CMake 的工作原理，则可以跳过此部分并查看可用的构建选项。

转到 Fluent Bit 源代码中的 _`build/`_ 目录

```bash
$ cd build/
```

使用 [CMake](http://cmake.org) 配置项目以确定安装根路径的位置:

```bash
$ cmake ../
-- The C compiler identification is GNU 4.9.2
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- The CXX compiler identification is GNU 4.9.2
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
...
-- Could NOT find Doxygen (missing:  DOXYGEN_EXECUTABLE)
-- Looking for accept4
-- Looking for accept4 - not found
-- Configuring done
-- Generating done
-- Build files have been written to: /home/edsiper/coding/fluent-bit/build
```

现在您可以通过简单的 _`make`_ 命令开始编译过程:

```bash
$ make
Scanning dependencies of target msgpack
[  2%] Building C object lib/msgpack-1.1.0/CMakeFiles/msgpack.dir/src/unpack.c.o
[  4%] Building C object lib/msgpack-1.1.0/CMakeFiles/msgpack.dir/src/objectc.c.o
[  7%] Building C object lib/msgpack-1.1.0/CMakeFiles/msgpack.dir/src/version.c.o
...
[ 19%] Building C object lib/monkey/mk_core/CMakeFiles/mk_core.dir/mk_file.c.o
[ 21%] Building C object lib/monkey/mk_core/CMakeFiles/mk_core.dir/mk_rconf.c.o
[ 23%] Building C object lib/monkey/mk_core/CMakeFiles/mk_core.dir/mk_string.c.o
...
Scanning dependencies of target fluent-bit-static
[ 66%] Building C object src/CMakeFiles/fluent-bit-static.dir/flb_pack.c.o
[ 69%] Building C object src/CMakeFiles/fluent-bit-static.dir/flb_input.c.o
[ 71%] Building C object src/CMakeFiles/fluent-bit-static.dir/flb_output.c.o
...
Linking C executable ../bin/fluent-bit
[100%] Built target fluent-bit-bin
```

继续在系统上安装二进制文件，只需执行以下操作:

```bash
$ make install
```

您可能需要 root 权限，因此可以尝试在命令前加上 _`sudo`_。

## 构建选项 <a id="build-options"></a>

Fluent Bit 提供了 CMake 的某些选项，这些选项可以在配置时启用或禁用，请参考下面的 _General Options_, _Development Options_, _Input Plugins_ 和 _Output Plugins_ 部分。

### General Options

| 选项 | 描述 | 默认值 |
| :--- | :--- | :--- |
| FLB\_ALL | 启用所有可用功能 | No |
| FLB\_JEMALLOC | 使用 Jemalloc 为默认的内存分配器 | No |
| FLB\_TLS | SSL/TLS 支持 | No |
| FLB\_BINARY | 生成可执行文件 | Yes |
| FLB\_EXAMPLES | 生成示例 | Yes |
| FLB\_SHARED\_LIB | 构建共享库 | Yes |
| FLB\_MTRACE | 启用 mtrace 支持 | No |
| FLB\_INOTIFY | 启用 Inotify 支持 | Yes |
| FLB\_POSIX\_TLS | 强制 POSIX 线程存储 | No |
| FLB\_SQLDB | 启用 SQL 嵌入式数据库支持 | No |
| FLB\_HTTP\_SERVER | 启用 HTTP 服务器 | No |
| FLB\_LUAJIT | 启用 Lua 脚本支持 | Yes |
| FLB\_RECORD\_ACCESSOR | 启用记录访问器 | Yes |
| FLB\_SIGNV4 | 启用 AWS Signv4 支持 | Yes |
| FLB\_STATIC\_CONF | 使用静态配置文件构建二进制文件。此选项的值必须是包含配置文件的目录. |  |
| FLB\_STREAM\_PROCESSOR | 启用流处理器 | Yes |

### Development Options

| 选项 | 描述 | 默认值 |
| :--- | :--- | :--- |
| FLB\_DEBUG | 使用 debug 模式构建二进制文件 | No |
| FLB\_VALGRIND | 启用 Valgrind 支持 | No |
| FLB\_TRACE | 启用跟踪模式 | No |
| FLB\_SMALL | 最小化二进制大小 | No |
| FLB\_TESTS\_RUNTIME | 启用运行时测试 | No |
| FLB\_TESTS\_INTERNAL | 启用内部测试 | No |
| FLB\_TESTS | 启用测试 | No |
| FLB\_BACKTRACE | 启用反向跟踪/堆栈跟踪支持 | Yes |

### Input Plugins

_输出插件_ 提供了从特定的源类型\(可以是网络接口，某些内置指标或通过特定的输入设备\)收集信息的某些功能，可以使用以下输入插件:

| 选项 | 描述 | 默认值 |
| :--- | :--- | :--- |
| [FLB\_IN\_COLLECTD](../../pipeline/inputs/collectd.md) | Enable Collectd input plugin | On |
| [FLB\_IN\_CPU](../../pipeline/inputs/cpu-metrics.md) | Enable CPU input plugin | On |
| [FLB\_IN\_DISK](../../pipeline/inputs/disk-io-metrics.md) | Enable Disk I/O Metrics input plugin | On |
| [FLB\_IN\_DOCKER](../docker.md) | Enable Docker metrics input plugin | On |
| [FLB\_IN\_EXEC](../../pipeline/inputs/exec.md) | Enable Exec input plugin | On |
| [FLB\_IN\_FORWARD](../../pipeline/inputs/forward.md) | Enable Forward input plugin | On |
| [FLB\_IN\_HEAD](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/input/head.md) | Enable Head input plugin | On |
| [FLB\_IN\_HEALTH](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/input/health.md) | Enable Health input plugin | On |
| [FLB\_IN\_KMSG](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/input/kmsg.md) | Enable Kernel log input plugin | On |
| [FLB\_IN\_MEM](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/input/mem.md) | Enable Memory input plugin | On |
| [FLB\_IN\_MQTT](../../pipeline/inputs/mqtt.md) | Enable MQTT Server input plugin | On |
| [FLB\_IN\_NETIF](../../pipeline/inputs/network-io-metrics.md) | Enable Network I/O metrics input plugin | On |
| [FLB\_IN\_PROC](../../pipeline/inputs/process.md) | Enable Process monitoring input plugin | On |
| [FLB\_IN\_RANDOM](../../pipeline/inputs/random.md) | Enable Random input plugin | On |
| [FLB\_IN\_SERIAL](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/input/serial.md) | Enable Serial input plugin | On |
| [FLB\_IN\_STDIN](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/input/stdin.md) | Enable Standard input plugin | On |
| [FLB\_IN\_SYSLOG](../../pipeline/inputs/syslog.md) | Enable Syslog input plugin | On |
| [FLB\_IN\_SYSTEMD](../../pipeline/inputs/systemd.md) | Enable Systemd / Journald input plugin | On |
| [FLB\_IN\_TAIL](../../pipeline/inputs/tail.md) | Enable Tail \(follow files\) input plugin | On |
| [FLB\_IN\_TCP](../../pipeline/inputs/tcp.md) | Enable TCP input plugin | On |
| [FLB\_IN\_THERMAL](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/input/thermal.md) | Enable system temperature\(s\) input plugin | On |
| [FLB\_IN\_WINLOG](../../pipeline/inputs/windows-event-log.md) | Enable Windows Event Log input plugin \(Windows Only\) | On |

### Filter Plugins

_Filter plugins_ 允许修改，丰富或删除记录。下表描述了此版本中可用的过滤器:

| 选项 | 描述 | 默认值 |
| :--- | :--- | :--- |
| [FLB\_FILTER\_AWS](../../pipeline/filters/aws-metadata.md) | Enable AWS metadata filter | On |
| FLB\_FILTER\_EXPECT | Enable Expect data test filter | On |
| [FLB\_FILTER\_GREP](../../pipeline/filters/grep.md) | Enable Grep filter | On |
| [FLB\_FILTER\_KUBERNETES](../../pipeline/filters/kubernetes.md) | Enable Kubernetes metadata filter | On |
| [FLB\_FILTER\_LUA](../../pipeline/filters/lua.md) | Enable Lua scripting filter | On |
| [FLB\_FILTER\_MODIFY](../../pipeline/filters/modify.md) | Enable Modify filter | On |
| [FLB\_FILTER\_NEST](../../pipeline/filters/nest.md) | Enable Nest filter | On |
| [FLB\_FILTER\_PARSER](../../pipeline/filters/parser.md) | Enable Parser filter | On |
| [FLB\_FILTER\_RECORD\_MODIFIER](../../pipeline/filters/record-modifier.md) | Enable Record Modifier filter | On |
| [FLB\_FILTER\_REWRITE\_TAG](../../pipeline/filters/rewrite-tag.md) | Enable Rewrite Tag filter | On |
| [FLB\_FILTER\_STDOUT](../../pipeline/filters/standard-output.md) | Enable Stdout filter | On |
| [FLB\_FILTER\_THROTTLE](../../pipeline/filters/throttle.md) | Enable Throttle filter | On |

### Output Plugins

_Output plugins_ 提供将信息刷新到某些外部接口，服务或终端的能力，下表描述了此版本可用的输出插件

| 选项 | 描述 | 默认值 |
| :--- | :--- | :--- |
| [FLB\_OUT\_AZURE](../../pipeline/outputs/azure.md) | Enable Microsoft Azure output plugin | On |
| [FLB\_OUT\_BIGQUERY](../../pipeline/outputs/bigquery.md) | Enable Google BigQuery output plugin | On |
| [FLB\_OUT\_COUNTER](../../pipeline/outputs/counter.md) | Enable Counter output plugin | On |
| [FLB\_OUT\_DATADOG](../../pipeline/outputs/datadog.md) | Enable Datadog output plugin | On |
| [FLB\_OUT\_ES](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/output/elasticsearch.md) | Enable [Elastic Search](http://www.elastic.co) output plugin | On |
| [FLB\_OUT\_FILE](../../pipeline/outputs/file.md) | Enable File output plugin | On |
| [FLB\_OUT\_FLOWCOUNTER](../../pipeline/outputs/flowcounter.md) | Enable Flowcounter output plugin | On |
| [FLB\_OUT\_FORWARD](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/output/forward.md) | Enable [Fluentd](http://www.fluentd.org) output plugin | On |
| [FLB\_OUT\_GELF](../../pipeline/outputs/gelf.md) | Enable Gelf output plugin | On |
| [FLB\_OUT\_HTTP](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/output/http.md) | Enable HTTP output plugin | On |
| [FLB\_OUT\_INFLUXDB](../../pipeline/outputs/influxdb.md) | Enable InfluxDB output plugin | On |
| [FLB\_OUT\_KAFKA](../../pipeline/outputs/kafka.md) | Enable Kafka output | Off |
| [FLB\_OUT\_KAFKA\_REST](../../pipeline/outputs/kafka-rest-proxy.md) | Enable Kafka REST Proxy output plugin | On |
| FLB\_OUT\_LIB | Enable Lib output plugin | On |
| [FLB\_OUT\_NATS](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/output/nats.md) | Enable [NATS](http://www.nats.io) output plugin | Off |
| FLB\_OUT\_NULL | Enable NULL output plugin | On |
| FLB\_OUT\_PGSQL | Enable PostgreSQL output plugin | On |
| FLB\_OUT\_PLOT | Enable Plot output plugin | On |
| FLB\_OUT\_SLACK | Enable Slack output plugin | On |
| [FLB\_OUT\_SPLUNK](../../pipeline/outputs/splunk.md) | Enable Splunk output plugin | On |
| [FLB\_OUT\_STACKDRIVER](../../pipeline/outputs/stackdriver.md) | Enable Google Stackdriver output plugin | On |
| [FLB\_OUT\_STDOUT](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/output/stdout.md) | Enable STDOUT output plugin | On |
| FLB\_OUT\_TCP | Enable TCP/TLS output plugin | On |
| [FLB\_OUT\_TD](https://github.com/fluent/fluent-bit-docs/tree/8ab2f4cda8dfdd8def7fa0cf5c7ffc23069e5a70/installation/output/td.md) | Enable [Treasure Data](http://www.treasuredata.com) output plugin | On |

