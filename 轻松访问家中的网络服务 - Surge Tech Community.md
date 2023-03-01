---
created: 2023-03-01T15:15:46 (UTC +08:00)
tags: []
source: https://community.nssurge.com/d/5/2
author: 
---
---
-   Edited

相信各位在自己的家中都有一些长期联网的设备，在外出时可能需要访问，比如 NAS 等，通常有两个方式实现：

1.  直接设置 NAT 端口转发：配置简单，但是问题在于直接将相应的服务暴露在公网上，安全性很低，且当增加服务时需要不断去维护端口转发表。
2.  架设 VPN 服务器：配置起来比较麻烦，但是安全性高，连上之后可以直接访问内网内所有设备。但是 VPN 发动慢，容易失败，性能较差。

利用 Surge iOS，可以做到无缝的远程安全访问内网设备。

首先我们需要做好以下准备：

1.  内网中需要一个可以运行代理服务器的设备（如 RaspberryPi、路由器、iMac Mini）
2.  DDNS 服务及客户端（DSM 可作为客户端，很多路由器也支持，服务推荐使用花生壳或 no-ip.com）

正式开始：

1.  请将内网的网段修改为一个不常见的网段，以避免与在外使用的 WiFi 网络的内网 IP 冲突导致问题，如 192.168.0.1/24 和 192.168.1.1/24 就是两个极为常见的网段，而 192.168.150.0/24 则很罕见，修改方式请参照路由器的配置说明。
    
2.  我们需要在内网建立一个代理服务器，如果在家中有一个长期开启的 macOS 设备，推荐直接使用 Surge Mac 运行 Snell 代理服务，配置方式参见手册，[https://manual.nssurge.com/others/snell-server.html](https://manual.nssurge.com/others/snell-server.html)。  
    如果没有，则可以选择任意设备，配置运行 snell-server 或者 ss-server，具体配置方式请参见程序的帮助。此处假设该设备 IP 为 192.168.150.4，代理服务端口号 6160。
    
3.  配置路由器，将 6160 端口的 TCP 访问转发至 192.168.150.4 的 6160。
    
4.  配置一个 DDNS，获得指向家中 IP 的域名，如：home.yach.me
    
5.  配置 Surge 规则，首先加入相应的 Proxy 设置
    
    ```
    [Proxy]
    HomeProxy = snell, home.yach.me, 6160, psk=password
    ```
    
    为了在内网中使用时，不再通过代理进行转发，可以使用 SSID Group。
    
    ```
    [Proxy Group]
    Home=ssid, default=HomeProxy, HomeSSID=DIRECT, HomeSSID2=DIRECT
    ```
    
    最后加入相应的规则
    
    ```
    [Rule]
    IP-CIDR,192.168.150.0/24,Home,no-resolve
    ```
    
6.  搞定，启动 Surge。现在在任何地方访问家中的设备都畅通无阻了。
    

（注 1：不要使用 home.yach.me 这样的容易猜测的域名，有导致自己的公网 IP 暴露的可能性。）

25 days later

[ifsc01](https://community.nssurge.com/d/5/2) Snell现在还不支持监听IPv6地址吧……  
我在GitHub上提过，不过yachen没回复。

a month later

按照教程，在内网的Ubuntu上搭建了snell，但是发现在外网的时候，可以通过snell代理访问任何外部网站，但是无法通过snell访问任何内网IP（我的网段是192.168.198.0/24），请问还需要做什么特别设置么？

更新下，发现用ios的surge是可以的，但是mac的surge不行……

18 days later

[shaoshuang](https://community.nssurge.com/d/5/5) 有没有针对内网地址，设定个规则呢比如

```
IP-CIDR,192.168.198.0/24,Home-Group,no-resolve
```

14 days later

[Larkin](https://community.nssurge.com/d/5/6) 设置了，后来发现其实MAC要解决只要打开增强模式就好了

a month later

[SurgeTeam](https://community.nssurge.com/d/5/9)  
\[Rule\]  
IP-CIDR,192.168.150.0/24,Home,no-resolve

这条好像和配置文件中开始部分的“跳过代理”中的设置冲突，一般都会把内网地址加入“跳过代理”这个段中，这个怎么解决？有没有什么设置可以在“跳过代理”中排除特定的地址或地址段

6 months later

无法连接，日志如下。

Events  
11:51:41.992755 Rule matched: IP-CIDR 192.168.xx.0/24  
11:51:41.993245 Use the last successful address: $myIPAddress  
11:51:41.993424 Connecting with address: $myIPAddress  
11:51:42.083760 Connected to address $myIPAddress in 90ms  
11:51:42.083906 TCP connection established  
11:51:42.437074 Remote closed read stream (Half-closed)  
11:51:42.437470 Disconnect with reason: Closed by client

19 days later

使用机场的托管配置，目前是不是不支持通过 module 的形式配置 gateway 的 \[snell server\] ?

18 days later

[@SurgeTeam](https://community.nssurge.com/u/SurgeTeam) 按照教程设置之后，发现在外网访问家里的设备的时候，IP会出现在Surge for mac的设备里面，这个有什么设置可以避免吗？勾选自动忽略没有配置的设备也不行……

16 days later

不知道snell server是否有计划支持IPv6环境下的监听？

6 months later

文档里跳转的 snell 的仓库已经404了，该如何安装呢？

2 months later

[QQ29701](https://community.nssurge.com/d/5/10) 要排除网段192.168.150.0/24（即在192.168.0.0/8范围内但不包括192.168.150.0/24的所有地址），可以使用CIDR表示法中的“排除”符号，即“！”符号，表示为：

192.168.0.0/8 !192.168.150.0/24

这表示在192.168.0.0/8范围内，排除192.168.150.0/24网段中的所有地址，即得到的地址范围为192.168.0.0/8到192.168.149.255/24和192.168.151.0/24到192.168.255.255/24。

Write a Reply...
