---
created: 2023-03-01T14:22:45 (UTC +08:00)
tags: [群晖,猫盘,Nas,tailscale]
source: https://hin.cool/posts/tailscale.html
author: W4J1e,admin@hin.cool
---
---
发表于2021-12-31|[分享](https://hin.cool/categories/%E5%88%86%E4%BA%AB/)

|字数总计:1.2k|阅读时长:3分钟|阅读量:24112|

## [](https://hin.cool/posts/tailscale.html#%E5%89%8D%E8%A8%80 "前言")前言

现在工信部在大力推动三大运营商发展ipv6，对家宽而言，未来能使用的公网ipv4地址将会更加稀缺。目前尚不知道未来运营商分配的ipv6是否可以直接互联，但是当前环境下，越来越多原本有公网ipv4的家庭宽带也逐渐变成了内网ip，这使得在公网中访问家庭内网资源变得更加困难。

好在针对这种情况有许多不同的解决方案。对于白嫖用户来说，博主此前推荐过的Frp穿透方案不失为一种好的公网访问家庭内网资源的方法。但是自建Frp服务成本较高，免费的Frp账户又有一定限制，所以很多人使用了诸如zerotier之类的内网穿透方案。

## [](https://hin.cool/posts/tailscale.html#%E4%B8%BA%E4%BB%80%E4%B9%88%E7%94%A8tailscale "为什么用tailscale")为什么用tailscale

如果你的宽带有公网ip，那么你应该不会在网上搜索本文之类的方法。

博主目前使用电信宽带，上行速率为40Mbps，家有一台猫盘刷的黑群晖，此前一直用的樱花Frp，免费用户带宽10Mbps，通过签到获得了128GB的流量，但是却极少使用过。如果是为了外网获取一些文档和图片，10Mbps的速率绝对够用，但是如果看一个码率较高的视频，则卡得让人失去继续看的欲望。而Zerotier对不同运营商的打洞成功率较低，所以放弃。

tailscale内网打洞方案，使用开源的 WireGuard 协议启用加密的点对点（P2P）连接，这意味着只有您专用网络上的设备才能相互通信，并且上传速度受限于宽带上行，数据不走服务器，所以相比Frp方案而言，前者更加高速，成本更低，安全性更强，上手难度更小。

其官网用了两张图片来对比传统VPN和tailscale内网穿透的区别：

[![传统VPN连接](https://cdn.hin.cool/pic/posts/tscvpn.jpg/sy)](https://cdn.hin.cool/pic/posts/tscvpn.jpg/sy)

传统VPN连接

[![tailscale点对点连接](https://cdn.hin.cool/pic/posts/tscp2p.jpg/sy)](https://cdn.hin.cool/pic/posts/tscp2p.jpg/sy)

tailscale点对点连接

## [](https://hin.cool/posts/tailscale.html#%E7%BE%A4%E6%99%96%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8 "群晖如何使用")群晖如何使用

Tailscale支持多个操作系统的客户端，其中包括群晖，并且无需使用ssh启动命令。

**①下载客户端；**

[点击这里进入官网下载](https://tailscale.com/download)常见设备客户端，[点击这里去github下载](https://github.com/tailscale/tailscale-synology/releases)群晖套件。目前最新版本是1.18.2，releases中有许多针对不同处理器的spk，大猫盘应当选择x86版本。

**②安装群晖客户端；**

打开套件中心，点击右上角手动安装，选择下载好的安装包，下一步并同意DSM的警告，点击完成。

**③登陆账号；**

打开刚安装的套件，会弹出一个登陆的页面，目前支持使用Goolge、微软和github账户登陆。自定义邮箱登陆为收费服务。

[![tailscale登陆界面](https://cdn.hin.cool/pic/posts/tsclogin.jpg/sy)](https://cdn.hin.cool/pic/posts/tsclogin.jpg/sy)

tailscale登陆界面

登陆完成后再次打开该套件，即可看到当前的状态，如：

[![taiscale登陆成功](https://cdn.hin.cool/pic/posts/tsclogind.jpg/sy)](https://cdn.hin.cool/pic/posts/tsclogind.jpg/sy)

taiscale登陆成功

**④安装并以同样的账户登陆其它设备的tailscale。**

在其它设备（如windows、ios）上使用该客户端可能需要安装一个虚拟网络驱动器或新建一个VPN连接。在ios客户端或windows打开的网页控制台中，你可以查看当前账户下所有的登陆设备和分配给它们的内网ip地址。

如果我要在ios设备上通过tailscale访问群晖的DSFile，只需要点击客户端左上角的按钮，然后复制群晖的ip地址，像平常一样登陆DSFile即可。

## [](https://hin.cool/posts/tailscale.html#%E5%85%B3%E4%BA%8E%E4%B8%8B%E8%BD%BD%E9%80%9F%E5%BA%A6 "关于下载速度")关于下载速度

虽然是P2P连接，但是其速度相较于真正的内网还是有很大的差距。

在我电脑上通过SMB映射的驱动器上，真内网复制文件的速度可超过90MB/s，通过tailscale打洞的方式连接，速度大概在6MB/s，相比于免费版Frp限速的1M/s还是要快许多。

[![taiscale下载速度测试](https://cdn.hin.cool/pic/posts/tscdownload.jpg/sy)](https://cdn.hin.cool/pic/posts/tscdownload.jpg/sy)

taiscale下载速度测试

## [](https://hin.cool/posts/tailscale.html#%E5%8F%AF%E8%83%BD%E5%87%BA%E7%8E%B0%E7%9A%84%E9%97%AE%E9%A2%98 "可能出现的问题")可能出现的问题

由于众所周知的原因，移动对于此类服务并不是特别友好，这也是我此前用zerotier打洞失败的原因。昨天首次使用移动4G访问电信家宽中的群晖失败，在控制台中看到DNS一项菜单，尝试修改DNS为阿里的公共DNS，然后就成功了。

[![taiscale的DNS设置](https://cdn.hin.cool/pic/posts/tscdns.jpg/sy)](https://cdn.hin.cool/pic/posts/tscdns.jpg/sy)

taiscale的DNS设置

## [](https://hin.cool/posts/tailscale.html#%E6%80%BB%E7%BB%93 "总结")总结

经过昨晚和今天的多次尝试，发现使用Tailscale基本可以取代此前安装的Frp服务了。在外网观看群晖中的一个高清视频，除了刚加载的时候稍微要等两三秒，后续的播放非常流畅，即便拖动进度条也不影响，但是我无法保证未来tailscale仍然有这样的效果。

上文已经提到了内网打洞相较于Frp的优势，但是另一方面，如果Frp服务器不宕机的话，它仍然是要访问无公网IP的内网资源最稳定的办法。

2022年要来了，祝各位来年万事顺意。

版权声明: 本博客所有文章除特别声明外，均为W4J1e原创，未经允许禁止转载！

