# 快速开始

以下指南假定您熟悉 [Fluent Bit](https://fluentbit.io)，否则，我们建议您先阅读官方手册:

* [Fluent Bit Manual](https://docs.fluentbit.io/manual/)

## Requirements

* [Fluent Bit](https://fluentbit.io) &gt;= v1.1.0 或从 [GIT Master](https://github.com/fluent/fluent-bit) 最版本编译的 Fluent Bit
* 对结构化查询语言\(SQL\)有基本了解

## Technical Concepts

| 概念 | 描述 |
| :--- | :--- |
| Stream | 流表示由输入插件输入的唯一数据流。默认情况下，Streams 使用插件名称加上内部数字标识来获取名称，例如: tail.0。可以通过设置 `alias` 属性来更改流名称 |
| Task | 流处理器配置具有 Task 的概念Tasks 代表一个执行单元，简而言之: SQL 查询被配置为一个 Task |
| Results | 当流处理器执行 SQL 查询时，将生成结果。这些结果可以重新输入到主 Fluent Bit 管道中，也可以为了调试直接重定向到标准输出。 |
| Tag | Fluent Bit 对记录进行分组并将其与 Tag 关联。标签用于定义路由规则，或附加到匹配特定模式的标签 |
| Match | 使用通配符来匹配与标签关联的特定记录的匹配规则 |

