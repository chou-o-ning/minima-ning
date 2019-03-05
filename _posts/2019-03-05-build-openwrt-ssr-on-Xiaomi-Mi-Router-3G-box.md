---
layout: post
title: "DIY: 基于OpenWRT和小米路由器的透明代理(SSR)"
categories: misc
---

### 一、前言

之前我在China Unix的博客 [DIY: 基于OpenWRT和小米路由器的透明代理](http://blog.chinaunix.net/uid-26598889-id-5791053.html) 中写了如何在家改造一台无线路由器，直接支持ShadowSocks，被GFW收录的网站走代理，其他网站不走代理。这篇是其姊妹篇，两者唯一的区别就是使用ShadowSocks和ShadowSocksR.  
另外注意：OpenWrt V18.06版本，mt76芯片组的的2.4GWiFi有缺陷，请谨慎使用。（5G频段是OK的）  
因为这个博客使用Jekyll，暂时无法支持评论，因此如果有技术问题，可以发送到我的邮箱一起探讨，邮箱地址为chou dot o dot ning at gmail dot com  

### 二、缘起

作为一个程序员，工作无法离开Google，因此一直使用ShadowSocks/ShadowSocksR上网。使用PC时，设置浏览器的socks5代理上网，使用手机时，使用ShadowSocks/ShadowSocksR代理的APP软件。最近Google推出了Android9版本，而我的Google Pixel2手机无法从Android8升级（即使手机开了代理也不行）。因此打算在家改造一台无线路由器，直接支持ShadowSocks/ShadowSocksR，被GFW收录的网站走代理，其他网站不走代理。这样家里的手机、PC、pad都不用再设置各种代理了。

本来以为在网上查找一些文章，按照其步骤指导，会很容易地打造一个基于OpenWRT和ShadowSocks/ShadowSocksR的无线路由器上网方案。但依次做下来，发现有很多坑！大多数的文章只告诉你怎么做，但并不告诉你为什么这么做，导致出现问题的时候，只能自己查找资料定位问题。因此，在踏过这些坑之后，打算把整个过程记录下来，为后来者提供一些参考。

### 三、准备工作

1、如果你是Windows用户，请安装putty、WinSCP或其他类似软件，主要用于ssh客户端登录路由器和sftp上载文件。如果你有Mac或者Ubuntu等linux主机，系统自带ssh和sftp客户端。

2、需要一个U盘，并且格式化为Fat32格式

3、最好有一些linux的基础知识（因为OpenWRT是基于linux开发的）

### 四、硬件的选择

第一步，需要选择一款支持OpenWRT的无线路由器，并不是市面上的每款无线路由器都被OpenWRT支持，因此需要在 https://openwrt.org/toh/start 中查找相关的硬件。

我这里推荐小米路由器3G（MiWiFi 3G），理由如下（主要是其性价比）：

1、OpenWRT主干版本和最新版本支持该硬件，当前最新版本是V18.06.2，V18.06是OpenWRT和LEDE合并后的第一个版本；

2、淘宝上购买包邮价格为199元，在同类性能的路由器里面算是超级便宜的；

3、一个Wan口和两个Lan上行口为千兆，可以匹配超过百兆的家庭光宽带；

4、cpu为MediaTek的MT7621AT，mips双核880MHz，每个核支持双线程；

5、Flash为128MB，这个比较重要，因为OpenWRT支持很多模块的安装。很多低端的路由器因为降成本，Flash配置很小，安装几个模块之后，Flash就满了；

6、内存为256MB，足够跑绝大多数的应用；

7、带USB3.0口，可以挂接移动硬盘或摄像头，还可以玩一些家庭NAS、监控等一些好玩的应用；

8、Wi-Fi支持2.4G和5G双频，MediaTek的无线驱动是开源的，因此开源社区支持力度较好（玩OpenWRT很多人不建议用Broadcom的芯片，主要原因就是其无线部分是闭源的）。注意：OpenWrt V18.06版本，mt76芯片组的的2.4GWiFi有缺陷，请谨慎使用。

9、PCB版留有UART口，可供在此方面有需求的开发人员使用（对大多数人来说，串口可以不需要，有ssh远程连接就够用了）。当然，如果变砖，可以通过UART来拯救（Bootloader没有被破坏的话）。

### 五、路由器烧录OpenWRT firmware

下面是烧录的步骤：

1、拆包装；

2、按照其用户手册，安装小米WiFi APP，注册账号，并绑定路由器后，可在APP中查看和配置路由器

3、到 http://www.miwifi.com/miwifi_download.html 中，寻找到ROM版本：ROM for R3G 开发版，下载并将路由器更新到该版本。升级时，会提示是否恢复缺省配置，这里不需要恢复缺省配置，否则APP还需要重新绑定和配置一次；

4、登陆到 https://d.miwifi.com/rom/ssh ，用之前注册的账号登录，获取其root密码。注意：如果你有多台小米路由器，每台的root密码是不同的。然后下载工具包miwifi_ssh.bin；

5、到 http://downloads.openwrt.org/releases/18.06.2/targets/ramips/mt7621/ 寻找到 mir3g-squashfs-kernel1.bin 和 mir3g-squashfs-rootfs0.bin，下载之，文件名带有前缀openwrt-18.06.2-ramips-mt7621-）。这里需要说明一下，kernel是linux的内核文件，rootfs是文件系统，嵌入式Linux的Flash一般会有多个分区，kernel和rootfs分别放在两个分区，因此需要分别下载；

6、将下载的三个文件（miwifi_ssh.bin、openwrt-18.06.2-ramips-mt7621-mir3g-squashfs-kernel1.bin、openwrt-18.06.2-ramips-mt7621-mir3g-squashfs-rootfs0.bin）拷贝到格式化为Fat32的U盘；

7、路由器下电，将U盘插入路由器的USB口，按住reset按钮（用回形针或缝衣针），然后路由器上电。等到路由器黄灯开始闪烁时，释放reset按钮。等路由器重启后，使用ssh访问路由器，用root用户登录（路由器的ip地址为192.168.31.1），密码为步骤4的密码（Mac或Ubuntu用户登录命令为ssh root@192.168.31.1)。注意：这个步骤后，路由器就失去了保修。

