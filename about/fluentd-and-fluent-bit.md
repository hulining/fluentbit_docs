---
description: 生产级生态系统
---

# Fluentd & Fluent Bit

通常，日志和数据处理可能是很复杂的，对于大规模的更为显著，这就是 [Fluentd](https://www.fluentd.org/) 诞生的原因。但现在它不仅仅是一个简单的工具，而是一个完整的生态系统，其中包含针对不同语言和子项目(如 [Fluent Bit](https://fluentbit.io/))的 SDK。

在此文中，我们将描述 [Fluentd](https://www.fluentd.org/) 和 [Fluent Bit](https://fluentbit.io/) 开源项目之间的关系，作为总结，我们可以说两者都是：

- 基于 Apache License v2.0 的条款许可
- 由[云原生计算基金会(CNCF)](https://cncf.io/)托管的子项目
- 生产级解决方案: 部署达每日**数千**次，每月**百万**次
- 社区推动的项目
- 业界广泛采用: 受到 AWS，Microsoft，Google Cloud 等数百个公司的信任
- 最初由 [Treasure Data](https://www.treasuredata.com) 创建

这两个项目有很多相似之处，[Fluent Bit](https://fluentbit.io) 是完全在 [Fluentd](https://www.fluentd.org) 体系结构和一般设计的最佳思想之上设计和构建的。选择使用哪一个取决于用户的最终需求。

下表描述了两个项目在不同方面的比较:

 | Fluentd | Fluent Bit
:---: | :---: | :---:
适用范围 | 容器/服务器 | 嵌入式 Linux/容器/服务器
开发语言 | Ruby & C | C
内存消耗 | 大约 40MB | 大约 650KB
性能 | 高性能 | 高性能
依赖 | 基于 Ruby Gem 构建，依赖一些 gem(Ruby 模块) | 无依赖，除了一些特殊的插件
插件 | 超过 1000 个可用插件 | 大约 70 个可用插件
协议 | [Apache License v2.0](http://www.apache.org/licenses/LICENSE-2.0) | [Apache License v2.0](http://www.apache.org/licenses/LICENSE-2.0)

[Fluentd](https://www.fluentd.org/) 和 [Fluent Bit](https://fluentbit.io/) 都可以充当聚合器或转发器，它们可以互补使用或单独用作为解决方案。
