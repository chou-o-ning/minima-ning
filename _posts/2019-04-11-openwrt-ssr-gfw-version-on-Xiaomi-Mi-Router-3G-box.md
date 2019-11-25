---
layout: post
title: "DIY: 基于OpenWRT和小米路由器的透明代理(SSR)(后记：支持GFW List模式)"
categories: misc
---

### 后记

我编译的源代码来自ywb94的 [ShadowsocksR-libev for OpenWrt](https://github.com/xluoke/ywb94-openwrt-ssr)，在网站的介绍中，支持两种运行模式：
【IP路由模式】
 - 源码提供一个china_ssr.txt，罗列的是国内IP网段，在这种模式下，落在国内IP网段的不走代理，其他IP网段（即国外网站）网段走代理

【GFW列表模式】
 - 源码提供gfw_list.conf，在GFW列表中的网站走代理；其他都不走代理；

在上一篇博客 [DIY: 基于OpenWRT和小米路由器的透明代理(SSR)](https://chou-o-ning.github.io/blog/misc/2019/03/05/build-openwrt-ssr-on-Xiaomi-Mi-Router-3G-box.html) 中只介绍了IP路由模式，因此这里再附上我编译的GFW列表模式的编译包，我自己编译了ShadowSocksR，如果不想自己编译，可以直接在这里下载这个文件 [luci-app-shadowsocksR-GFW_1.2.1_all.ipk]({{ site.url }}/blog/assets/luci-app-shadowsocksR-GFW_1.2.1_all.ipk) 。这个版本，我是用OpenWrt v18.06.2版本编译的，使用的是以下编译选项：  
Target System: MediaTek Ralink MIPS  
Subtarget: MT7621 based boards  
Target Profile: Xiaomi Mi Router 3G  
其他架构的单板请不要使用这个文件安装。

安装方法如下：
opkg update
luci-app-shadowsocksR-GFW_1.2.1_all.ipk 组件会安装dnsmasq-full，和dnsmasq冲突，因此需要先将dnsmasq组件删除
opkg remove dnsmasq
opkg install luci-app-shadowsocksR-GFW_1.2.1_all.ipk
下载 https://cokebar.github.io/gfwlist2dnsmasq/dnsmasq_gfwlist_ipset.conf 到 /etc目录下
编辑 /etc/dnsmasq.conf文件，在文件的在最后一行添加
conf-file=/etc/dnsmasq_gfwlist_ipset.conf
重启dnsmasq-full服务即可
/etc/init.d/dnsmasq restart

在使用过程中，经常会遇到有些网站被墙而该网站又没有列在gfw_list.conf文件中，如果自己维护gfw_list.conf，一旦该文件被更新，则自己维护的列表信息就会丢失。因此可以参考我的博客文章 [DIY: 基于OpenWRT和小米路由器的透明代理之后续（自动更新gfwlist）](http://blog.chinaunix.net/uid-26598889-id-5791265.html) ，做一个自己维护的文件，然后使用脚本将自己的文件和gfw_list.conf文件进行拼接，以此来解决了这个问题。   


另外注意：OpenWrt V18.06.2版本，mt76芯片组的的2.4GWiFi有缺陷，请谨慎使用。（5G频段是OK的）（2019/11/25更新：V18.06.5版本，这个WiFi缺陷已经修复。）

因为这个博客使用Jekyll，暂时无法支持评论，因此如果有技术问题，可以发送到我的邮箱一起探讨，邮箱地址为chou dot o dot ning at gmail dot com  