8、在ssh命令行下  
cd /extdisks/sda1 (如果你拔出再重插回U盘，这个路径可能会有变化)  
mtd write openwrt-18.06.2-ramips-mt7621-mir3g-squashfs-kernel1.bin kernel1  
mtd write openwrt-18.06.2-ramips-mt7621-mir3g-squashfs-rootfs0.bin rootfs0  
nvram set flag_last_success=1  
nvram commit  
reboot  

9、重启后，再次使用ssh登录（这时ip地址变更为192.168.1.1，用户名还是root，无密码），这时可以看到OpenWRT的命令行界面了。

这里解释一下上面步骤的原理：

A、小米路由器的版本缺省是没有ssh组件的，小米公司对DIY用户还是非常友好的，提供了带ssh server组件的版本。

B、通过一种特定的操作方式（插入有miwifi_ssh.bin文件的U盘，长按reset重启），将ssh功能启用，这样用户登陆后就可以做任何事情，但同时也失去了保修。

C、登陆后，如果敲入下面的命令

root@XiaoQiang:~# cat /proc/mtd  
dev:    size   erasesize  name  
mtd0: 07f80000 00020000 "ALL"  
mtd1: 00080000 00020000 "Bootloader"  
mtd2: 00040000 00020000 "Config"  
mtd3: 00040000 00020000 "Bdata"  
mtd4: 00040000 00020000 "Factory"  
mtd5: 00040000 00020000 "crash"  
mtd6: 00040000 00020000 "crash_syslog"  
mtd7: 00040000 00020000 "reserved0"  
mtd8: 00400000 00020000 "kernel0"  
mtd9: 00400000 00020000 "kernel1"  
mtd10: 02000000 00020000 "rootfs0"  
mtd11: 02000000 00020000 "rootfs1"  
mtd12: 03580000 00020000 "overlay"  
mtd13: 012a6000 0001f000 "ubi_rootfs"  
mtd14: 030ec000 0001f000 "data"

