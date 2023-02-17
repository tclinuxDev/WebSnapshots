---
created: 2023-02-17T13:17:40 (UTC +08:00)
tags: [iptables和nftables有什么区别?]
source: https://www.linuxrumen.com/rmxx/1669.html
author: 
---

# iptables和nftables有什么区别?-智传网优云课堂

> ## Excerpt
> 在本文中，我们将讨论nftables和iptables之间的区别，并展示在新的nftables语法中配置防火墙规则的示例。

---
## 1\. 前言

在本文中，我们将讨论`nftables`和`iptables`之间的区别，并展示在新的`nftables`语法中配置防火墙规则的示例。

每个Linux管理员肯定都使用过`iptables`，这是一个长期存在的Linux防火墙，多年来为我们提供了良好的服务。但是您可能还不熟悉`nftables`，它是一个Linux系统的新工具，旨在为我们提供一些非常需要的升级，并最终取代年纪大的`iptables`。

![iptables和nftables有什么区别?](http://images.linuxrumen.com/linux/iptables-vs-nftables/linux-firewall.png-1)

## 2\. 为什么要使用nftables而不是iptables?

`nftables`是由`Netfilter`开发的，该组织与当前维护`iptables`的组织相同。它的创建是为了解决`iptables`的问题，即可伸缩性和性能。

除了新的语法和一些升级之外，您还会发现它的功能与以前的版本非常相似。

新推出的`nftables`的另一个理由是，`iptables`框架已经变得有点复杂，`iptables`、`ip6tables`、`arptables`和`ebtables`都提供了不同但类似的功能。

例如，在`iptables`中创建`IPv4`规则和在`ip6tables`中创建`IPv6`规则并保持二者同步是非常低效的。`nftables`的目标是取代所有这些，成为一个集中的解决方案。

尽管`nftables`从2014年开始就被包含在Linux内核中，但是随着采用范围的扩大，它最近获得了更多的关注。在Linux世界中，变化是缓慢的，过时的实用程序通常需要几年甚至更长时间才能被淘汰，取而代之的是升级版。

`nftables`正在成为推荐的防火墙，Linux管理员有必要更新它们的工具列表。现在是学习`nftables`和更新现有`iptables`配置的好时机。

如果您已经使用`iptables`很多年了，并且对于必须学习一种全新的实用程序这个想法不是很兴奋，不要担心，我们已经在本指南中介绍了。  
![linux-firewall-02](http://images.linuxrumen.com/linux/iptables-vs-nftables/linux-firewall-02.png-1)

## 3\. nftable中的链(chains)和规则(rules)

在`iptables`中，有三个默认的链:输入(input)、输出(output)和转发(forward)。这三个“链”(以及其他链，如果您配置了任何链)包含“rules/规则”，`iptables`通过将网络流量与链中的规则列表匹配来工作。如果所检查的流量与任何规则都不匹配，则链的默认策略将用于流量，即接受(ACCEPT), DROP(丢弃)。

`nftables`的工作原理与此类似，也有“链”和“规则”。但是，它没有从任何基础链开始，这使得配置更加灵活。

`iptables`的一个低效之处是，所有网络数据都必须通过上述一个或多个链，即使流量与任何规则都不匹配。无论是否配置了链，`iptables`仍然会根据这些链检查网络数据。

## 4\. 在Linux上安装`nftables`

`nftables`在所有主要的Linux发行版中都可用，您可以使用发行版的包管理器轻松安装它。

在Ubuntu或基于debian的发行版上，你可以使用以下命令:

确保`nftables`在系统重新启动时自动启动:

## 5\. `iptables`和`nftable`之间的语法差异

`nftables`的语法与`iptables`不同，也简单得多。说实话，`iptables`语法总是不清楚，需要额外的学习。幸运的是，对于那些从`iptables`迁移过来的人来说，`nftables`仍然接受旧的语法。

![Linux iptables防火墙](http://images.linuxrumen.com/linux/iptables-vs-nftables/iptables.jpg-1)

您还可以使用`iptables-translate`实用程序，它将接受`iptables`命令并将它们转换为`nftables`。这是查看这两种语法差异的简单方法。

使用以下命令在Ubuntu和基于debian的发行版上安装`iptables-translate`:

安装之后，可以将`iptables`语法传递给`iptables-translate`命令，它将返回`nftables`等效的命令。

让我们来看一些示例，以便您可以了解这些命令之间的不同之处。

### 5.1 阻塞入方向的流量

这个命令会阻止来自IP地址192.168.2.1的连接:

```
nft add rule ip filter INPUT ip saddr 192.168.2.1 counter drop
```

### 5.2 允许入口方向SSH连接

让我们来看看更多的例子—在加固Linux服务器时，您通常会发现自己在`iptables`中键入的一些常见内容。

```
nft add rule ip filter INPUT tcp dport 22 ct state new,established counter accept
```

### 5.3 允许来自特定IP范围的SSH连接

如果你想允许192.168.1.0/24的SSH连接:

```
nft add rule ip filter INPUT ip saddr 192.168.1.0/24 tcp dport 22 ct state new,established counter accept
```

### 5.4 允许MySQL连接到eth0网络接口

以下是`iptables`和`nftables`的语法:

### 5.5 允许入口方向的HTTP和HTTPS流量

为了允许特定类型的流量，下面是这两个命令的语法:

从这些示例中可以看到，语法仍然与`iptables`非常相似，但是命令更直观一些。

使用nftables作日志记录  
上面的`nft`命令示例中的`counter`选项告诉`nftables`计算一个规则被触发的次数，就像`iptables`在默认情况下所做的那样。

在`nftables`中，它们是可选的，必须指定它们。

nftables内置了用于导出配置的选项。它目前支持XML和JSON。

## 6\. 结论

在本文中，我解释了为什么推荐`nftables`是Linux防火墙的新选择。我还列出了旧的`iptables`和新`nftables`之间的许多不同之处，包括它们的功能和语法。

本指南向您展示了为什么要考虑升级到`nftables`，以及如何开始使用您需要熟悉的新语法，以便成功升级旧的iptables规则。

如果你有任何问题或建议，请在评论中告诉我。
