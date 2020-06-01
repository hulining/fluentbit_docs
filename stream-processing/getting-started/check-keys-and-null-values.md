# Check Keys and NULL values

> Fluent Bit &gt;= 1.2 可用功能

当使用结构化消息\(记录\)时，在某些情况下，我们想知道某个键是否存在，它的值是 _null_ \(空\)还是具有非 _null_ \(非空\)的值。

[Fluent Bit](https://fluentbit.io) 内部记录是带有键值映射的序列化二进制数据。其值可以为空，这是有效的数据类型。在我们的 SQL 语言中，我们提供了以下可作为条件语句的语句:

## Check if a key value IS NULL

以下 SQL 语句可用于从 _test_ 数据流查找 _phone_ 键值为 _null_ 的所有记录:

```sql
SELECT * FROM STREAM:test WHERE phone IS NULL;
```

## Check if a key value IS NOT NULL

与上面的示例类似，在某些情况下，我们希望查找所有某些键的值不为 _null_ 的记录：

```sql
SELECT * FROM STREAM:test WHERE phone IS NOT NULL;
```

## Check if a key exists

另一个常见用例是检查记录中是否存在某些键。我们提供了可以在 SQL 语句的条件部分中使用特定的记录函数。用于检查记录中是否存在键的函数原型如下：

```text
@record.contains(key)
```

下面的示例查询包含 _phone_ 的键的所有记录:

```sql
SELECT * FROM STREAM:test WHERE @record.contains(phone);
```

