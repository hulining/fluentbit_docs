# 手动提取记录

在某些情况下，使用 Fluent Bit 库将记录从调用者应用程序发送到某个目的地，这个过程称为_手动数据提取_。

为此，我们提供了一个名为 **lib** 的特殊输入插件，可以与 **`flb_lib_push()`** API 函数结合使用。

## Data Format

**lib** 输入插件期望数据采用固定的 JSON 格式，如下所示:

```text
[UNIX_TIMESTAMP, MAP]
```

每个记录必须是一个至少包含两项的 JSON 数组。第一个是 Unix 时间戳，表示事件生成的时间，第二个是带有键值列表的 JSON 映射。有效的示例可以是如下内容:

```javascript
[1449505010, {"key1": "some value", "key2": false}]
```

## Usage

以下 C 代码片段演示了如何在运行的 Fluent Bit 引擎中插入一些 JSON 记录:

```c
#include <fluent-bit.h>

#define JSON_1   "[1449505010, {\"key1\": \"some value\"}]"
#define JSON_2   "[1449505620, {\"key1\": \"some new value\"}]"

int main()
{
    int ret;
    int in_ffd;
    int out_ffd;
    flb_ctx_t *ctx;

    /* Create library context */
    ctx = flb_create();
    if (!ctx) {
        return -1;
    }

    /* Enable the input plugin for manual data ingestion */
    in_ffd = flb_input(ctx, "lib", NULL);
    if (in_ffd == -1) {
        flb_destroy(ctx);
        return -1;
    }

    /* Enable output plugin 'stdout' (print records to the standard output) */
    out_ffd = flb_output(ctx, "stdout", NULL);
    if (out_ffd == -1) {
        flb_destroy(ctx);
        return -1;
    }

    /* Start the engine */
    ret = flb_start(ctx);
    if (ret == -1) {
        flb_destroy(ctx);
        return -1;
    }

    /* Ingest data manually */
    flb_lib_push(ctx, in_ffd, JSON_1, sizeof(JSON_1) - 1);
    flb_lib_push(ctx, in_ffd, JSON_2, sizeof(JSON_2) - 1);

    /* Stop the engine (5 seconds to flush remaining data) */
    flb_stop(ctx);

    /* Destroy library context, release all resources */
    flb_destroy(ctx);

    return 0;
}
```

