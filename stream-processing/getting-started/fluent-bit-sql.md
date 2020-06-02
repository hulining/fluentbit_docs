# Fluent Bit + SQL

Fluent Bit 流处理器使用通用 SQL 执行查询记录。以下部分描述了其可用功能及示例。

## Statements

您可以在[这里](https://github.com/fluent/fluent-bit/tree/master/src/stream_processor)中找到 BNF 形式的详细查询语言语法。下一节将简要介绍如何使用 SQL 查询语句进行 Fluent Bit 流处理。

### SELECT Statement

#### Synopsis

```sql
SELECT results_statement
  FROM STREAM:stream_name | TAG:match_rule
  [WINDOW TUMBLING (integer SECOND)]
  [WHERE condition]
  [GROUP BY groupby]
```

#### Description

从来自数据流或与特定模式匹配的记录中选择关键字。请注意，与流创建**不**关联的简单的 `SELECT` 语句会将结果发送到标准输出接口\(stdout\)，这对于调试目的来说很有用

查询语句支持使用 `WHERE` 语句来过滤结果。我们将在后面的 aggregation 聚合函数部分解释 `WINDOW` 和 `GROUP BY` 语句。

#### Examples

从来自名为 _apache_ 的流记录中选择所有键:

```sql
SELECT * FROM STREAM:apache;
```

从以 _apache._ 开头的标签的记录中选择所有键:

```sql
SELECT code AS http_status FROM TAG:'apache.*';
```

> 由于 TAG 选择器支持使用通配符，因此我们将值放在单引号中间。

### CREATE STREAM Statement

#### Synopsis

```sql
CREATE STREAM stream_name
  [WITH (property_name=value, [...])]
  AS select_statement
```

#### Description

使用 `SELECT` 语句的结果创建一个新的数据流。如果在 `WITH` 语句中设置了 `Tag` 属性，则可以选择将新创建的数据流重新输入到 Fluent Bit 管道中

#### Examples

从名为 _apache_ 的数据流中创建一个名为 _hello_ 的新数据流:

```sql
CREATE STREAM hello AS SELECT * FROM STREAM:apache;
```

为所有原始标签以 _apache_ 开头的记录创建一个名为 _hello_ 的新数据流:

```sql
CREATE STREAM hello AS SELECT * FROM TAG:'apache.*';
```

## Aggregation Functions

聚合函数在选择关键字的 `results_statement`\(结果声明\)中使用，从而可以对记录进行分组数据计算。聚合函数应用的记录组由 `WINDOW` 关键字确定。如果指定 `WINDOW` 关键字，则聚合函数将应用在接收到记录的当前缓冲区，该记录可能具有不确定的元素数量。可以将聚合函数应用于特定时间间隔窗口中的记录\(请参见 select 语句中 `WINDOW` 的语法\)。

Fluent Bit 支持窗口滚动，这是非重叠窗口类型。这意味着大小为 5 秒的窗口将对 5 秒间隔内的记录进行聚合运算，然后为下一时间间隔开始新的聚合运算。

另外，该语法支持 `GROUP BY` 语句，当 `GROUP BY` 指定的键具有相同的值时，结果按照一个或多个键进行分组。

### AVG

#### Synopsis

```sql
SELECT AVG(size) FROM STREAM:apache WHERE method = 'POST' ;
```

#### Description

计算 POST 请求的平均值大小

### COUNT

#### Synopsis

```sql
SELECT host, COUNT(*) FROM STREAM:apache WINDOW TUMBLING (5 SECOND) GROUP BY host;
```

#### Description

按主机 IP 地址计算 5 秒滑动窗口中的记录数

### MIN

#### Synopsis

```sql
SELECT MIN(key) FROM STREAM:apache;
```

#### Description

获取一组记录中指定键的值的最小值

### MAX

#### Synopsis

```sql
SELECT MIN(key) FROM STREAM:apache;
```

#### Description

获取一组记录中指定键的值的最大值

### SUM

#### Synopsis

```sql
SELECT SUM(key) FROM STREAM:apache;
```

#### Description

计算一组记录中键的所有值的总和

## Time Functions

时间函数添加新键到带有计时数据的记录中

### NOW

#### Synopsis

```sql
SELECT NOW() FROM STREAM:apache;
```

#### Description

使用如下格式添加当前系统时间: %Y-%m-%d %H:%M:%S. 输出示例: 2019-03-09 21:36:05.

### UNIX\_TIMESTAMP

#### Synopsis

```sql
SELECT UNIX_TIMESTAMP() FROM STREAM:apache;
```

#### Description

将当前的 Unix 时间戳添加到记录中. 输出示例: 1552196165 .

## Record Functions

Record Functions 记录函数使用记录上下文中的值将新键添加到记录

### RECORD\_TAG

#### Synopsis

```sql
SELECT RECORD_TAG() FROM STREAM:apache;
```

#### Description

将记录的标签\(字符串\)追加为记录的新关键字

### RECORD\_TIME

#### Synopsis

```sql
SELECT RECORD_TIME() FROM STREAM:apache;
```

## WHERE Condition

与常规 SQL 语句类似，Fluent Bit 查询语言支持 `WHERE` 条件。该语言支持对键和子键的进行条件筛选，例如：

```sql
SELECT AVG(size) FROM STREAM:apache WHERE method = 'POST' AND status = 200;
```

使用记录的 `@record.contains` 函数可以对记录中是否存在指定的键进行判断:

```sql
SELECT MAX(key) FROM STREAM:apache WHERE @record.contains(key);
```

检查键的值是否为`NULL`:

```sql
SELECT MAX(key) FROM STREAM:apache WHERE key IS NULL;
```

```sql
SELECT * FROM STREAM:apache WHERE user IS NOT NULL;
```

