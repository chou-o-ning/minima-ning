---
layout: post
title: "chisel 在电信网络中被阻断问题解决"
categories: misc
---

client 端（公司网络），然而却可以连接家中 NAT 后的 client 端（家用网络）。在客户端抓包如下：
![抓包图](/blog/assets/client_cap.png)

从抓包看，tcp 一开始的连接是成功的，然而一旦进行 websocket 连接（http get 报文），连接马上被 reset 掉了。看起来有点像被防火墙阻断。

查看 Portainer 的源代码，websocket 连接是用 chisel 包开发的。查看 chisel，发现其是一个搭建隧道的工具，猜想该包被电信防火墙识别并阻断。被识别的特征码可能是上面图中的“chisel-v3”。

尝试修改 chisel 的源代码，并修改“chisel-v3”为一个随机字符串后，Portainer 运行正常。


因为这个博客使用Jekyll，暂时无法支持评论，因此如果有技术问题，可以发送到我的邮箱一起探讨，邮箱地址为chou dot o dot ning at gmail dot com  

