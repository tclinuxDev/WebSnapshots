---
created: 2023-02-16T13:33:18 (UTC +08:00)
tags: []
source: https://www.cnblogs.com/nuoforever/p/14176630.html
author: soso101
            粉丝 - 1
            关注 - 2
        
    
    
    
    
                +加关注
---

# 在Ubuntu上启用和禁用NetworkManager - soso101 - 博客园

> ## Excerpt
> NetworkManager是一项后端服务，用于控制Ubuntu操作系统上的网络接口。NetworkManager的替代方法是systemd-networked。在Ubuntu桌面上，网络管理器是通过

---
NetworkManager是一项后端服务，用于控制Ubuntu操作系统上的网络接口。NetworkManager的替代方法是systemd-networked。在Ubuntu桌面上，网络管理器是通过图形用户界面管理网络界面的默认服务。因此，如果要通过GUI配置IP地址，则应启用网络管理器。

Ubuntu网络管理器的替代方法是systemd-networkd，这是Ubuntu服务器18.04中的默认后端服务。

因此，如果要禁用NetworkManager，则应启用网络服务，而在网络管理器运行时最好禁用网络服务。

## 禁用网络管理器并启用systemd-networkd

首先，运行以下命令以禁用NetworkManager：

```
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
sudo systemctl mask NetworkManager
```

接下来，启动并启用systemd-networkd：

```
sudo systemctl unmask systemd-networkd.service
sudo systemctl enable systemd-networkd.service
sudo systemctl start systemd-networkd.service
```

将接口配置添加到netplan配置文件（在/etc/netplan目录中）：

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: yes
```

通过运行以下命令来应用更改：

```
sudo netplan apply
```

在前面的示例中，我们将enp0s3接口配置为从DHCP服务器租用IP地址。如果要设置静态IP地址，请单击以下链接以了解[如何使用netplan配置静态IP地址](https://www.configserverfirewall.com/ubuntu-linux/configure-ubuntu-server-static-ip-address/)。

### 启用NetworkManager并禁用systemd-networkd

可以通过以下步骤启动和启用Ubuntu Network Manager（在Ubuntu服务器中不建议这样做）。

首先，停止系统联网服务：

```
sudo systemctl disable systemd-networkd.service
sudo systemctl mask systemd-networkd.service
sudo systemctl stop systemd-networkd.service
```

在Ubuntu上安装NetworkManager：

```
sudo apt-get install network-manager
```

打开/etc/netplan目录中的.yaml配置文件，并用以下内容替换现有配置：

```
network:
  version: 2
  renderer: NetworkManager
```

使用netplan命令为NetworkManager生成特定于后端的配置文件：

```
sudo netplan generate
```

启动NetworkManager服务：

```
sudo systemctl unmask NetworkManager
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
```

现在启用了NetworkManager，可以使用nmcli命令通过GUI或从命令行完成接口配置。

尽管可以通过网络管理器在Ubuntu服务器上管理网络，但是它已被[Netplan](https://www.configserverfirewall.com/ubuntu-linux/configure-ubuntu-server-static-ip-address/#ubuntu-netplan-yaml)取代。因此，建议在Ubuntu Server 18.04及更高版本上使用systemd-networkd。
