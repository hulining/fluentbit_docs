---
description: 可靠地进行数据处理
---

# 缓冲区

管道中的`缓冲`阶段在先前的[缓冲](../buffering.md)概念部分中定义过，旨在提供统一的持久性机制来存储数据，无论是主要使用内存模式中还是基于文件系统的模式。

![](../../.gitbook/assets/logging_pipeline_buffer%20%281%29.png)

`缓冲` 阶段包含已经处于不可变状态的数据，这意味着无法应用其它的过滤器。

{% hint style="info" %}
请注意，缓冲阶段的数据不再是原始文本，而是 Fluent Bit 内部二进制表示形式。
{% endhint %}

Fluent Bit 在文件系统中提供了种缓冲机制，用作 _backup system\(备份系统\)_ 来避免系统故障时造成数据丢失。

