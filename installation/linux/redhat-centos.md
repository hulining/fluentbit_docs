# Redhat / CentOS

## Install on Redhat / CentOS

Fluent Bit 发布的软件包为 **td-agent-bit**，可用于最新稳定版 CentOS 系统 。支持以下架构

* x86_64
* aarch64 / arm64v8

## Configure Yum

我们通过 Yum 仓库提供 **td-agent-bit** 软件包。为了将存储库引用添加到您的系统，请在 _`/etc/yum.repos.d/`_ 中添加一个名为 _`td-agent-bit.repo`_ 的文件，其内容如下:

```text
[td-agent-bit]
name = TD Agent Bit
baseurl = https://packages.fluentbit.io/centos/7/$basearch/
gpgcheck=1
gpgkey=https://packages.fluentbit.io/fluentbit.key
enabled=1
```

> 注意: 出于安全考虑，我们建议您始终使用 _gpgcheck_。我们所有的软件包均已签名。

GPG 密钥指纹为 `F209 D876 2A60 CD49 E680 633B 4FF8 368B 6EA0 722A`

### Install

配置存储库后，运行以下命令进行安装

```bash
$ yum install td-agent-bit
```

使用 _systemd_ 启动服务

```bash
$ sudo service td-agent-bit start
```

如果进行状态检查，应该会看到类似如下的输出:

```bash
$ service td-agent-bit status
Redirecting to /bin/systemctl status  td-agent-bit.service
● td-agent-bit.service - TD Agent Bit
   Loaded: loaded (/usr/lib/systemd/system/td-agent-bit.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2016-07-07 02:08:01 BST; 9s ago
 Main PID: 3820 (td-agent-bit)
   CGroup: /system.slice/td-agent-bit.service
           └─3820 /opt/td-agent-bit/bin/td-agent-bit -c etc/td-agent-bit/td-agent-bit.conf
...
```

**td-agent-bit** 的默认配置是收集 CPU 使用率的指标数据并将记录发送到标准输出，您可以在 _`/var/log/messages`_ 文件中看到输出的数据。
