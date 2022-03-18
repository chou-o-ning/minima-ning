---
layout: post
title: "Ubuntu 20.04 Samba 安全性提升引发访问的变化"
categories: misc
---

之前我一直使用的是 Ubuntu 18.04，因为支持时间快到了（LTS 为5年，23年4月到期），最近在家中安装的版本都是 Ubuntu 20.04。

安装和配置 Samba 的方式结束后，访问服务器 smb://gserver 发现出现下面的问题:

出现下面的错误提示为：无法访问位置 从服务器获取共享列表失败：无效的参数
![错误提示](/blog/assets/samba_1.png)

但是使用同样的方式访问 Ubuntu 18.04 的 Samba 是好的。

如果访问的命令修改为 smb://gserver/share ，屏幕跳出输入用户和密码的弹窗，访问正常。

因此可以看出 Ubuntu 20.04 的 Samba 安全性有了提升，无法直接访问该服务器，而必须输入访问的共享目录。

另外需要注意，按照 Samba 的配置（见下图）：共享目录名为 smb://gserer/share 而不是 smb://gserver/home/ning/project

![Samba配置](/blog/assets/samba_2.png)

因为这个博客使用Jekyll，暂时无法支持评论，因此如果有技术问题，可以发送到我的邮箱一起探讨，邮箱地址为chou dot o dot ning at gmail dot com  

