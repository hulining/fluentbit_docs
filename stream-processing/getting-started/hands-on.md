# Hands On! 101

本文通过非常具体且简单的步骤来学习流处理器的工作原理。为了简单起见，我们使用了一个自定义 Docker 镜像，其中包含测试需要的相关组件。

## Requirements

以下教程需要以下软件组件:

* [Fluent Bit](https://fluentbit.io) &gt;= v1.2.0
* [Docker Engine](https://www.docker.com/products/docker-engine) \(如果您的系统中已经安装了 Fluent Bit 二进制文件，则此项是非必须的\)

另外，下载以下数据样本文件\(130KB\):

* [https://fluentbit.io/samples/sp-samples-1k.log](https://fluentbit.io/samples/sp-samples-1k.log)

## 在命令行进行流处理

接下来的所有步骤，我们将从命令行运行 Fluent Bit，为简单起见，我们将使用官方的 Docker 镜像。

### 1. 查看 Fluent Bit 版本 <a id="1-fluent-bit-version"></a>

```bash
$ docker run -ti fluent/fluent-bit:1.4 /fluent-bit/bin/fluent-bit --version
Fluent Bit v1.4.0
```

### 2. 解析样例日志文件 <a id="2-parse-sample-files"></a>

样本文件包含 JSON 记录。在此命令中，我们将添加解析器配置文件，并指示 _tail_ 输入插件将内容解析为 _json_:

```bash
$ docker run -ti -v `pwd`/sp-samples-1k.log:/sp-samples-1k.log      \
     fluent/fluent-bit:1.4                                          \
     /fluent-bit/bin/fluent-bit -R /fluent-bit/etc/parsers.conf     \
                                -i tail -p path=/sp-samples-1k.log  \
                                        -p parser=json              \
                                -o stdout -f 1
```

上面的命令将把已解析的内容打印到标准输出。内容将打印与每个记录关联的 _Tag_ 以及具有两个字段的数组: 记录时间戳和记录的键值对映射

```text
Fluent Bit v1.4.0
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2019/05/08 13:34:16] [ info] [storage] initializing...
[2019/05/08 13:34:16] [ info] [storage] in-memory
[2019/05/08 13:34:16] [ info] [storage] normal synchronization mode, checksum disabled
[2019/05/08 13:34:16] [ info] [engine] started (pid=1)
[2019/05/08 13:34:16] [ info] [sp] stream processor started
[0] tail.0: [1557322456.315513208, {"date"=>"22/abr/2019:12:43:51 -0600", "ip"=>"73.113.230.135", "word"=>"balsamine", "country"=>"Japan", "flag"=>false, "num"=>96}]
[1] tail.0: [1557322456.315525280, {"date"=>"22/abr/2019:12:43:52 -0600", "ip"=>"242.212.128.227", "word"=>"inappendiculate", "country"=>"Chile", "flag"=>false, "num"=>15}]
[2] tail.0: [1557322456.315532364, {"date"=>"22/abr/2019:12:43:52 -0600", "ip"=>"85.61.182.212", "word"=>"elicits", "country"=>"Argentina", "flag"=>true, "num"=>73}]
[3] tail.0: [1557322456.315538969, {"date"=>"22/abr/2019:12:43:52 -0600", "ip"=>"124.192.66.23", "word"=>"Dwan", "country"=>"Germany", "flag"=>false, "num"=>67}]
[4] tail.0: [1557322456.315545150, {"date"=>"22/abr/2019:12:43:52 -0600", "ip"=>"18.135.244.142", "word"=>"chesil", "country"=>"Argentina", "flag"=>true, "num"=>19}]
[5] tail.0: [1557322456.315550927, {"date"=>"22/abr/2019:12:43:52 -0600", "ip"=>"132.113.203.169", "word"=>"fendered", "country"=>"United States", "flag"=>true, "num"=>53}]
```

到目前为止，还没有流处理，在步骤 \#3 中，我们将开始进行一些基本查询。

### 3. 选择带有指定键的记录 <a id="3-selecting-specific-record-keys"></a>

如下命令通过 **-T** 选项引入一个流处理器\(SP\)查询，并将输出插件更改为 _null_，其目的是在标准输出接口中获得 流处理结果，避免混淆。

```bash
$ docker run -ti -v `pwd`/sp-samples-1k.log:/sp-samples-1k.log           \
     fluent/fluent-bit:1.2                                               \
     /fluent-bit/bin/fluent-bit                                          \
         -R /fluent-bit/etc/parsers.conf                                 \
         -i tail                                                         \
             -p path=/sp-samples-1k.log                                  \
             -p parser=json                                              \
         -T "SELECT word, num FROM STREAM:tail.0 WHERE country='Chile';" \
         -o null -f 1
```

上面的查询旨在查询 _country_ 键的值与 _Chile_ 匹配的所有记录，并且对于每个匹配记录，仅使用 _word_ 和 _num_ 键字段组成一条记录并输出:

```text
[0] [1557322913.263534, {"word"=>"Candide", "num"=>94}]
[0] [1557322913.263581, {"word"=>"delightfulness", "num"=>99}]
[0] [1557322913.263607, {"word"=>"effulges", "num"=>63}]
[0] [1557322913.263690, {"word"=>"febres", "num"=>21}]
[0] [1557322913.263706, {"word"=>"decasyllables", "num"=>76}]
```

### 4. 计算平均值 <a id="4-calculate-average-value"></a>

以下查询与上一步中的查询类似，但这次我们将使用 AVG\(\) 的聚合函数来获取输入记录的平均值:

```bash
$ docker run -ti -v `pwd`/sp-samples-1k.log:/sp-samples-1k.log           \
     fluent/fluent-bit:1.2                                               \
     /fluent-bit/bin/fluent-bit                                          \
         -R /fluent-bit/etc/parsers.conf                                 \
         -i tail                                                         \
             -p path=/sp-samples-1k.log                                  \
             -p parser=json                                              \
         -T "SELECT AVG(num) FROM STREAM:tail.0 WHERE country='Chile';"  \
         -o null -f 1
```

输出:

```text
[0] [1557323573.940149, {"AVG(num)"=>61.230770}]
[0] [1557323573.941890, {"AVG(num)"=>47.842106}]
[0] [1557323573.943544, {"AVG(num)"=>40.647060}]
[0] [1557323573.945086, {"AVG(num)"=>56.812500}]
[0] [1557323573.945130, {"AVG(num)"=>99.000000}]
```

为什么我们得到了多个记录？答案是: 当 Fluent Bit 处理数据时，记录以数据块的方式输入，流处理器对数据块进行处理，输入插件提取了 5 个记录块，流处理器会独立地处理每个块的查询。要一次处理多个块，我们必须在窗口时间内对结果进行分组。

### 5. 对结果分组和窗口 <a id="5-grouping-results-and-window"></a>

结果分组旨在简化数据处理，且在定义的时间范围内使用时，我们可以完成很多事情。如下查询示例按 _country_ 关键字将结果分组，并计算 _num_ 值的平均值，处理窗口时间为 1 秒意味着: 处理 1 秒窗口时间内传入的所有数据块:

```bash
$ docker run -ti -v `pwd`/sp-samples-1k.log:/sp-samples-1k.log      \
     fluent/fluent-bit:1.2                                          \
     /fluent-bit/bin/fluent-bit                                     \
         -R /fluent-bit/etc/parsers.conf                            \
         -i tail                                                    \
             -p path=/sp-samples-1k.log                             \
             -p parser=json                                         \
         -T "SELECT country, AVG(num) FROM STREAM:tail.0            \
             WINDOW TUMBLING (1 SECOND)                             \
             WHERE country='Chile'                                  \
             GROUP BY country;"                                     \
         -o null -f 1
```

输出:

```text
[0] [1557324239.003211, {"country"=>"Chile", "AVG(num)"=>53.164558}]
```

### 6. 接收流处理结果作为新的数据流 <a id="6-ingest-stream-processor-results-as-new-stream-of-data"></a>

现在，我们看到了一个更真实的用例。将数据流处理结果发送到标准输出对于学习很有帮助，现在我们将指示流处理器将结果作为输入，成为 Fluent Bit 数据管道的一部分，并在其上附加一个 Tag。

可以使用 **`CREATE STREAM`** 语句完成此操作，该语句还将使用 **sp-results** 作为结果的标签。请注意，现在输出插件为 _stdout_，且带有匹配所有带有 _sp-results_ 标签记录的参数:

```bash
$ docker run -ti -v `pwd`/sp-samples-1k.log:/sp-samples-1k.log      \
     fluent/fluent-bit:1.2                                          \
     /fluent-bit/bin/fluent-bit                                     \
         -R /fluent-bit/etc/parsers.conf                            \
         -i tail                                                    \
             -p path=/sp-samples-1k.log                             \
             -p parser=json                                         \
         -T "CREATE STREAM results WITH (tag='sp-results')          \
             AS                                                     \
               SELECT country, AVG(num) FROM STREAM:tail.0          \
               WINDOW TUMBLING (1 SECOND)                           \
               WHERE country='Chile'                                \
               GROUP BY country;"                                   \
         -o stdout -m 'sp-results' -f 1
```

输出:

```text
[0] sp-results: [1557325032.000160100, {"country"=>"Chile", "AVG(num)"=>53.164558}]
```

## F.A.Q

### Where STREAM name comes from?

Fluent Bit 具有流的概念，且每个输入插件实例都有一个默认名称。您可以通过设置别名来覆盖默认名称。注意以下示例中 **alias** 参数和新的 **stream** 名称。

```bash
$ docker run -ti -v `pwd`/sp-samples-1k.log:/sp-samples-1k.log      \
     fluent/fluent-bit:1.4                                          \
     /fluent-bit/bin/fluent-bit                                     \
         -R /fluent-bit/etc/parsers.conf                            \
         -i tail                                                    \
             -p path=/sp-samples-1k.log                             \
             -p parser=json                                         \
             -p alias=samples                                       \
         -T "CREATE STREAM results WITH (tag='sp-results')          \
             AS                                                     \
               SELECT country, AVG(num) FROM STREAM:samples         \
               WINDOW TUMBLING (1 SECOND)                           \
               WHERE country='Chile'                                \
               GROUP BY country;"                                   \
         -o stdout -m 'sp-results' -f 1
```

