---
title: 將 DN42 的網段分配給其他設備
published: 2024-12-23
description: '在上一篇文章我們已經註冊了 DN42 的相關資訊，並宣告了 IP 網段，這次我們要把 IP 網段中的 IP 分給其他設備使用'
image: ''
tags: [dn42, vpn, bgp, linux]
category: 'Network'
draft: false
---

## 前言

在上一篇文章我們已經註冊了 DN42 的相關資訊，並宣告了 IP 網段，這次我們要把 IP 網段中的 IP 分給其他設備使用。

## 目標

我們將使用 Wireguard 與 VPS 建立隧道，並透過該隧道分配 IP 給其他設備及訪問 DN42。

homesrv Wireguard -> VPS -> Wireguard -> Peer -> DN42

VPS wg_clt_dn42: `192.168.50.1/24`
homesrv wg_clt_dn42: `192.168.50.2/24, 172.21.75.130/27`

## 開始配置

首先在 VPS 上配置 Wireguard 通道：

```conf
[Interface]
Address = 192.168.50.1/24
PrivateKey = <VPS 的 Wireguard 私鑰>
ListenPort = <VPS 的連接埠>

[Peer]
PublicKey = <homesrv 的 Wireguard 公鑰>
PresharedKey = <預共享金鑰>     # 可選
AllowedIPs = 192.168.50.2/32, 172.21.75.130/32
```

之後輸入這串指令啟動隧道：

```
wg-quick up wg_[隧道名]
```

接著是 homesrv 的設定，一樣配置 Wireguard 通道：

```conf
[Interface]
PrivateKey = <homesrv 的 Wireguard 私鑰>
Address = 192.168.50.2/24, 172.21.75.130/27
DNS = 192.168.50.1     # 可選，我在 VPS 上建了 DNS 服務進行公網及 DN42 域名的分流，使其可以同時訪問公網及 DN42

[Peer]
PublicKey = <VPS 的 Wireguard 公鑰>
PresharedKey = <預共享金鑰>     # 可選
AllowedIPs = 10.0.0.0/8, 172.20.0.0/14, 172.31.0.0/16, 192.168.50.0/24, fd00::/8, fe80::/64
Endpoint = srv-a1.bob0623.net:51825
```

之後啟動隧道後就可以 Ping 看看 DN42 的服務了

## 結語

配置其實很簡單，但我研究了大半天才弄出來，這裡我沒有教 IPv6 的配置方法，不過大同小異，我們下篇文章見。