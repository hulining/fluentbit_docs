# 单位

Fluent Bit 中的某些配置指令涉及单位大小，例如在定义缓冲区大小或特定限制时，我们可以在 [Tail Input](tail)，[Forward Input](forward) 之类的插件或在 \[Mem\_Buf\_Limit\] 之类的通用属性中找到这些单位。

从 [Fluent Bit](http://fluentbit.io) v0.11.10 开始，所有单位大小已在内核和插件之间进行了标准化，下表描述了可以使用的选项及其含义:

| 单位后缀 | 描述 | 示例 |
| :--- | :--- | :--- |
|  | 如果未指定后缀，则给定的值以字节表示。 | Specifying a value of 32000, means 32000 bytes |
| k, K, KB, kb | Kilobyte: a unit of memory equal to 1,000 bytes. | 32k means 32000 bytes. |
| m, M, MB, mb | Megabyte: a unit of memory equal to 1,000,000 bytes | 1M means 1000000 bytes |
| g, G, GB, gb | Gigabyte: a unit of memory equal to 1,000,000,000 bytes | 1G means 1000000000 bytes |

