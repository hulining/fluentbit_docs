# 内存管理

在某些情况下，理想的方法是估计 Fluent Bit 可以使用多少内存，这对于必须限制内存的容器化环境非常有用。

为了进行估算，我们将假定输入插件已设置 **Mem\_Buf\_Limit** 选项\(您可以在[积压](backpressure.md)部分了解更多信息\)。

## 估算 <a id="estimating"></a>

输入插件无限制地追加数据，因此为了进行估算，应通过 **Mem\_Buf\_Limit** 选项施加限制。如果限制设置为 _10MB_，我们需要估计在最坏的情况下，输出插件可能会使用 _20MB_。

Fluent Bit 具有用于处理数据的内部二进制表示形式，但是当此数据到达输出插件时，该插件可能会在新的内存缓冲区中创建自己的表示形式以进行处理。最好的例子是 [InfluxDB](../pipleline/outputs/influxdb.md) 和 [Elasticsearch](../pipeline/outputs/elasticsearch.md) 输出插件，在与后端服务器通信之前，它们都需要将二进制表示形式转换为各自的自定义 JSON 格式。

因此，如果我们对输入插件施加 _10MB_ 的限制，并考虑输出插件消耗 _20MB_ 的最坏情况，则至少需要\(_30MB_ x 1.2=\)**36MB**。

## Glibc 与内存碎片 <a id="glibc-and-memory-fragmentation"></a>

众所周知，在密集型环境中，内存分配按数量级进行，Glibc 提供的默认内存分配器可能会导致碎片过多，从而导致服务占用大量内存。

强烈建议在任何生产环境，都应启用 [jemalloc](http://jemalloc.net/)\(如 `-DFLB_JEMALLOC=On`\) 的情况下构建 Fluent Bit。Jemalloc 是一种替代的内存分配器，除其余功能外，它可以减少碎片，从而提高性能。

您可以如下命令检查 Fluent Bit 是否使用 Jemalloc 进行构建:

```text
$ bin/fluent-bit -h|grep JEMALLOC
```

输出应该如下:

```text
Build Flags =  JSMN_PARENT_LINKS JSMN_STRICT FLB_HAVE_TLS FLB_HAVE_SQLDB
FLB_HAVE_TRACE FLB_HAVE_FLUSH_LIBCO FLB_HAVE_VALGRIND FLB_HAVE_FORK
FLB_HAVE_PROXY_GO FLB_HAVE_JEMALLOC JEMALLOC_MANGLE FLB_HAVE_REGEX
FLB_HAVE_C_TLS FLB_HAVE_SETJMP FLB_HAVE_ACCEPT4 FLB_HAVE_INOTIFY
```

如果在构建标志中列出了 `FLB_HAVE_JEMALLOC` 选项，一切都会很好。

