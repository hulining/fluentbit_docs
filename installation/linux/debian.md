# Debian

Fluent Bit 发布的软件包为 **td-agent-bit**，可用于最新\(和旧\)稳定版 Debian 系统: Buster, Stretch 和 Jessie。

## Server GPG key

第一步，将我们的服务器 GPG 密钥添加到您的密钥环中，这样就可以获取我们签名的软件包:

```text
$ wget -qO - https://packages.fluentbit.io/fluentbit.key | sudo apt-key add -
```

## Update your sources lists

在 Debian 上，您需要将我们的 APT 服务器实例添加到您的源列表中，请在 **/etc/apt/sources.list** 文件中添加如下内容:

#### Debian 10 \(Buster\)

```text
deb https://packages.fluentbit.io/debian/buster buster main
```

#### Debian 9 \(Stretch\)

```text
deb https://packages.fluentbit.io/debian/stretch stretch main
```

#### Debian 8 \(Jessie\)

```text
deb https://packages.fluentbit.io/debian/jessie jessie main
```

### Update your repositories database

现在，让您的系统更新 _apt_ 数据库:

```bash
$ sudo apt-get update
```

## Install TD Agent Bit

现在，使用以下 _`apt-get`_ 命令，您可以安装最新的 _td-agent-bit_:

```text
$ sudo apt-get install td-agent-bit
```

Now the following step is to instruct _systemd_ to enable the service:

```bash
$ sudo service td-agent-bit start
```

如果进行状态检查，应该会看到类似如下的输出:

```bash
sudo service td-agent-bit status
● td-agent-bit.service - TD Agent Bit
   Loaded: loaded (/lib/systemd/system/td-agent-bit.service; disabled; vendor preset: enabled)
   Active: active (running) since mié 2016-07-06 16:58:25 CST; 2h 45min ago
 Main PID: 6739 (td-agent-bit)
    Tasks: 1
   Memory: 656.0K
      CPU: 1.393s
   CGroup: /system.slice/td-agent-bit.service
           └─6739 /opt/td-agent-bit/bin/td-agent-bit -c /etc/td-agent-bit/td-agent-bit.conf
...
```

**td-agent-bit** 的默认配置是收集 CPU 使用率的指标数据并将记录发送到标准输出，您可以在 _\`/var/log/syslog_\`\_ 文件中看到输出的数据。

