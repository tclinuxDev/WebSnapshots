---
page-title: "使用 ZeroTier 异地组网 – BOXKS"
url: https://blog.boxks.com/archives/zerotier-0ffsite-networking/
date: "2023-02-03 14:56:23"
---
![](https://blog.boxks.com/wp-content/uploads/2020/01/zerotier-0ffsite-networking-2020-01-16.png)

店铺装有监控，虽然和家里只差一条街，但是我需要将两个地方的网络组建一个局域网，之前用 OpenVPN，但是配置麻烦，效果一般。然后发现 ZeroTier 组网很简单。

路由器层级需要懂 Linux 命令项操作才可以，如果你只是电脑之间的话直接用客户端界面就可以。

注意：ZeroTier 分配 IP 时候需要两位数的 IP 才可以正常分配，一位数的会出现不能正常分配的情况。不知道我是不是个例。

首先，你要打电话给运营商客服，说你家有监控，需要公网 IP，电信一般会给的，联通看运气，移动基本上不可能给的。没有公网 IP 的话 ZeroTier 会不是很稳定。

![](https://blog.boxks.com/wp-content/uploads/2020/01/%E7%BD%91%E7%BB%9C%E5%9B%BE-g.png)

在店铺用 Asus AC87u 路由器，然后我使用 Raspberry Pi Zero 加 USB 网卡。

家里用的路由器是 3865u 的软路由，使用 PVE 虚拟一个 OpenWrt 和黑群晖，家里的监控也直接用黑群晖录像，因为群晖的 Apps 很好用，以后打算入手一个 4 盘位的白群晖，这样就能正常推送消息了。

两个地方的路由器 IP 段是不能一样的！也就是路由器地址不要一样是 192.168.1.1，需要分别修改成其他的，例如：家里的修改成 192.168.2.1，店铺路由器改成 192.168.3.1。

### 注册 ZeroTier

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_00-31-50-1024x726.png)

打开 [https://www.zerotier.com](https://www.zerotier.com/) 注册登录之后，在顶部菜单栏点击 Networks，在下面点击 Create a Networks 按钮，这样就新建了一个虚拟局域网。然后点入去。

如果你只是电脑和电脑或者电脑和手机之间组局域网直接在 [https://www.zerotier.com/download/](https://www.zerotier.com/download/) 下载对应客户端就然后加入网络就可以了。

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_00-32-38-1024x726.png)

Network ID 就是你网络的 ID，这个 ID 最好不要公开出来，因为只需要 Network ID 就可以申请加入你的局域网了。

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_00-32-38-1-1024x726.png)

Access Control 勾选 PRIVATE 确保你的网络是私有的，只有你确认过才可以加入。

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_00-32-49-1024x726.png)

IPv4 Auto-Assign 就是设置你的虚拟局域网 IP 段，默认的就可以了，不需要特别修改。

再下面的 Members 就是客户端列表，你以后分配 IP 和设置功能都在这里的列表，后面细说。

### OpenWrt 安装设置

原版 OpenWrt 的 ZeroTier 是没有图形界面的，所以需要终端连接上路由器，如果你是用国内编译好的 OpenWrt 一般服务界面都会有图形版的 ZeroTier 设置页面的。因为我用原版 OpenWrt 所以我只说终端操作。

#### 安装 ipk

SSH 连接 OpenWrt 之后，输入 `opkg install zerotier` 安装，然后 `zerotier-cli status` 启动。

#### 连接 ZeroTier

`zerotier-cli join XXXXXXXXXXX` 其中 `XXXXXXXXXXX` 是你的 Network ID，输入后就会申请加入虚拟局域网，然后你要登录 [https://my.zerotier.com/](https://my.zerotier.com/) 

在 Networks 页面进入虚拟局域网管理页面，在 Members 项就会看到你 OpenWrt 出现在列表。

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_00-33-09-1024x726.png)

先在右边 Managed IPs 项目填上一个网络 IP，我的网络是 10.147.20.0/24，这样就填 10.147.20.20，然后按左边的「![➕](https://s.w.org/images/core/emoji/14.0.0/svg/2795.svg)」就会分配一个 IP，然后最左边 Auth? 那里点上![✔](https://s.w.org/images/core/emoji/14.0.0/svg/2714.svg)，接着点击 ![🔧](https://s.w.org/images/core/emoji/14.0.0/svg/1f527.svg) 打开高级设置勾选 「Allow Ethernet Bridging」允许这个设备桥接，这样才能访问路由器下的其他设备的。

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_00-32-49.png)

