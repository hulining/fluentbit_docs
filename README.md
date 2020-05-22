---
description: 高性能日志处理器
---

# Fluent Bit v1.4 中文文档

> 此文档是根据 [Fluent Bit](https://docs.fluentbit.io/manual/) 翻译的非官方中文手册，旨在为自己和大家提供一个关关于 Fluent Bit 学习的中文参考。鉴于本人能力有限，翻译或理解有错误的地方，还请大家多多包涵并帮忙修订校正，不吝感谢。

![](.gitbook/assets/logo_documentation_1.4.png)

[Fluent Bit](http://fluentbit.io/) 是用于 Linux，OSX，Windows 和 BSD 系列操作系统的快速轻量级日志处理器，流处理器和转发器。它非常注重性能，允许对不同来源的事件进行收集且简单易用。

## 特性 <a id="routing-with-wildcard"></a>

* 高性能
* 数据转换
  * 使用我们提供的解析器转换您的非结构化消息: [JSON](pipeline/parsers/json.md), [Regex](pipeline/parsers/regular-expression.md), [LTSV](pipeline/parsers/ltsv.md) 和 [Logfmt](pipeline/parsers/logfmt.md)
* 可靠性和数据完整性
  * [Backpressure](administration/backpressure.md) 处理
  * 内存和文件系统中的[数据缓冲](administration/buffering-and-storage.md)
* 网络
  * 安全性: 内建 TLS/SSL 支持
  * 异步 I/O
* 可插拔架构和[可扩展性](development/library_api.md): 输入，过滤和输出
  * 提供 50 多个内建插件
  * 易扩展
    * 所有输入，过滤和输出插件均使用 C 语言编写
    * 另外包含[使用 Lua 编写的过滤插件](pipeline/filters/lua.md)和[使用 Golang 编写的输出插件](development/golang-output-plugins.md)
* [监控](administration/monitoring.md): 通过 HTTP 公开 JSON 和 [Prometheus](https://prometheus.io/) 格式的内部数据指标
* [流式处理](./): 使用简单的 SQL 查询执行数据选择和转换
  * 使用查询结果创建新的数据流
  * 聚合窗口
  * 数据分析和预测: 时间序列预测
* 可移植: 可在 Linux, MacOS, Windows 和 BSD 系统上运行

## Fluent Bit, Fluentd and CNCF

[Fluent Bit](http://fluentbit.io/) 是 [Fluentd](http://fluentd.org/) 项目生态系统的子组件，基于[Apache License v2.0](http://www.apache.org/licenses/LICENSE-2.0) 协议许可。此项目由 [Treasure Data](https://www.treasuredata.com/) 创建，目前 Treasure Data 是其主要贡献者。

如今，Fluent Bit 获得了众多团队和个人的贡献，与 [Fluentd](https://www.fluentd.org/) 一样，它被托管为 [CNCF](https://cncf.io/) 子项目。
