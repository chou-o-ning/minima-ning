---
layout: post
title: "小米路由器 4C 支持 SSR"
categories: misc
---

SSR 的安装基于 OpenWrt 21.02.3 版本。

首先需要在小米路由器 4C 上安装 OpenWRT，[小米路由器 4C 烧录 OpenWRT 方法及变砖问题的解决](https://www.jianshu.com/p/c4e8279aff20) 。

配置 ssr 的编译[https://github.com/xluoke/ywb94-openwrt-ssr](https://github.com/xluoke/ywb94-openwrt-ssr)(先不要编译)

然后在 make menuconfig 的时候配置如下：
Base system: 去掉 dnsmasq 添加 dnsmasq-full (luci-app-shadowsocksR-GFW_1.2.1_all.ipk 组件会和 dnsmasq 冲突)
Network -> Web Servers/Proxies -> 添加 shadowsocks-libev-ss-local shadowsocks-libev-ss-redir shadowsocks-libev-ss-rules shadowsocks-libev-ss-tunnel (因为小米路由器 4C 的 Flash 版本 linux 内核不支持，需要修改驱动代码后重新编译，因此这些组件不能直接用 opkg install 的方式下载安装，必须自己编译)
LuCI -> Collections: 勾选 luci (支持 Web UI)
LuCI -> Applications: luci-app-shadowsocksR-GFW (选择 M，即以 ipk 方式安装)
    
编译后，烧录 firmware。并安装 luci-compat luci-lib-ipkg luci-app-shadowsocksR-GFW_1.2.1_all.ipk

配置 dnsmasq_gfwlist_ipset.conf 文件到 dnsmasq.conf（如果不配置, google 无法访问）

到此，大功告成。

因为这个博客使用Jekyll，暂时无法支持评论，因此如果有技术问题，可以发送到我的邮箱一起探讨，邮箱地址为chou dot o dot ning at gmail dot com  

