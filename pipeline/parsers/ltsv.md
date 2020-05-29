# LTSV

**ltsv** 解析器允许解析 [LTSV](http://ltsv.org/) 格式的文本。

Labeled Tab-separated Values 带标签的制表符分隔值\(LTSV 格式是 Tab 分隔的值\(TSV\)的变体。LTSV 文件中的每个记录都用单行表示。每个字段都由 Tab 分隔并具有标签和值。标签和值之间使用 ":" 分隔。

这是一个如何在 apache access log 中使用此格式的示例:

在 httpd.conf 中进行配置

```text
LogFormat "host:%h\tident:%l\tuser:%u\ttime:%t\treq:%r\tstatus:%>s\tsize:%b\treferer:%{Referer}i\tua:%{User-Agent}i" combined_ltsv
CustomLog "logs/access_log" combined_ltsv
```

`parser.conf` 配置如下:

```python
[PARSER]
    Name        access_log_ltsv
    Format      ltsv
    Time_Key    time
    Time_Format [%d/%b/%Y:%H:%M:%S %z]
    Types       status:integer size:integer
```

对于上述定义的解析器来说，以下日志记录是有效的:

```text
host:127.0.0.1  ident:- user:-  time:[10/Jul/2018:13:27:05 +0200]       req:GET / HTTP/1.1      status:200      size:16218      referer:http://127.0.0.1/       ua:Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0
host:127.0.0.1  ident:- user:-  time:[10/Jul/2018:13:27:05 +0200]       req:GET /assets/plugins/bootstrap/css/bootstrap.min.css HTTP/1.1        status:200      size:121200     referer:http://127.0.0.1/       ua:Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0
host:127.0.0.1  ident:- user:-  time:[10/Jul/2018:13:27:05 +0200]       req:GET /assets/css/headers/header-v6.css HTTP/1.1      status:200      size:37706      referer:http://127.0.0.1/       ua:Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0
host:127.0.0.1  ident:- user:-  time:[10/Jul/2018:13:27:05 +0200]       req:GET /assets/css/style.css HTTP/1.1  status:200      size:1279       referer:http://127.0.0.1/       ua:Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0
```

处理后，在内部表示为:

```text
[1531222025.000000000, {"host"=>"127.0.0.1", "ident"=>"-", "user"=>"-", "req"=>"GET / HTTP/1.1", "status"=>200, "size"=>16218, "referer"=>"http://127.0.0.1/", "ua"=>"Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0"}]
[1531222025.000000000, {"host"=>"127.0.0.1", "ident"=>"-", "user"=>"-", "req"=>"GET /assets/plugins/bootstrap/css/bootstrap.min.css HTTP/1.1", "status"=>200, "size"=>121200, "referer"=>"http://127.0.0.1/", "ua"=>"Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0"}]
[1531222025.000000000, {"host"=>"127.0.0.1", "ident"=>"-", "user"=>"-", "req"=>"GET /assets/css/headers/header-v6.css HTTP/1.1", "status"=>200, "size"=>37706, "referer"=>"http://127.0.0.1/", "ua"=>"Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0"}]
[1531222025.000000000, {"host"=>"127.0.0.1", "ident"=>"-", "user"=>"-", "req"=>"GET /assets/css/style.css HTTP/1.1", "status"=>200, "size"=>1279, "referer"=>"http://127.0.0.1/", "ua"=>"Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0"}]
```

_time_ 字段已经被转换为 Unix 时间戳\(UTC\)。

