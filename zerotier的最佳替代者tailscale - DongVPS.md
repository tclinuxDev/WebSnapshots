---
created: 2023-03-01T14:18:41 (UTC +08:00)
tags: []
source: https://www.dongvps.com/2022-08-29/zerotier%E7%9A%84%E6%9C%80%E4%BD%B3%E6%9B%BF%E4%BB%A3%E8%80%85tailscale/
author: 
---
---
## [Linux](https://www.dongvps.com/category/linux/) / [Nas](https://www.dongvps.com/category/nas/) · 2022年8月29日 **[3](https://www.dongvps.com/2022-08-29/zerotier%e7%9a%84%e6%9c%80%e4%bd%b3%e6%9b%bf%e4%bb%a3%e8%80%85tailscale/#comments)**

前面讲了zerotier，我的评价是zerotier是最简单易用的内网穿透工具，上手容易，操作逻辑非常清晰。

但是没有一个产品会满足所有人的需求，在众多的同类产品中，tailscale无疑是最好的哪个，但是你要问我zerotier与tailscale哪个好，我只能说，我也不知道。

## **Tailscale 是什么**

![](https://www.dongvps.com/wp-content/uploads/2022/08/chrome_4eLXjxT6wz-1024x690.png)

WireGuard 是一个易于配置、快速且安全的**_开源 VPN_**，它利用了最新的加密技术。目的是提供一种更快、更简单、更精简的通用 VPN，它可以轻松地在树莓派这类低端设备到高端服务器上部署。

但是不同于VPN，WireGuard 没有 VPN 网关，所有节点之间都可以**_点对点（P2P）连接_**，全互联模式（full mesh），效率更高，速度更快，成本更低。

![](https://www.dongvps.com/wp-content/uploads/2022/08/site-to-site-complex-1024x391.png)

wireguard图

简而言之，我们可以将 Tailscale 看成是更为易用、功能更完善的 WireGuard。

Tailscale 是一款商业产品，个人用户在接入设备**_不超过 20 台的情况下是可以免费使用的_**

![](https://www.dongvps.com/wp-content/uploads/2022/08/new-pricing-1024x566.png)

（虽然有一些限制，比如子网网段无法自定义，且无法设置多个子网，只能配置一个虚拟路由）。

对于大部份用户来说， Tailscale 已经足够了，如果你有更高的需求，比如自定义网段，可以选择付费。

相比于zerotier，Tailscale更容易添加虚拟路由，任何一个节点就都可以被设置为路由，从而让所在内网被其他节点访问到。

如果你有更高的需求，更多的客户端数量，更多的虚拟网段，更多的虚拟路由，那就需要headscale，不过本期内容是tailscale，headscale等下期。

tailscale可以做什么

-   让内网内所有的设备和服务都可以被其他节点访问到

## 使用tailscale

#### 1、申请账户，可以直接使用谷歌账号、github账号

官方网站：申请账号 [Tailscale](https://tailscale.com/)

tips:建议使用微软账号，没有被墙。

登陆成功后，主页上赫然显示所有平台的客户端下载按钮

#### 2、首先是windows，下载安装启动，一气合成。

点击会打开浏览器跳转到tailscale主页，登录状态的话会显示认证成功。

windows客户端就这么好了。

```
tailscale up --accept-routes=true --accept-dns=false --advertise-routes=192.168.188.0/24
```

#### 3、linux客户端

直接下载。例如：

[https://pkgs.tailscale.com/stable/](https://pkgs.tailscale.com/stable/)

```
wget https://pkgs.tailscale.com/stable/tailscale_1.28.0_amd64.tgz
```

解压：

```
tar zxvf tailscale_1.28.0_amd64.tgz x tailscale_1.28.0_amd64/ x 
```

`tailscale_1.28.0_amd64/tailscale x tailscale_1.28.0_amd64/tailscaled x tailscale_1.28.0_amd64/systemd/ x tailscale_1.28.0_amd64/systemd/tailscaled.defaults x tailscale_1.28.0_amd64/systemd/tailscaled.service`

将二进制文件复制到官方软件包默认的路径下：

```
cp tailscale_1.28.0_amd64/tailscaled /usr/sbin/tailscaled
cp tailscale_1.28.0_amd64/tailscale /usr/bin/tailscale
chmod +x /usr/sbin/tailscaled
chmod +x /usr/bin/tailscale
```

将 systemD service 配置文件复制到系统路径下：

```
cp tailscale_1.28.0_amd64/systemd/tailscaled.service /lib/systemd/system/tailscaled.service
```

将环境变量配置文件复制到系统路径下：

```
cp tailscale_1.28.0_amd64/systemd/tailscaled.defaults /etc/default/tailscaled
```

启动 tailscaled.service 并设置开机自启：

`$ systemctl enable --now tailscaled`

查看服务状态：

`$ systemctl status tailscaled`

Tailscale 接入：

```
tailscale up --accept-routes=true --accept-dns=false
# tailscale up --accept-routes=true --accept-dns=false --advertise-routes=192.168.188.0/24
```

这里推荐将 DNS 功能关闭，因为它会覆盖系统的默认 DNS。如果你对 DNS 有需求，可自己研究官方文档，这里不再赘述。

打开后台

#### 4、openwrt客户端，下载对应版本

[https://github.com/adyanth/openwrt-tailscale-enabler](https://github.com/adyanth/openwrt-tailscale-enabler)

![](https://www.dongvps.com/wp-content/uploads/2022/08/chrome_0WuS1rXxHe.png)

-   解压到根目录:

```
tar x -zvC / -f openwrt-tailscale-enabler-v1.28.0-e707d17-autoupdate.tgz
```

-   安装依赖：

```
opkg update opkg install libustream-openssl ca-bundle kmod-tun
```

-   启动服务，第一次启动

```
/etc/init.d/tailscale start 
tailscale up --accept-dns=false --advertise-routes=192.168.188.0/24
```

因为是第一次启动，会下载 tailscale packag， 会非常慢，等待完成

设置自启动

`/etc/init.d/tailscale enable`

检查是否存在以下文件

`ls /etc/rc.d/S*tailscale*`

同样的，完成后去tailscale后台查看是否获取到ip了

设置为虚拟路由的情况

此时未完，打开openwrt的接口，查看是否由tailscale接口了，如果没有就创建一个，防火墙区域，设置为lan

![https://yomis.blog/content/images/2022/04/Screenshot-2022-04-23-at-04.01.46.jpg](https://yomis.blog/content/images/2022/04/Screenshot-2022-04-23-at-04.01.46.jpg)

此时此刻，虚拟路由应该生效了，openwrt所在子网的设备均可被其他节点访问到

#### 5、群晖客户端

[https://github.com/tailscale/tailscale-synology](https://github.com/tailscale/tailscale-synology)

群晖客户端简单了，下载对应版本的包，手动安装

启动，打开tailscale登录页面，认证

群晖担任虚拟路由的情况，同样的，去ssh执行

```
tailscale up --accept-routes=true --accept-dns=false --advertise-routes=192.168.188.0/24
```

记得，启动虚拟路由必须去tailscale后端去打开，路由才会生效。

至于安卓与ios客户端就不说了，大同小异的，macos的请自行查找吧

## 结语

tailscale作为一款商业软件限制还是有的，无法自定义虚拟子网段，最多20台设备的额度，但是大部分人的需求已经足够满足了

至于打洞后的速度，除了打洞本身需要的时间有一些不同，zerotier与tailscale基本没有性能上的差异，他们都是最优秀的内网穿透工具。

本期视频到这里了，我是张大七，下期见。
