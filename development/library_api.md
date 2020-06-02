# C Library API

[Fluent Bit](http://fluentbit.io) 库是用 C 语言编写的，可以在任何 C 或 C++ 应用程序中使用。在深入研究规范之前，建议您先了解运行时的工作流程。

## 工作流程 <a id="workflow"></a>

[Fluent Bit](http://fluentbit.io) 作为服务运行，这意味着为开发人员公开的 API 提供了用于创建和管理上下文，指定输入/输出，设置配置参数及事件/记录的路由路径的接口。该库的典型用法包括:

* 创建库实例/上下文并设置属性
* 启用 _input_ 插件并设置相关属性
* 启用 _output_ 插件并设置相关属性
* 启动库运行时
* 可选的手动提取记录
* 停止库运行时
* 销毁库实例/上下文

## 数据类型 <a id="data-types"></a>

从 Fluent Bit v0.9 开始，库仅公开一种数据类型，且约定前缀为 **flb\_**。

| Type | Description |
| :--- | :--- |
| flb\_ctx\_t | 主库上下文。它多用于引用 `flb_create()` 返回的上下文 |

## API 参考 <a id="api-reference"></a>

### 创建库上下文 <a id="library-context-creation"></a>

如前所述，使用库的第一步是使用 **`flb_create()`** 创建它的上下文。

**原型**

```c
flb_ctx_t *flb_create();
```

**返回值**

如果成功，**`flb_create()`** 返回库的上下文；如果出错，返回NULL。

**用法**

```c
flb_ctx_t *ctx;

ctx = flb_create();
if (!ctx) {
    return NULL;
}
```

### 设置服务属性 <a id="set-service-properties"></a>

使用 **`flb_service_set()`** 函数可以设置上下文属性

**原型**

```c
int flb_service_set(flb_ctx_t *ctx, ...);
```

**返回值**

如果成功，返回 0；如果出错，则返回一个负数。

**用法**

**`flb_service_set()`** 支持以键值对的字符串形式设置一个或多个属性，如:

```c
int ret;

ret = flb_service_set(ctx, "Flush", "1", NULL);
```

如上示例指定了 **Flush** 属性的值。请注意，参数值始终是字符串\(char_\)，且一旦没有更多参数，必须在参数列表末尾添加 \*NULL_ 参数。

### 启用输入插件实例 <a id="enable-input-plugin-instance"></a>

构建完成后，[Fluent Bit](http://fluentbit.io) 库包含一些内置的 _input_ 插件。要启用 _input_ 插件，可以用 **`flb_input()`** 函数创建它的实例。

> 对于插件来说，实例表示已启用的插件上下文。您可以创建同一个插件的多个实例。

**原型**

```c
int flb_input(flb_ctx_t *ctx, char *name, void *data);
```

参数 **ctx** 表示由 **`flb_create()`** 创建的上下文，**name** 是要启用的输入插件名称。

第三个参数 **data** 可用于将自定义引用传递给插件实例，这通常由自定义或第三方插件使用，对于通用插件，一般传入 _NULL_。

**返回值**

如果成功，**`flb_input()`** 返回一个大于等于 0 的整型值\(类似于文件描述符\)；如果出错，返回一个负数。

**用法**

```c
int in_ffd;

in_ffd = flb_input(ctx, "cpu", NULL);
```

### 设置输入插件属性 <a id="set-input-plugin-properties"></a>

通过 **`flb_input()`** 创建的插件实例可能提供一些配置属性，使用 **`flb_input_set()`** 函数可以提供这些属性。

**原型**

```c
int flb_input_set(flb_ctx_t *ctx, int in_ffd, ...);
```

**返回值**

如果成功，返回 0；如果出错，返回一个负数。

**用法**

The **`flb_input_set()`** 允许在键/值字符串模式下设置一个或多个属性，例如:

```c
int ret;

ret = flb_input_set(ctx, in_ffd,
                    "tag", "my_records",
                    "ssl", "false",
                    NULL);
```

参数 **ctx** 表示由 **`flb_create()`** 创建的上下文。上面的示例指定了 **tag** 和 **ssl** 属性值，请注意，该值始终是字符串\(char\*\)，一旦没有更多参数，则必须在参数列表的末尾添加 _NULL_ 参数。

每个输入插件允许的属性在该插件的文档中指定。

### 启用输出插件实例 <a id="enable-output-plugin-instance"></a>

构建完成后，[Fluent Bit](http://fluentbit.io) 库包含一些内置的 _output_ 插件。要启用 _output_ 插件，可以用 **`flb_output()`** 函数创建它的实例。

> 对于插件来说，实例表示已启用的插件上下文。您可以创建同一个插件的多个实例。

**原型**

```c
int flb_output(flb_ctx_t *ctx, char *name, void *data);
```

参数 **ctx** 表示由 **`flb_create()`** 创建的上下文，**name** 是要启用的输入插件名称。

第三个参数 **data** 可用于将自定义引用传递给插件实例，这通常由自定义或第三方插件使用，对于通用插件，一般传入 _NULL_。

**返回值**

如果成功，**`flb_output()`** 返回输出插件实例；如果出错，返回一个负数。

**用法**

```c
int out_ffd;

out_ffd = flb_output(ctx, "stdout", NULL);
```

### 设置输出插件属性 <a id="set-output-plugin-properties"></a>

通过 **`flb_output()`** 创建的插件实例可能提供一些配置属性，使用 **`flb_output_set()`** 函数可以提供这些属性。

**原型**

```c
int flb_output_set(flb_ctx_t *ctx, int out_ffd, ...);
```

**返回值**

如果成功，返回大于等于 0 的整数；如果出错，返回一个负数。

**用法**

The **`flb_output_set()`** 允许在键/值字符串模式下设置一个或多个属性，例如:

```c
int ret;

ret = flb_output_set(ctx, out_ffd,
                     "tag", "my_records",
                     "ssl", "false",
                     NULL);
```

参数 **ctx** 表示由 **`flb_create()`** 创建的上下文。上面的示例指定了 **tag** 和 **ssl** 属性值，请注意，该值始终是字符串\(char\*\)，一旦没有更多参数，则必须在参数列表的末尾添加 _NULL_ 参数。

每个输出插件允许的属性在该插件的文档中指定。

## 启动 Fluent Bit 引擎 <a id="start-fluent-bit-engine"></a>

创建库上下文并设置输入/输出插件实例后，下一步就是启动引擎。 启动后，引擎将在新线程\(POSIX thread\)内运行，而不会阻塞调用者应用程序。要启动引擎，请使用 **`flb_start()`** 函数。

**原型**

```c
int flb_start(flb_ctx_t *ctx);
```

**返回值**

如果成功，返回 0；如果出错，则返回一个负数。

**Usage**

这个简单的调用仅需要 **ctx** 作为参数传入，它是对先前 **`flb_create()`** 创建的上下文的引用:

```c
int ret;

ret = flb_start(ctx);
```

## 停止 Fluent Bit 引擎 <a id="stop-fluent-bit-engine"></a>

我们提供了 **`flb_stop()`** 函数用于停止运行中的 Fluent Bit 引擎。

**原型**

```c
int flb_stop(flb_ctx_t *ctx);
```

参数 **ctx** 是对先前 **`flb_create()`** 函数创建的且通过 **`flb_start()`** 函数启动的上下文的引用。

当我们调用此函数时，引擎将最多等待五秒钟以刷新缓冲区并释放正在使用的资源。停止的上下文可以随时重新启动，但没有任何数据。

**返回值**

如果成功，返回 0；如果出错，则返回一个负数。

**Usage**

```c
int ret;

ret = flb_stop(ctx);
```

## 销毁库上下文 <a id="destroy-library-context"></a>

库上下文在不需要时必须被销毁。请注意，必须先执行之前的 **`flb_stop()`** 函数调用。销毁后，将释放所有与之关联的资源。

**原型**

```c
void flb_destroy(flb_ctx_t *ctx);
```

参数 **ctx** 是对先前 **`flb_create()`** 创建的上下文的引用。

**返回值**

没有返回值。

**用法**

```c
flb_destroy(ctx);
```

## 手动提取数据 <a id="ingest-data-manually"></a>

在某些情况下，应用程序调用可能希望将数据摄取到 Fluent Bit中，为此目的，我们提供了 **`flb_lib_push()`** 函数。

**原型**

```c
int flb_lib_push(flb_ctx_t *ctx, int in_ffd, void *data, size_t len);
```

第一个参数是先前通过 **`flb_create()`** 创建的上下文。 **in\_ffd** 是输入插件的数字引用\(在这种情况下，它应该是一个插件 **lib** 类型的输入\)，**data** 是对输入消息的引用，**len** 是从消息中获取的字节数限定。

**返回值**

如果成功，它返回写入的字节数；如果出错，返回 -1。

**用法**

有关如何正确使用此功能的更多详细信息和示例，请参阅下一小节 - [手动提取记录](ingest-records-manually.md)。

