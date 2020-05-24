# Docker

Fluent Bit 容器镜像可以在 Docker Hub 上获取，可用于生产环境。当前可用的镜像可以部署在多种架构体系中。

## 标签与版本 <a id="tags-and-versions"></a>

下表描述了可在 Docker Hub [fluent/fluent-bit](https://hub.docker.com/r/fluent/fluent-bit/) 镜像仓库中使用的镜像及其标签:

| 标签 | 架构 | 版本描述 |
| :--- | :--- | :--- |
| 1.4 | x86\_64, arm64v8, arm32v7 | 1.4.x 系列的最新版本 |
| 1.4.4 | x86\_64, arm64v8, arm32v7 | Release [v1.4.4](https://fluentbit.io/announcements/v1.4.4/) |
| 1.4-debug, 1.4.1-debug | x86\_64 | v1.4.x releases + Busybox |
| 1.4.3 | x86\_64, arm64v8, arm32v7 | Release [v1.4.3](https://fluentbit.io/announcements/v1.4.3/) |
| 1.4-debug, 1.4.3-debug | x86\_64 | v1.4.x releases + Busybox |
| 1.4.2 | x86\_64, arm64v8, arm32v7 | Release [v1.4.2](https://fluentbit.io/announcements/v1.4.2/) |
| 1.4-debug, 1.4.1-debug | x86\_64 | v1.4.x releases + Busybox |
| 1.4.1 | x86\_64, arm64v8, arm32v7 | Release [v1.4.1](https://fluentbit.io/announcements/v1.4.1/) |
| 1.4-debug, 1.4.1-debug | x86\_64 | v1.4.x releases + Busybox |
| 1.4.0 | x86\_64, arm64v8, arm32v7 | Release [v1.4.0](https://fluentbit.io/announcements/v1.4.0) |
| 1.4-debug, 1.4.0-debug | x86\_64 | v1.4.x releases + Busybox |

强烈建议您始终使用 Fluent Bit 的最新镜像。

## 多架构镜像 <a id="multi-architecture-images"></a>

我们的 x86\_64 稳定版镜像基于 [Distroless](https://github.com/GoogleContainerTools/distroless) 构建，专注于仅包含 Fluent Bit 二进制文件和最小的系统库以及基本配置的安全性。另外，我们为 x86\_64 提供包含 Busybox 的 **debug** 镜像，可用于进行故障排查或测试。

此外，主清单提供了用于 arm64v8 和 arm32v7 架构的镜像。从部署的角度来看，无需指定架构，拉取容器镜像的客户端工具将根据其运行的架构获取正确的镜像层。

对于每种架构，我们使用以下基础图像来构建镜像

| 架构 | 基础镜像 |
| :--- | :--- |
| x86\_64 | [Distroless](https://github.com/GoogleContainerTools/distroless) |
| arm64v8 | arm64v8/debian:buster-slim |
| arm32v7 | arm32v7/debian:buster-slim |

## 快速开始 <a id="getting-started"></a>

从 1.4 系列版本拉取稳定版本的镜像:

```bash
$ docker pull fluent/fluent-bit:1.4
```

拉取完成后，运行以下测试，使 Fluent Bit 可以测量容器的 CPU 使用率

```bash
$ docker run -ti fluent/fluent-bit:1.4 /fluent-bit/bin/fluent-bit -i cpu -o stdout -f 1
```

该命令将使 Fluent Bit 每秒测量一次 CPU 使用率并将结果刷新到标准输出，例如:

```bash
Fluent-Bit v1.4.x
Copyright (C) Treasure Data

[2019/10/01 12:29:02] [ info] [engine] started
[0] cpu.0: [1504290543.000487750, {"cpu_p"=>0.750000, "user_p"=>0.250000, "system_p"=>0.500000, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>1.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>1.000000, "cpu2.p_cpu"=>1.000000, "cpu2.p_user"=>1.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>0.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>0.000000}]
```

## 常见问题 <a id="faq"></a>

### 为什么没有基于 Alpine Linux 的 Fluent Bit Docker 镜像 ? <a id="why-there-is-no-fluent-bit-docker-image-based-on-alpine-linux"></a>

Alpine Linux 使用 Musl C 库而不是 Glibc。 Musl 与 Glibc不完全兼容，当与 Fluent Bit 一起使用时，会产生许多以下方面的问题:

* 内存分配: 为了在高负载环境中平稳运行 Fluent Bit，我们使用 Jemalloc 作为默认的内存分配器，这样可以减少内存碎片且提供更好的性能。Jemalloc 无法与 Musl 一起平稳运行，且需要额外的工作
* Alpine Linux Musl 函数引导程序在加载 Golang 共享库时存在兼容性问题，这在 Fluent Bit 中尝试加载 Golang 输出插件时会产生问题
* Alpine Linux Musl Time 格式解析器不支持 Glibc 扩展
* 由于安全和维护原因，维护人员在基本镜像方面的偏好是 Distroless 和 Debian

### `latest` 表示什么标签? <a id="where-latest-tag-points-to"></a>

我们的 Docker 容器镜像每天部署数千次，我们非常重视安全性和稳定性。

_`latest`_ 标签 _大多数时间_ 指向最新的稳定版镜像。当我们发布 Fluent Bit 的主要更新\(例如从 v1.3.x 到 v1.4.0\)时，发布两周后我们才将 _`latest`_ 标签进行改变。这能够给我们额外的时间以便社区核实一切都能够按预期的进行。

