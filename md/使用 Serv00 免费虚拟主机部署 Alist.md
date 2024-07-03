> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/680607217)

> Serv00是一个提供免费的Virtual Host的平台，其托管平台使用的是FreeBSD系统，并不是Linux。每个账号有效期10年，超过三个月不登入Panel以及SSH则会被删除账号。其提供的服务大致如下表所示： 名称Serv00 免费提供…

[Serv00](https://link.zhihu.com/?target=https%3A//www.serv00.com/) 是一个提供**免费**的 Virtual Host 的平台，其托管平台使用的是 **FreeBSD** 系统，并**不是 Linux**。每个账号有效期 **10 年**，超过**三个月**不登入 Panel 以及 SSH 则会被删除账号。

其提供的服务大致如下表所示：

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th>名称</th><th>Serv00 免费提供</th></tr><tr><td>存储空间</td><td>3 GB</td></tr><tr><td>每月流量</td><td>unlimited</td></tr><tr><td>网站数量</td><td>100</td></tr><tr><td>MySQL</td><td>10</td></tr><tr><td>PostgreSQL</td><td>3</td></tr><tr><td>MongoDB</td><td>3</td></tr><tr><td>GIT/SVN/HG 仓库</td><td>3</td></tr><tr><td>TCP/UDP 端口</td><td>3</td></tr><tr><td>PHP 解释器</td><td>3</td></tr><tr><td>系统进程</td><td>15</td></tr><tr><td>RAM</td><td>512MB</td></tr><tr><td>备份</td><td>7 天</td></tr><tr><td>服务器放置</td><td>欧盟</td></tr><tr><td>免费子域名</td><td><a href="https://link.zhihu.com/?target=http%3A//login.serv00.net" target="_blank" rel="nofollow noreferrer" data-za-detail-view-id="1043">http://login.serv00.net</a></td></tr><tr><td>技术支持</td><td>只有论坛</td></tr><tr><td>SLA</td><td>不支持</td></tr><tr><td>现代技术</td><td>支持</td></tr><tr><td>SSH 访问</td><td>支持</td></tr><tr><td>SSH 隧道</td><td>不支持</td></tr><tr><td>远程数据库访问</td><td>不支持</td></tr><tr><td>固态硬盘</td><td>支持</td></tr><tr><td>没有广告</td><td>支持</td></tr><tr><td>价格</td><td>免费</td></tr></tbody></table>

> 更详细的配置说明请查阅官方说明。

Serv00 使用的面板是这种共享虚拟主机常用的 DevilWEB，还挺好上手的。下面详细介绍一下使用 Serv00 部署 Alist 的方法步骤：

> **觉得一步步手动操作太麻烦的，可以直接翻到最末，提供了我的 blog 地址，提供了一些简化操作的一键式命令。**

注册账号
----

首先去 serv00 的官网[注册一个账号](https://link.zhihu.com/?target=https%3A//www.serv00.com/offer/create_new_account)，最好**不要使用国内邮箱**，注册信息请尽可能**真实填写**。

![](https://pic1.zhimg.com/v2-0f3b0d1861e310f672a9f228de6d16cc_r.jpg)

接着你可以在邮箱里收到你的注册信息的邮件：

![](https://pic2.zhimg.com/v2-29b69c403e9bb492ea1e75dad8cc6aa5_r.jpg)

邮件最末有 Panel 的入口地址和[文档链接](https://link.zhihu.com/?target=https%3A//docs.serv00.com/)以及[论坛链接](https://link.zhihu.com/?target=https%3A//forum.serv00.com/)，接下来登入 Panel 进行操作：

![](https://pic4.zhimg.com/v2-a3ae7c972a622d7ab3b137f7f0eb7243_r.jpg)

首先去左侧的 **Additional Service** 选项卡中，找到 **Run your own applications** 选项，将其设置为允许：

![](https://pic1.zhimg.com/v2-494fc0693210873b8a122e3e33403be4_r.jpg)

接着去 **Port reservation** 选项卡，使用 **Add port** 功能，随机添加一个 **TCP 端口**：

![](https://pic1.zhimg.com/v2-8b0cca6d0ff3e209e7b1780205cc6b50_r.jpg)

记下你添加的端口，后面要用。

接着使用 SSH 登入到你的账户，我使用的 SSH 客户端是 [Termius](https://link.zhihu.com/?target=https%3A//termius.com/)：

![](https://pic1.zhimg.com/v2-da4ba766ecd2b3a96aa7d5d5b9d6cb28_r.jpg)

> 注意，这一步可能你的网络不一定连得上，可能需要使用打倒美帝的上网手段。

部署 Alist
--------

[Alist 的官方仓库](https://link.zhihu.com/?target=https%3A//github.com/alist-org/alist)并没有提供 FreeBSD 版本的可执行文件构筑，但是我在 Github 上找到了 Unofficial 的，专门为 Alist 构筑 FreeBSD 版本的[仓库](https://link.zhihu.com/?target=https%3A//github.com/uubulb/alist-freebsd)，所以只要使用这个仓库就可以在 Serv00 上部署使用 Alist 了。

Serv00 本身提供的网站托管在`~/domains`路径下，所以我建议把 Alist 也部署到这个路径下的子目录：

```
mkdir -p ~/domains/alist && cd ~/domains/alist
```

接着下载目前`uubulb/alist-freebsd`提供的最新版的 Alist 的可执行二进制文件构筑：

```
wget https://github.com/uubulb/alist-freebsd/releases/download/v3.30.0/alist && chmod +x alist
```

然后需要先启动一次 Alist 以生成配置文件，此次启动一定会失败，请不用在意：

```
./alist server
```

接着回到 Panel 中，找到 MySQL 选项卡，使用 Add database 功能新建一个数据库：

![](https://pic2.zhimg.com/v2-f6b1c1086558dab0fdcb382180438599_r.jpg)

> 密码要求含有大写字母、小写字母和数字三种字符，且长度必须超过 6 个字符。

接下来进入 File manager 选项卡，进入`~/domains/alist/data`路径，可以看到一个名为`config.json`的文件，右键点击，选择 View/Edit > Source Editor，进行编辑：

![](https://pic3.zhimg.com/v2-428d35264f44fe6b12152658813114d6_r.jpg)

我主要修改了 CDN、database、scheme 三个部分，其中 CDN 可以在 [Alist 的官方文档](https://link.zhihu.com/?target=https%3A//alist.nn.ci/zh/config/configuration.html%23cdn)找到，请选择你本地网络连接速度最快的一个。

scheme 部分，我选择修改 adress 为`127.0.0.1`本地回环，是为了避免被他人使用`http://ip:port` 的方式进行访问。至于自己怎么访问，我在本文后面的部分会进行介绍。port 要改成自己前面放行的端口。

database 部分，type 需要改成 `mysql`，host 填写你在注册邮件中看到的 mysql 的地址，port 是默认的 3306，用户名、密码、数据库名则按照你创建的情况进行填写。

改完之后，点击 save 保存，接着回到 SSH 窗口中进行操作。

先启动一次，查看运行是否正常：

```
./alist server
```

![](https://pic3.zhimg.com/v2-722472003c13bc70e85ad58d7902038a_r.jpg)

运行正常，记得把管理员用户的密码记住。接着使用`Ctrl+c`停止运行。

绑定域名
----

此时还没有访问 Alist 的方法，因为监听的地址是本地回环，所以需要将其反向代理出来。我选择使用 Cloudflare 提供的 Argo 通道，顺带给 Alist 绑定自己的域名。

Cloudflared 官方仓库没有提供 FreeBSD 平台的客户端，但是和 Alist 一样的，我找到了 Unofficial 的 FreeBSD 版本的[构筑](https://link.zhihu.com/?target=https%3A//cloudflared.bowring.uk/)，接下来使用它打隧道：

新建并进入 Cloudflared 的工作目录：

```
mkdir -p ~/domains/cloudflared && cd ~/domains/cloudflared
```

下载 Cloudflared：

```
wget https://cloudflared.bowring.uk/binaries/cloudflared-freebsd-2023.10.0.7z && 7z x cloudflared-freebsd-2023.10.0.7z && rm cloudflared-freebsd-2023.10.0.7z && mv -f ./temp/cloudflared-freebsd-2023.10.0 ./cloudflared && rm -rf temp
```

然后在 [Cloudflare 的面板](https://link.zhihu.com/?target=https%3A//one.dash.cloudflare.com/)中，找到 Networks 分类下的 Tunnels 功能，点击 Create a tunnel，选择 Cloudflared，Next，随便取个名字，Next，往下翻，可以看到 Run the following command，然后给了一串命令，将其复制出来，大概是这样的：

```
cloudflared.exe service install eyJhIjoiNzh...............V5TWpBeSJ9
```

前面的不需要管，只需要保留最后 ey 开头的那串很长的 TOKEN，去 SSH 中测试运行 Cloudflared：

```
./cloudflared tunnel --edge-ip-version auto --protocol http2 --heartbeat-interval 10s run --token eyJhIjoiNzh...............V5TWpBeSJ9
```

> 记得把最后的那一串替换成你的 TOKEN。

接着回到 Cloudflare 的面板，继续点击 Next，然后添加一个自己的域名，Service 中，Type 选择 HTTP，URL 填写 localhost:PORT，其中 PORT 为你的 Alist 对应的端口。点击 Save Tunnel 后，可以看到自己新建的 Tunnel 上线。

接着使用`Ctrl+c`停止运行。然后安装进程管理工具 pm2：

```
bash <(curl -s https://raw.githubusercontent.com/k0baya/alist_repl/main/serv00/install-pm2.sh)
```

然后使用 pm2 启动 Cloudflared：

```
~/.npm-global/bin/pm2 start ./cloudflared -- tunnel --edge-ip-version auto --protocol http2 --heartbeat-interval 10s run --token eyJhIjoiNzh...............V5TWpBeSJ9
```

> 记得把最后的那一串替换成你的 TOKEN。

再启动 Alist：

```
cd ~/domains/alist && ~/.npm-global/bin/pm2 start ./alist -- server
```

到这里，就可以直接通过你的域名访问刚刚部署的 Alist 了。

收尾工作
----

听说 Serv00 会不定时重启机器，所以我们把 pm2 添加开机自启，可以保证每次重启都能由 pm2 调动 Alist 和 Cloudflared。而且 Serv00 每三个月内必须要有一次登录面板或者 SSH 连接，不然会删号，也可以通过一个脚本解决问题，接下来我会详细说明。

### 自动定时 SSH

在 Panel 中找到 File manager 选项卡，进入 alist 所在的路径，然后找到上方 Send 按钮左边的 +，选择 New empty file，文件名命名为`auto-renew.sh`， 右键点击`auto-renew.sh`，选择 View/Edit > Source Editor，进行编辑，把下面的代码块的内容都复制进去：

```
#!/bin/bash

while true; do
  sshpass -p '密码' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -tt 用户名@地址 "exit" &
  sleep 259200  #30天为259200秒
done
```

> 记得把其中的密码、用户名、ssh 的地址修改为你自己的。

保存后回到 SSH 中，进入`auto-renew.sh` 所在的路径，并使用 pm2 管理运行它：

```
cd  ~/domains/alist && chmod +x auto-renew.sh && ~/.npm-global/bin/pm2 start ./auto-renew.sh
```

这样就会每隔一个月自动执行一次 SSH 连接，自己 SSH 自己进行续期。

### 添加开机自启

在 Panel 中找到 Cron jobs 选项卡，使用 Add cron job 功能添加任务：

![](https://pic2.zhimg.com/v2-8aef51a10357b6c41bab39cee776a0bd_r.jpg)

Specify time 选择 After reboot，即为重启后运行。Form type 选择 Advanced，Command 写：

```
/home/你的用户名/.npm-global/bin/pm2 resurrect
```

> 记得把你的用户名改为你的用户名

添加完之后，在 SSH 窗口保存 pm2 的当前任务列表快照：

```
~/.npm-global/bin/pm2 save
```

这样每次 serv00 不定时重启任务时，都能自动调用 pm2 读取保存的任务列表快照，恢复任务列表。

以上就是 Serv00 搭建使用 Alist 的全部过程。如果你觉得这样一步一步手动部署太繁琐，你可以参照我的 blog，有更轻松的部署办法：[Saika's Blog](https://link.zhihu.com/?target=https%3A//blog.rappit.site/)