可以看到flash的分区，这里看一下几个重要的分区：

A、Bootloader是启动分区，上电第一条指令从这里运行，然后启动linux kernel，再挂载rootfs文件系统。如果这个分区被破坏，会导致路由器变砖

B、kernel0/kernel1以及rootfs0/rootfs1是linxu内核和文件系统，双份，如果一份出错（比如路由器升级过程中断电），有备份，可以防止路由器变砖。

C、小米路由器也基于OpenWRT开发的，因此无须修改bootloader的配置，只要将kernel和rootfs刷新即可。

10、回头看上面的刷文件的命令

mtd write openwrt-18.06.1-ramips-mt7621-mir3g-squashfs-kernel1.bin kernel1 （更新kernel1)   
mtd write openwrt-18.06.1-ramips-mt7621-mir3g-squashfs-rootfs0.bin rootfs0。（更新rootfs0）  
nvram set flag_last_success=1（这个命令我不是很确定，可能是让bootloader从kernel1引导，即引导OpenWRT的firmware而不是小米的firmware）  
nvram commit （保存配置）  
reboot（重启）  

### 六、安装ShadowSocksR

我自己编译了ShadowSocksR，如果不想自己编译，可以直接在这里下载这个文件 [luci-app-shadowsocksR_1.2.1_all.ipk]({{ site.url }}/blog/assets/luci-app-shadowsocksR_1.2.1_all.ipk) 。这个版本，我是用OpenWrt v18.06.1版本编译的，使用的是以下编译选项：  
Target System: MediaTek Ralink MIPS  
Subtarget: MT7621 based boards  
Target Profile: Xiaomi Mi Router 3G  
其他架构的单板请不要使用这个文件安装。

如果需要自己编译，ShadowsSocksR的源代码在这里，请[参考](https://github.com/ywb94/openwrt-ssr)。

为了实现从pc到路由器的上载，需要安装openssh-sftp-server  
root@OpenWrt:~# opkg update  
root@OpenWrt:~# opkg install openssh-sftp-server  
使用sftp上载该ipk文件（Windows使用WinSCP命令，Mac或者Ubuntu环境使用命令 sftp root@192.168.1.1），完成后，安装ShadowSocksR  
root@OpenWrt:~# opkg install luci-app-shadowsocksR_1.2.1_all.ipk  

安装完毕，登录网页显示：
![初始页面](/blog/assets/pic01.jpg)  

点击编辑后，配置SSR服务器的参数  
![编辑服务器参数](/blog/assets/pic02.jpg)  

然后配置客户端的参数  
![配置客户端参数](/blog/assets/pic03.jpg)  

重启路由器即可（这个版本有些小bug，路由器不重启不生效）。

### 七、测试

打开路由器的SSR的状态页面，点击连接测试，如果连接测试都OK，即可使用。  

![状态页面](/blog/assets/pic04.jpg)

### 八、其他
我编译版本的路由策略是：  
国内网站：直连  
国外网张：代理  
源代码还支持GFW的白名单模式，即在白名单中代理，其他直连。我没有试过，可以[参考](https://github.com/ywb94/openwrt-ssr)。  

### 九、一些调试手段

1、如果错误地保存了一些配置导致pc无法通过ssh连接路由器（比如我曾经配置了一条错误的iptables命令导致pc无法连接路由器），需要用针按住reset 5秒以上，看到灯出现闪烁时放开，这时路由器会恢复缺省配置；  
2、使用logread命令查看日志，有些错误日志对问题定位很有帮助；  
3、有些问题可能需要通过源代码才能定位，其实对于程序员来说并不算复杂，设置开发环境、下载源码编译，调试、修改。这里就不展开了。