返回上面的「Advanced」在「Managed Routes」下的 「Add Routes」的 Destination 填写路由器 IP 段，我的是 192.168.2.0/24，(Via) 填写分配给 OpenWrt 的虚拟 IP：10.147.20.20，然后点击 Submit 提交。

然后在终端输入 `zerotier-cli listnetworks` 看你加入的虚拟网络列表，你可以加入不止一个的虚拟网络的。

#### 设置接口和防火墙

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_01-12-48-1024x726.png)

在 OpenWrt 管理页面的 网络 -> 接口 页面添加新接口然后再物理设置那里的接口下拉选择一个新出现的一串字母和数字组合的接口。然后保存并应用。

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_01-15-34-1024x726.png)

防火墙 -> 基本设置 转发数据选择「接受」，在下面 「区域」添加一个新的选择「覆盖网络」选择刚新建的接口，入站、出站、转发都设置为接受。端口触发勾选 lan 和 wan 然后保存就好了。

接着在防火墙的自定义页面加入下面的命令，其中 `zt7xxxx2xx` 就是上面的 ZeroTier 的接口。

```
iptables -I FORWARD -i zt7xxxx2xx -j ACCEPT
iptables -I FORWARD -o zt7xxxx2xx -j ACCEPT
iptables -t nat -I POSTROUTING -o zt7xxxx2xx -j MASQUERADE
```

### 树莓派设置

#### 树莓派安装 ZeroTier

用官方的方法安装，SSH 连接树莓派输入 `curl -s https://install.zerotier.com && sudo bash` 下载并安装，然后 `sudo zerotier-cli status` 启动，`sudo zerotier-cli join XXXXXXXXXXX` 加入网络，重复上面的 OpenWrt 时候的步骤就可以了，设置好虚拟 IP 和路由就好了。

我店铺的路由器设置的是 192.168.3.0/24，分配给树莓派的虚拟 IP 是  10.147.20.33。那么「Managed Routes」就分别填对应的就好了。

#### 设置树莓派转发

根据网上[找到的教程](https://github.com/aturl/awesome-anti-gfw/blob/master/ZeroTier/ZeroTier's_VPN.md)设置好树莓派的转发功能。

#### 在路由器设置路由表

登录 AC87u 管理界面，在内部网络(LAN) – 路由设置页面启动静态路由列表，按照我图片那样填写，要加两个，就是虚拟局域网的网段和家里路由器的网段都要加上。

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_01-24-11-1024x726.png)

然后返回家里 OpenWrt 网络 -> 静态路由页面添加店铺路由的路由表。

![](https://blog.boxks.com/wp-content/uploads/2020/01/Snipaste_2020-01-16_02-54-46-1024x771.png)

静态 IPv4 路由那里添加一项，然后接口选择 ZeroTier 的接口，主机 IP 或网络就填写店铺的网络 192.168.3.0/24，IPv4 子网掩码如果你没改通常是 255.255.255.0，然后网关就填树莓派的虚拟 IP，然后保存并应用就可以了。

这样在家里就可以之直接输入店里局域网的IP直接访问店里的内网设备了，反过来，在店铺的网络下也可以直接访问家里网络的设备了。

#### 结语

异地组网比直接路由器端口映射安全很多，当然是在两个路由器之间组网才是，如果是手机在外面连接设备的话还是路由器开放端口会方便很多，不需要手机打开 Apps 在连接。

由于 ZeroTier 使用的是 UDP 连接，所以如果数据量太大的话可能会被限速。你想监控的录像是异地录像的话就最好还是路由器开放端口实际一些，这样就不会被限速了。

如果你的网络访问 ZeroTier 的节点慢的话你需要自建 moon 节点，我这里速度不错就没有建，你需要的话网上随便搜索都有很多教程了。

Moon 节点是支持 DDNS 的，你可以在 OpenWrt 设置一个域名 moon 节点，然后其他地方添加这个节点。

---

## 文章导航