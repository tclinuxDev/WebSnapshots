---
created: 2023-02-17T13:15:10 (UTC +08:00)
tags: []
source: https://bynss.com/linux/597467.html
author: 
---

# Iptables 与 Nftables：有什么区别？ | 月灯依旧

> ## Excerpt
> 每个 Linux 管理员肯定都使用过 iptables，这是多年来一直为我们提供良好服务的长期 Linux 防火墙。 但是您可能还不熟悉 nftables，一个新来者旨在为我们提供一些急需的升级并最终取代老化的 iptables。

---
每个 Linux 管理员肯定都使用过 iptables，这是多年来一直为我们提供良好服务的长期 Linux 防火墙。 但是您可能还不熟悉 nftables，一个新来者旨在为我们提供一些急需的升级并最终取代老化的 iptables。

这 [nftables](https://bynss.com/go/?url=https://www.netfilter.org/projects/nftables/index.html) 是由 [网络过滤器](https://bynss.com/go/?url=https://www.netfilter.org/)，目前维护 iptables 的同一组织。 它是为了解决问题而创建的 [iptables](https://bynss.com/go/?url=https://linux.die.net/man/8/iptables)，即可扩展性和性能。

除了新的语法和一些升级之外，您会发现它的功能与其前身非常相似。

使用新实用程序的另一个理由是 iptables 框架变得有点复杂，iptables、ip6tables、arptables 和 ebtables 都提供不同但相似的功能。

为了 example，在 iptables 中创建 IPv4 规则和在 ip6tables 中创建 IPv6 规则并保持两者同步是非常低效的。 Nftables 旨在取代所有这些并成为一个集中的解决方案。

尽管自 2014 年以来 nftables 已包含在 Linux 内核中，但随着采用率的提高，它最近越来越受到关注。 Linux 世界的变化是缓慢的，过时的实用程序通常需要几年或更长时间才能被淘汰，取而代之的是升级的对应程序。

Nftables 正在成为推荐的首选防火墙，Linux 管理员应该更新他们的功能。 现在是学习 nftables 和更新现有 iptables 配置的好时机。

如果您已经使用 iptables 多年并且对必须学习全新实用程序的想法不太兴奋，请不要担心，我们已经在本指南中为您提供了帮助。 在本文中，我们将介绍 nftables 和 iptables 之间的差异，并展示在新的 nftables 语法中配置防火墙规则的示例。

## nftables 中的链和规则

在 iptables 中，有三个默认链：输入、输出和转发。 这三个“链”（以及其他链，如果您有任何配置）保存“规则”，iptables 通过将网络流量与链中的规则列表匹配来工作。 如果正在检查的流量与任何规则都不匹配，则将对流量使用链的默认策略（即 ACCEPT、DROP）。

Nftables 的工作方式与此类似，也有“链”和“规则”。 然而，它并没有从任何基础链开始，这使得配置更加灵活。

iptables 的一个低效领域是所有网络数据都必须遍历上述一条或多条链，即使流量不符合任何规则。 无论您是否配置了链，iptables 仍然会根据它们检查您的网络数据。

## 在 Linux 上安装 nftables

Nftables 在所有主要的 Linux 发行版中都可用，您可以使用发行版的包管理器轻松安装它。

在基于 Ubuntu 或 Debian 的发行版上，您可以使用以下命令：

```
sudo apt install nftables
```

要确保系统重新启动时 nftables 自动启动：

```
sudo systemctl enable nftables.service
```

## iptables 和 nftables 之间的语法差异

Nftables 具有与 iptables 不同且更简单的语法。 老实说，iptables 的语法总是不清楚，需要额外努力学习。 幸运的是，对于那些从 iptables 迁移的人来说，nftables 仍然接受旧语法。

您还可以使用 **iptables-翻译** 实用程序，它将接受 iptables 命令并将它们转换为 nftables 等效项。 这是查看两种语法有何不同的简单方法。

使用以下命令在 Ubuntu 和基于 Debian 的发行版上安装 iptables-translate：

```
sudo apt install iptables-nftables-compat
```

安装后，您可以将 iptables 语法传递给 iptables-translate 命令，它会返回与 nftables 等效的命令。

让我们看一些示例，以便您了解这些命令之间的区别。

### 阻止传入连接

此命令将阻止来自 IP 地址 192.168.2.1 的传入连接：

```
[email protected]:~$ iptables-translate -A INPUT -s 192.168.2.1 -j DROP
nft add rule ip filter INPUT ip saddr 192.168.2.1 counter drop
```

### 允许传入的 SSH 连接

让我们看一些更多的例子——在加固 Linux 服务器时，您通常会发现自己在 iptables 中输入的常见内容。

```
[email protected]:~$ iptables-translate -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
nft add rule ip filter INPUT tcp dport 22 ct state new,established counter accept
```

### 允许来自特定 IP 范围的传入 SSH 连接

如果要允许来自 192.168.1.0/24 的传入 SSH 连接：

```
[email protected]:~$ iptables-translate -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
nft add rule ip filter INPUT ip saddr 192.168.1.0/24 tcp dport 22 ct state new,established counter accept
```

### 允许 MySQL 连接到 eth0 网络接口

下面是 iptables 和 nftables 的语法：

```
[email protected]:~$ iptables-translate -A INPUT -i eth0 -p tcp --dport 3306 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
nft add rule ip filter INPUT iifname eth0 tcp dport 3306 ct state new,established counter accept
```

### 允许传入的 HTTP 和 HTTPS 流量

为了允许某种类型的流量，这两个命令的语法如下：

```
[email protected]:~$ iptables-translate -A INPUT -p tcp -m multiport --dports 80,443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
nft add rule ip filter INPUT ip protocol tcp tcp dport { 80,443} ct state new,established counter accept
```

从这些示例中可以看出，语法仍然与 iptables 非常相似，但命令更直观一些。

### 使用 nftables 进行日志记录

上面 nft 命令示例中的“counter”选项告诉 nftables 计算规则被触及的次数，就像 iptables 在默认情况下所做的那样。

在 nftables 中，它们是可选的并且必须指定。

```
nft add rule ip filter INPUT ip saddr 192.168.2.1 counter accept
```

Nftables 内置了用于导出配置的选项。 它目前支持 XML 和 JSON。

```
nft export xml
```

**结论**

在本文中，我解释了为什么 nftables 是 Linux 防火墙的新推荐选择。 我还列出了旧 iptables 和新 nftables 之间的许多差异，包括它们的功能和语法。

本指南向您展示了为什么要考虑升级到 nftables，以及怎样开始使用您需要熟悉的新语法，以便成功升级旧的 iptables 规则。

如果您有任何问题或建议，请在评测中告诉我。
