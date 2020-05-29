# Table of contents

* [Fluent Bit 中文文档](README.md)

## 关于 <a id="about"></a>

* [Fluent Bit 是什么？](about/what-is-fluent-bit.md)
* [Fluent Bit 简要历史](about/history.md)
* [Fluentd & Fluent Bit](about/fluentd-and-fluent-bit.md)
* [协议](about/license.md)

## 概念 <a id="concepts"></a>

* [核心概念](concepts/key-concepts.md)
* [缓冲](concepts/buffering.md)
* [数据管道](concepts/data-pipeline/README.md)
  * [输入](concepts/data-pipeline/input.md)
  * [解析器](concepts/data-pipeline/parser.md)
  * [过滤器](concepts/data-pipeline/filter.md)
  * [缓冲](concepts/data-pipeline/buffer.md)
  * [路由器](concepts/data-pipeline/router.md)
  * [输出](concepts/data-pipeline/output.md)

## 安装 <a id="installation"></a>

* [升级说明](installation/upgrade-notes.md)
* [平台支持](installation/supported-platforms.md)
* [依赖](installation/requirements.md)
* [源码安装](installation/sources/README.md)
  * [下载源码](installation/sources/download-source-code.md)
  * [构建和安装](installation/sources/build-and-install.md)
  * [以静态配置构建](installation/sources/build-with-static-configuration.md)
* [Linux 软件包](installation/linux/README.md)
  * [Amazon Linux](installation/linux/amazon-linux.md)
  * [Redhat / CentOS](installation/linux/redhat-centos.md)
  * [Debian](installation/linux/debian.md)
  * [Ubuntu](installation/linux/ubuntu.md)
  * [树莓派](installation/linux/raspbian-raspberry-pi.md)
* [Docker](installation/docker.md)
* [Kubernetes](installation/kubernetes.md)
* [Windows](installation/windows.md)

## 管理 <a id="administration"></a>

* [配置 Fluent Bit](administration/configuring-fluent-bit/README.md)
  * [格式与模式](administration/configuring-fluent-bit/format-schema.md)
  * [配置文件](administration/configuring-fluent-bit/configuration-file.md)
  * [变量](administration/configuring-fluent-bit/variables.md)
  * [命令](administration/configuring-fluent-bit/commands.md)
  * [上游服务负载均衡](administration/configuring-fluent-bit/upstream-servers.md)
  * [单位](administration/configuring-fluent-bit/unit-sizes.md)
* [安全性](administration/security.md)
* [缓冲与存储](administration/buffering-and-storage.md)
* [积压](administration/backpressure.md)
* [调度与重试](administration/scheduling-and-retries.md)
* [内存管理](administration/memory-management.md)
* [监控](administration/monitoring.md)
* [内部状态导出/信号](administration/dump-internals-signal.md)

## 数据管道 <a id="pipeline"></a>

* [输入插件](pipeline/inputs/README.md)
  * [标准输入](pipeline/inputs/standard-input.md)
  * [Systemd](pipeline/inputs/systemd.md)
  * [tail](pipeline/inputs/tail.md)
* [解析器](pipeline/parsers/README.md)
  * [JSON](pipeline/parsers/json.md)
  * [正则表达式](pipeline/parsers/regular-expression.md)
  * [LTSV](pipeline/parsers/ltsv.md)
  * [Logfmt](pipeline/parsers/logfmt.md)
* [过滤器](pipeline/filters/README.md)
  * [Grep](pipeline/filters/grep.md)
  * [Nest](pipeline/filters/nest.md)

