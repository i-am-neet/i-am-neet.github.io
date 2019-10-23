---
layout: single
title: 'Linux找不到網卡、網卡設定'
date: 2018-07-02 12:06
comments: true
categories: Development-Environment
tags: Linux Driver Network
---
最近重灌一台Intel NUC，要在上面跑Ubuntu 16.04

但遇到開機後接網路線怎麼都沒反應

而且裝了一個新的無線網路卡，系統也無法讀取到，想用無線網路找資料也沒辦法真的很痛苦

總之下面是解決的過程:

```bash
$ ifconfig
# 查詢系統資訊, ethX是網路孔的網卡
$ dmesg | grep ethX
```

查看一下是否有讀不到網卡等訊息

到Intel支援下載[Linux e1000e 驅動](https://www.intel.com/content/www/us/en/support/articles/000005480/network-and-i-o/ethernet-products.html)

```bash
$ tar zxf e1000e-<x.x.x>.tar.gz
$ sudo make install
# 安裝位置可能是在/lib/modules/<KERNEL VERSION>/kernel/drivers/net/e1000e/e1000e.[k]o
# 或是/lib/modules/<KERNEL VERSION>/updates/drivers/net/ethernet/intel/e1000e/e1000e.[k]o
$ insmod /lib/<PATH TO e1000e>/e1000e.[k]o
$ modprobe e1000e insmod e1000e
```

下面是網卡設定的方法
```bash
$ sudo vim /etc/network/interfaces
```
```vim
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto ethX
allow-hotplug ethX
iface ethX inet dhcp

## 固定IP
#auto ethX
#iface ethX inet static
#  address 192.168.X.X
#  netmask 255.255.255.0
#  gateway 192.168.0.1
```

# 重新啟動網卡
```bash
$ sudo /etc/init.d/networking restart
## 或是針對單一網卡重新啟動
$ sudo ifdown ethX
$ sudo ifup ethX
```

# 無線網卡設定
## 搜尋無線裝置
```bash
$ sudo iwlist *wlanX* scan | grep ESSID
```
## 無線網卡設定
編輯```/etc/network/interface```
```bash
auto lo
iface lo inet loopback

auto wlanX
iface wlanX inet static
  address 192.168.0.X
  netmask 255.255.255.0
  network 192.168.0.0
  broadcast 192.168.0.255
  gateway 192.168.0.1
  dns-nameservers 8.8.8.8
  wpa-ssid "AP_DEVICE"
  wpa-psk "PASSWORD"
```

# 多網卡指定預設路由
預設所使用的gateway可能不是你所希望的對外閘道
```bash
$ route
Kernel IP routing table
Destination   Gateway       ...
**default       192.168.X.X   ...**
...
...
# 更改預設路由
$ sudo route del default
$ sudo route add default gw *192.168.0.1*
```
