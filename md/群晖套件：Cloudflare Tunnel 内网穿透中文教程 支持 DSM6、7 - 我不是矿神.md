> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [imnks.com](https://imnks.com/5984.html#google_vignette)

> Cloudflare Tunnel 提供一个安全便捷，跨...

Cloudflare Tunnel 提供一个安全便捷，跨网络的轻量级连接器，可以让我们在本机服务和 Cloudflare 之间创建安全的，仅出站连接。使用这种方式，用户无需配置防火墙，无需监听对外的 IP 地址，只要能访问公网就是能 CF 发布服务，把本机服务代理发布到公网。

Cloudflare Tunnel 源于 CF 的 Argo Smart Routing（Cloudflare 的流量加速功能）部分功能，之前叫 Argo Tunnel，是收费功能，基于带宽流量收费。最近该产品被分离出来免费开放并重命名为 Cloudflare Tunnel。

##### Cloudflare 网站：[https://www.cloudflare.com/zh-cn/](https://imnks.com/go/aHR0cHM6Ly93d3cuY2xvdWRmbGFyZS5jb20vemgtY24v)

###### 项目开源地址：[https://github.com/cloudflare/cloudflared](https://github.com/cloudflare/cloudflared)

矿神群晖 SPK 套件中心 提供各类国内常用的 DSM6、DSM7 套件，目前上架 DSM7 套件：Aria2、ffmpeg、Jellyfin、qBittorrent、Syncthing、Transmission 等等，持续更....

![](https://i.imnks.com/2022/07/3875613092.png!I)

3875613092.png

1、注册 Cloudflare 账号，添加自有的域名
--------------------------

![](https://i.imnks.com/2022/07/1127440168.png!I)

1127440168.png

使用 Free 免费计划

![](https://i.imnks.com/2022/07/3600975949.png!I)

3600975949.png

直接点击继续

![](https://i.imnks.com/2022/07/3263146300.png!I)

3263146300.png

修改域名的 DNS 记录

![](https://i.imnks.com/2022/07/1030840785.png!I)

1030840785.png

![](https://i.imnks.com/2022/07/3149883090.png!I)

3149883090.png

2、获取 Tunnel 的 token
-------------------

等待域名 DNS 解析生效后（约 10-30 分钟，多刷新几次）

![](https://i.imnks.com/2022/07/3920276627.png!I)

3920276627.png

访问 Zero Trust，创建一个隧道

![](https://i.imnks.com/2022/07/2793552961.png!I)

2793552961.png

![](https://i.imnks.com/2022/07/2795202256.png!I)

2795202256.png

点击 Configure，获取 token！！！

![](https://imnks.com/usr/themes/spimes/images/piex.gif)

4214735426.png

Public Hostname 里面添加一个隧道，域名前缀自定义，后期也可以在此直接修改添加！

支持添加 HTTP、HTTPS、TCP（以及 SSH、RDP 等待），群晖常用就 HTTP（网页）TCP（APP）

![](https://imnks.com/usr/themes/spimes/images/piex.gif)

809891328.png

![](https://imnks.com/usr/themes/spimes/images/piex.gif)

3460825405.png

安装 Cloudflare
-------------

填入上面获取的 token 即可

![](https://imnks.com/usr/themes/spimes/images/piex.gif)

1912044242.png

访问测试，速度有点慢啊。。。
--------------

![](https://imnks.com/usr/themes/spimes/images/piex.gif)

3813981892.png

![](https://imnks.com/usr/themes/spimes/images/piex.gif)

2856683432.png

原创文章，作者：我不是矿神，本文章内容未经书面许可禁止一切形式的转载：https://imnks.com/5984.html