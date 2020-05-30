# PostgreSQL

**pgsql** 输出插件允许将数据从 Fluent Bit 发送到 9.4 或更高版本的 PostgreSQL 数据库服务器。为了存储数据，我们使用JSON 类型，因此请确保您的 PostgreSQL 实例支持该类型。

## Configuration Parameters

| Key | Description | Default |
| :--- | :--- | :--- |
| Host | PostgreSQL 实例的主机名 | 127.0.0.1 |
| Port | PostgreSQL 端口 | 5432 |
| Database | 要连接的数据库名 | fluentbit |
| Table | 存储数据的表名 | fluentbit |
| User | PostgreSQL 用户名 | 当前用户 |
| Password | PostgreSQL  密码 |  |
| Timestamp\_Key | 存储记录时间戳的键 | date |

## Configuration Example

在您的主要配置文件中，添加如下配置段:

```text
[OUTPUT]
    Name          pgsql
    Match         *
    Host          172.17.0.2
    Port          5432
    Database      fluentbit
    Table         fluentbit
    User          postgres
    Password      YourCrazySecurePassword
    Timestamp_Key timestamp
```

