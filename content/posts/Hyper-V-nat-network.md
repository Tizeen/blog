---
title: "Hyper-V Nat Network"
date: 2022-08-24T16:06:29+08:00
tags: ["Hyper-V"]
author: "likungong"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: true
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---

## 简介

Hyper-V 的 `Default Switch` 网络，虚拟机能上网，但是重启之后 IP 可能会发生变化，用 SSH 连接时特别不方便。

这个时候，可以创建一个 `NAT Network`，然后给虚拟机设置固定 IP，解决这个问题。

## 配置

本过程使用 `PowerShell` 创建

1. 管理员模式打开 `PowerShell`

2. 创建一个交换机

```PowerShell
New-VMSwitch -SwitchName "mySwitch" -SwitchType Internal

Get-NetAdapter
```

记住刚才创建的网卡 `ifIndex Status` 数字，后续需要使用。

3. 创建NAT网关

`InterfaceIndex` 就是虚拟机编号

```PwerShell
New-NetIPAddress -IPAddress 192.168.124.1 -PrefixLength 24 -InterfaceIndex 65
```

4. 创建NAT网络

```PowerShell
 New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.124.0/24
```

5. 在虚拟机设置中，将网络适配器换成刚才创建的交换机，进入虚拟机后还**需要手动配置静态 IP，刚才创建的交换机是没有 DHCP 功能的**，注意 DNS 也需要配置，不要配置成 `Nat Gateway IP`，不然上不了外网

## 来源

- [Set up a NAT network](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network)

## 后续

已经不用 Hyper-V 了，Hyper-V 会导致一些 VPN 软件有异常，初步判断是 DNS 相关的问题。