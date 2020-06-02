# Golang 输出插件

Fluent Bit 当前仅支持将 Golang 插件集成为仅用于输出插件的共享库。Golang 插件的接口目前正在开发中，但可以使用。

Fluent Bit currently supports integration of Golang plugins built as shared objects for output plugins only. The interface for the Golang plugins is currently under development but is functional.

Fluent Bit当前仅支持将 Golang 插件集成为仅用于输出插件的共享库。Golang 插件的接口目前正在开发中，但可以使用。

## Getting Started

启用 Golang 支持选项编译 Fluent Bit，如:

```text
$ cd build/
$ cmake -DFLB_DEBUG=On -DFLB_PROXY_GO=On ../
$ make
```

编译后，我们可以在二进制中看到新选项 `-e`，它代表 _external plugin_，例如:

```text
$ bin/fluent-bit -h
Usage: fluent-bit [OPTION]

Available Options
  -c  --config=FILE    specify an optional configuration file, 指定可选的配置文件
  -d, --daemon        run Fluent Bit in background mode, 以后台方式运行 Fluent Bit
  -f, --flush=SECONDS    flush timeout in seconds (default: 5), 刷新数据到输出的超时时间(默认 5 秒)
  -i, --input=INPUT    set an input, 设置输入
  -m, --match=MATCH    set plugin match, same as '-p match=abc', 设置插件匹配的标签模式，与 '-p match=abc' 相同
  -o, --output=OUTPUT    set an output, 设置输出
  -p, --prop="A=B"    set plugin configuration property, 设置插件配置参数
  -e, --plugin=FILE    load an external plugin (shared lib), 加载扩展插件(共享库)
  ...
```

## Build a Go Plugin

**`fluent-bit-go`** 软件包可用于帮助开发人员创建 Go 插件

[https://github.com/fluent/fluent-bit-go](https://github.com/fluent/fluent-bit-go)

作为最小形式，Go 插件至少如下所示：

```go
package main

import "github.com/fluent/fluent-bit-go/output"

//export FLBPluginRegister
func FLBPluginRegister(def unsafe.Pointer) int {
    // Gets called only once when the plugin.so is loaded
    return output.FLBPluginRegister(ctx, "gstdout", "Stdout GO!")
}

//export FLBPluginInit
func FLBPluginInit(plugin unsafe.Pointer) int {
    // Gets called only once for each instance you have configured.
    return output.FLB_OK
}

//export FLBPluginFlushCtx
func FLBPluginFlushCtx(ctx, data unsafe.Pointer, length C.int, tag *C.char) int {
    // Gets called with a batch of records to be written to an instance.
    return output.FLB_OK
}

//export FLBPluginExit
func FLBPluginExit() int {
    return output.FLB_OK
}

func main() {
}
```

上面的代码是编写输出插件的模板，将包名称设置为 `main` 并添加显式的 `main()` 函数非常重要。这是必需的，因为代码将被构建为共享库。

要构建上面的代码，请使用以下命令行:

```bash
$ go build -buildmode=c-shared -o out_gstdout.so out_gstdout.go
```

构建完成后，共享库 `out_gstdout.so` 将可被使用。仔细检查最终的 .so 文件确实是我们所期望的，这非常重要。对库执行 `ldd` 命令，我们应该看到类似于如下的内容:

```text
$ ldd out_gstdout.so
    linux-vdso.so.1 =>  (0x00007fff561dd000)
    libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fc4aeef0000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fc4aeb27000)
    /lib64/ld-linux-x86-64.so.2 (0x000055751a4fd000)
```

## Run Fluent Bit with the new plugin

```bash
$ bin/fluent-bit -e /path/to/out_gstdout.so -i cpu -o gstdout
```

