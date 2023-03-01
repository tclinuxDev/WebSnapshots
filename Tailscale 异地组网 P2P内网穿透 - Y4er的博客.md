---
created: 2023-03-01T10:22:15 (UTC +08:00)
tags: []
source: https://y4er.com/posts/tailscale/
author: Y4er
---
---
疫情居家办公期间，折腾了一段时间的cloudflare tunnel，但是网速时好时坏，rdp卡成ppt，向日葵在mac上也不好用，后来在v2ex上发现了Tailscale和ZeroTier两款点对点穿透工具，折腾一下发现比cf tunnel好用一万倍，nat打洞成功之后ping只有5ms，这里记录下折腾的笔记。

用GitHub账号登录就行

传统的vpn结构如图

[![https://y4er.com/img/uploads/tailscale/1.png](https://y4er.com/img/uploads/tailscale/1.png)](https://y4er.com/img/uploads/tailscale/1.png "1.png")

Tailscale的网络结构如图

[![https://y4er.com/img/uploads/tailscale/2.png](https://y4er.com/img/uploads/tailscale/2.png)](https://y4er.com/img/uploads/tailscale/2.png "2.png")

就近原则

两台机器链接如果可以穿透nat打洞成功，那么就是直连，否则就是走Tailscale的中继服务器（derp）。

两台机器其实都是基于wireguard的实现，链接上Tailscale的机器会加一个网卡用于wireguard链接。

[![https://y4er.com/img/uploads/tailscale/3.png](https://y4er.com/img/uploads/tailscale/3.png)](https://y4er.com/img/uploads/tailscale/3.png "3.png")

更详细的原理见官方的博客 [https://tailscale.com/blog/how-nat-traversal-works/](https://tailscale.com/blog/how-nat-traversal-works/)

没啥好讲的，安装对应的客户端，然后登录账号就行了。

Tailscale提供了一个类似内网dns的服务，将所有加入到Tailscale网络中的节点分配一个对应的ip和dns，这样用户不需要记ip了。

[![https://y4er.com/img/uploads/tailscale/4.png](https://y4er.com/img/uploads/tailscale/4.png)](https://y4er.com/img/uploads/tailscale/4.png "4.png")

自动打洞，work机器和poop机器在同一局域网，所以打洞用的192的段，我测试公司电脑和家里电脑同城打洞成功后只有5ms延迟，没打洞成功就是走的图中的中继DERP服务器了。

tailscale可以实现FQ，需要先开启ip转发 [https://tailscale.com/kb/1103/exit-nodes/#enable-ip-forwarding](https://tailscale.com/kb/1103/exit-nodes/#enable-ip-forwarding)

<table><tbody><tr><td role="columnheader"><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span></code></pre></td><td role="columnheader"><pre tabindex="0"><code data-lang="bash"><span><span>If your Linux system has a /etc/sysctl.d directory, use:
</span></span><span><span>
</span></span><span><span><span>echo</span> <span>'net.ipv4.ip_forward = 1'</span> <span>|</span> sudo tee -a /etc/sysctl.d/99-tailscale.conf
</span></span><span><span><span>echo</span> <span>'net.ipv6.conf.all.forwarding = 1'</span> <span>|</span> sudo tee -a /etc/sysctl.d/99-tailscale.conf
</span></span><span><span>sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
</span></span><span><span>
</span></span><span><span>
</span></span><span><span>Otherwise, use:
</span></span><span><span>
</span></span><span><span><span>echo</span> <span>'net.ipv4.ip_forward = 1'</span> <span>|</span> sudo tee -a /etc/sysctl.conf
</span></span><span><span><span>echo</span> <span>'net.ipv6.conf.all.forwarding = 1'</span> <span>|</span> sudo tee -a /etc/sysctl.conf
</span></span><span><span>sudo sysctl -p /etc/sysctl.conf
</span></span><span><span>If your Linux node uses firewalld, you may need to also allow masquerading due to a known issue. As a workaround, you can allow masquerading with this command:
</span></span><span><span>
</span></span><span><span>firewall-cmd --permanent --add-masquerade
</span></span></code></pre></td></tr></tbody></table>

通过在国外vps上运行

<table><tbody><tr><td role="columnheader"><pre tabindex="0"><code><span>1
</span></code></pre></td><td role="columnheader"><pre tabindex="0"><code data-lang="fallback"><span><span>sudo tailscale up --advertise-exit-node
</span></span></code></pre></td></tr></tbody></table>

然后网页控制台中这个节点点击`Edit route settings`开启此功能

[![https://y4er.com/img/uploads/tailscale/5.png](https://y4er.com/img/uploads/tailscale/5.png)](https://y4er.com/img/uploads/tailscale/5.png "5.png")

然后在windows的托盘图标中，可以选择本机所有流量走出口节点

[![https://y4er.com/img/uploads/tailscale/6.png](https://y4er.com/img/uploads/tailscale/6.png)](https://y4er.com/img/uploads/tailscale/6.png "6.png")

因为每个账号只有20台设备数，所以可能会遇到需要开放一整个网段192.168.1.1/24的情况，我还没遇到，读者看文档自己配置吧。

[https://tailscale.com/kb/1019/subnets/](https://tailscale.com/kb/1019/subnets/)

打洞成功率和网络情况有关系，所以为了链接成功率和保证低延迟，可以自建derp中继服务器，我不需要，所以读者移步 [https://icloudnative.io/posts/custom-derp-servers/](https://icloudnative.io/posts/custom-derp-servers/)

向日葵、蒲公英、anydesk、todesk什么的都不考虑

1.  parsec
2.  n2n
3.  Nebula
4.  ZeroTier
5.  tinc

相对来说tailscale界面好看，功能也不差，主要是稳定。

1.  [https://icloudnative.io/posts/custom-derp-servers/](https://icloudnative.io/posts/custom-derp-servers/)
2.  [https://tailscale.com/kb/](https://tailscale.com/kb/)
3.  [https://tailscale.com/blog/how-nat-traversal-works/](https://tailscale.com/blog/how-nat-traversal-works/)

这篇文章更多是记录一下自己折腾的东西，水文。

文笔垃圾，措辞轻浮，内容浅显，操作生疏。不足之处欢迎大师傅们指点和纠正，感激不尽。
