> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.rappit.site](https://blog.rappit.site/2024/01/27/serv00_logs/)

> A Reading focusing Blog

[这个平台](https://www.serv00.com/)是个 Virtual Host ，没有 Root ，还是 FreeBSD 的系统，不是 Linux ，不太好用。但是优点是隔离性差， Memory 和 vCPU 能短时间内超过 100% 进行调用。

已经玩了不少时间了，起初看到 Github 上有使用 Serv00 搭建 Vless 节点的[仓库](https://github.com/qwer-search/serv00-vless)，就上手玩了一下，后来发现极其不稳， screen 运行的进程总是过一段时间就掉了（后经网友讨论确认为 Serv00 有时候会重启宿主机），又得 ssh 上去启动，相当不友好，且后来又发现了 Hax 这样的玩具，就对 Serv00 视如敝履了。

但是这两天有[群友](https://jq.qq.com/?_wv=1027&k=qssjFvAs)突然提醒我才想起，我在 Hax 上用的很舒服的 pm2 也可以在 Serv00 上使用，这个十年有效期的玩具突然显得有用了起来。

经过我的尝试，成功在 Serv00 上部署了一些服务，接下来进行记录：

[](#面板自带功能 "面板自带功能")面板自带功能
==========================

[](#域名 "域名")域名
--------------

Serv00 上如果想要使用自己的域名，有两种方式，一种是通过 Cloudflare 提供的 Argo 隧道，不仅能绑域名，免配置 ssl ，还可以享受 Cloudflare 的免费 CDN 提速。第二种就是直接使用面板内自带的 DNS 服务器功能绑定自己的域名。

在 Panel 中进入 DNS zones 选项卡，使用 Add new zone 功能添加自己的域名或者自己的域名的子域，然后在 Zone list 中找到刚刚添加的域名，点击 Edit 查看 DNS 记录，把当中列出的全部记录添加到自己的域名的 DNS 记录中即可完成域名的绑定。

Serv00 本身对于绑定在其上的域名提供了许多的服务支持，这里所说的绑定在 Serv00 上的域名包括自己绑定的自己的域名，以及 Serv00 在注册账户时赠送的域名 `USERNAME.serv00.net` ，其服务包括免费的一键申请式的 SSL 证书、域名邮箱、 DNS 管理等多种功能。

### [](#SSL证书申请 "SSL证书申请")SSL 证书申请

在 Panel 中进入 WWW websites 选项卡，点击 Manage SSL certificates ，在你需要申请 SSL 证书的域名的 A 记录指向的那个 IP 地址右侧点击 Manage ，再点击 Add certificate ， Type 选择 Generate Let’s Encrypt certificate ，Domain 选择要申请 SSL 证书的域名，再点击 Add 即可。

### [](#域名邮箱 "域名邮箱")域名邮箱

Panel 中进入 E-mail 选项卡，注册账号后会自动注册一个域名邮箱，用户名是 `USERNAME@USERNAME.serv00.net` 是 Serv00 的账户密码。可以使用 Add new e-mail 功能新建邮箱账户。

也可以在 Add new alias 功能中新建别名邮箱，其别名邮箱功能也提供了和 Cloudflare 一样的 Catch-all 的 Advanced settings 选项，用来批量注册东西十分方便。

目前我的测试中，似乎没有在 Manage whitelist 中添加进白名单的域名邮箱发来的邮件全部都会被识别为垃圾邮件。所以有需要的话可以在 Manage whitlist 中添加你需要接受邮件的邮箱的域名，比如 `qq.com` 、 `gmail.com` 等等。

如果绑定了自己的域名，想要使用自己的域名配置域名邮箱的话，要在 Domain list 中找到自己的域名，点击最右边的 DKIM ， action 选择 Add DNS record automatically ，然后 Sign domain 以注册域名，使得新的域名邮箱能够通过一些邮件接收服务器的验证。

Open web client 功能就可以进入邮箱的登录页面了，其使用方法与大多数的邮箱相同，不再赘述。

### [](#DNS管理 "DNS管理")DNS 管理

DNS zones 选项卡中在自己绑定的域名右侧点击 Edit ，即可查看当前域名的所有 DNS 记录，在 Add new record 中可以手动添加新的 DNS 记录，与大多数的域名服务商提供的 DNS 管理的功能类似。

### [](#Proxy "Proxy")Proxy

WWW websites 选项卡中可以根据语言不同添加多种网站，其中 PHP 的 `eval() function` 和 `exec() function` 都要在添加完网站后，在 Manage > Details 中打开。不同类型的网页其 Details 中的选项也都有差异，可以按需查看配置，这里重点讲一下 Proxy 类型指向自己的应用程序监听端口的配置。

Add new website 功能中， Domain 填写自己的域名或者 serv00 分配的域名，或者它们的子域，展开 Advanced settings， Website type 选择 Proxy ，Proxy target 选择 localhost ， Proxy port 选择自己的应用监听的端口，其他选项留空或者保持默认，点击 Add 即可。接下来就能使用刚刚填写的域名访问自己部署的对应端口的应用了。如果需要 https 访问，再按前文的步骤去申请 SSL 证书即可。

[](#运行自己的应用 "运行自己的应用")运行自己的应用
-----------------------------

Additional services 选项卡中找到 Run your own applications 项目，将其设置为 Enabled 即可。**如果不开启这一项，自己的用户目录下的所有文件都无法添加可执行权限。**

[](#File-manager "File manager")File manager
--------------------------------------------

文件管理，有一定的在线编辑和预览的功能，兼具文件的上传下载，删除新建等各种管理功能，十分便利。

[](#Port-reservation "Port reservation")Port reservation
--------------------------------------------------------

需要使用端口都得在这申请。

[](#数据库 "数据库")数据库
-----------------

Serv00 提供了 MySQL 、 PostgreSQL 、 MongoDB 三种数据库，可以按需新建数据库、数据库用户。同时， Serv00 还提供了三种数据库的 webui ，十分便利。

需要注意的是，所有数据库在新建时，其用户名和数据库名都有一个 `mxxx_` 的前缀，在使用时容易被忽视。

[](#Cron-jobs "Cron jobs")Cron jobs
-----------------------------------

Cron jobs 选项卡提供了一些计划性任务的设置功能，在这里可以设置开机自启任务，或者定时循环任务，当然常用的还是开机自启任务的设定， Specify time 选择 After reboot 即为开机自启。

[](#部署应用前的一些准备工作 "部署应用前的一些准备工作")部署应用前的一些准备工作
============================================

在部署自己的应用之前，我建议提前安装好 pm2 以及 Cloudflared （可选）。前者是进程管理工具，用来方便开机自启，以及程序崩溃后自启，查阅进程运行情况等等。后者是 Cloudflare 的 Argo 隧道客户端，用它也可以给自己部署的应用加域名。特别是 Uptime Kuma ，更加推荐使用 Cloudflared 加域名，而不建议使用面板自带的 Proxy 。

[](#Pm2 "Pm2")Pm2
-----------------

这个是重中之重，如果不是成功安装了 pm2 ，我甚至不会尝试探索 Serv00 这个玩具有什么用，所以 pm2 的安装方法记录在开头。

在 SSH 连接 serv00 之后，直接使用一键脚本安装 pm2 ：

折叠代码块 BASH 复制代码

```
bash <(curl -s https://raw.githubusercontent.com/k0baya/alist_repl/main/serv00/install-pm2.sh)
```

> 如果安装完成后执行 `pm2` 提示命令未找到，你可以断开 SSH 连接，再重新连接，即可。

[](#Cloudflared "Cloudflared")Cloudflared
-----------------------------------------

Cloudflared 官方仓库并没有构建 FreeBSD 系统上能够使用的二进制文件，但是同样的，我找到了[第三方的构筑](https://cloudflared.bowring.uk/)。使用第三方构筑的二进制文件，就能愉快的使用隧道了。

关于 Cloudflared 是什么，有什么用，ARGO_TOKEN 如何获取等部分，这里不再赘述，详细可以查看我的关于 CodeSandbox 和 Hax 的文章。

创建并进入 Cloudflared 的工作目录：

折叠代码块 BASH 复制代码

```
mkdir -p ~/domains/cloudflared && cd ~/domains/cloudflared
```

下载 Cloudflared：

折叠代码块 BASH 复制代码

```
wget https://cloudflared.bowring.uk/binaries/cloudflared-freebsd-latest.7z && 7z x cloudflared-freebsd-latest.7z && rm cloudflared-freebsd-latest.7z && mv -f ./temp/* ./cloudflared && rm -rf temp
```

测试运行：

折叠代码块 BASH 复制代码

```
./cloudflared tunnel --edge-ip-version auto --protocol http2 --heartbeat-interval 10s run --token ARGO_TOKEN
```

> 其中 ARGO_TOKEN 要替换成自己的。确定运行没有问题后，按 `Ctrl+c`即可停止运行。

使用 pm2 启动 Cloudflared：

折叠代码块 BASH 复制代码

```
pm2 start ./cloudflared -- tunnel --edge-ip-version auto --protocol http2 --heartbeat-interval 10s run --token ARGO_TOKEN
```

> 其中 ARGO_TOKEN 要替换成自己的。

接着去 CLoudflare 的面板中设置域名对应端口，即可使用域名访问自己搭建的服务了。

[](#安装-go1-22 "安装 go1.22")安装 go1.22
-----------------------------------

> 如果你有安装自己使用 go build 构建的需求，你可以选择安装最新的 go1.22 ，这里记录其安装过程。

由于 Serv00 服务器上并未提供 go1.22 ，只提供了 go1.20.3 ，无法正常进行构建工作，所以需要手动安装 go1.22 环境。

折叠代码块 BASH 复制代码

```
# 创建安装目录
mkdir -p ~/local/soft && cd ~/local/soft
# 下载编译好的 go1.22 的程序包
wget https://dl.google.com/go/go1.22.0.freebsd-amd64.tar.gz
# 解压
tar -xzvf go1.22.0.freebsd-amd64.tar.gz
# 删除压缩文件
rm go1.22.0.freebsd-amd64.tar.gz
# 修改 .profile 文件
echo 'export PATH=~/local/soft/go/bin:$PATH' >> ~/.profile
# 使 .profile 的修改生效
source ~/.profile
# 检查 go 版本
go version
```

[](#部署自己的应用 "部署自己的应用")部署自己的应用
=============================

> 关于设定 PHP 版本、插件、参数等配置均可参考文档的 [.htaccess](https://docs.serv00.com/htaccess/) 部分进行配置，由于 PHP 的应用部署实在是太简单，故本文不会过多介绍。

[](#WordPress "WordPress")WordPress
-----------------------------------

实际上在 serv00 的[文档](https://docs.serv00.com/)中有搭建网站的示例，没错，示例就有 WordPress ，实际上 WordPress 确实可以搭建，十分简单好用。这里不做过多介绍，按照文档一步步操作即可。

除了 WordPress 外，文档中还详细介绍了 Redis、Memcached、Imapsync、WP-CLI、Tomcat 等服务的搭建方法，有需求的都可以照着抄。

[](#KodBox "KodBox")KodBox
--------------------------

虽然 Serv00 能够部署 KodBox，但是实在是不太好用。最直观的感受就是卡，因为 KodBox 运行期间需要调用多个 PHP 组件，而 Serv00 限制同时处理三个 PHP 进程，所以显得特别慢。其次， Serv00 没有 Root 权限，部分 PHP 插件没有安装，也无法安装，导致有一些 KodBox 的插件无法正常运行。

当然如果只是图新奇搭一个玩玩，也是可以的。下面是步骤：

首先在 Panel 中 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>PHP</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 安装KodBox
bash <(curl -s https://pan.rappit.site/d/shell/kodbox1.49/serv00-kodbox-install.sh)
```

然后去 Panel 中的 MySQL 选项卡，新建数据库和用户，用以接入 KodBox 。再去 WWW Websites 选项卡中找到 用户名. serv00.net ，点击右侧的 Manage > Details 进入设置，把 GZIP compression、Allow PHP eval() function、Allow PHP exec() function 三个功能打开。

然后使用浏览器访问你的 KodBxo 的域名，进行安装配置即可。初次启动需要较长的时间，请耐心等待。

[](#Lsky-Pro "Lsky-Pro")[Lsky-Pro](https://github.com/lsky-org/lsky-pro)
------------------------------------------------------------------------

一开始看[兰空图床的文档](https://docs.lsky.pro/)没看到 webdav 功能的相关介绍，只看到几个我都不用的存储介质，遂不感兴趣的搁置了，然而群友近日又提起，我打开 GitHub 才发现首页有个 Commit 的标题就是 webdav 相关，即兰空图床支持 webdav 。于是我便部署了一下，体验感觉还不错，简单易用。

本来无意在本篇文章再多写 PHP 相关的站点部署，因为过于简单。但是奈何群友有需求，遂做个简单的步骤记录：

首先在 Panel 中 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>PHP</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下：

折叠代码块 BASH 复制代码

```
# 下载图床应用
release_info=$(curl -s https://api.github.com/repos/lsky-org/lsky-pro/releases/latest)
asset_url=$(echo "$release_info" | jq -r '.assets[] | select(.name != "source code") | .browser_download_url')
curl -L -o temp.zip "$asset_url" && unzip -q temp.zip && rm -f temp.zip
rm -rf public_html && ln -s "$PWD/public" "$PWD/public_html"
```

接着在 Panel 中 WWW websites 选项卡内，点击自己刚刚创建的用于部署 Lsky-Pro 的域名的 Manage > Details ，在 **Open Basedir directories** 的最末添加：

折叠代码块 复制代码

```
:/usr/home/用户名/domains/xxx.USERNAME.serv00.net
```

> 记得把用户名和最末的域名换成自己的。

然后把 **GZIP compression** 、**Allow PHP eval() function** 、**Allow PHP exec() function** 都打开，点击 save changes 保存。

然后去 Panel 中的 MySQL 选项卡，新建数据库和用户，用以接入 Lsky-Pro 。

然后使用浏览器访问你的 Lsky-Pro 的域名，进行安装配置即可。

> 上面的应用不需要占用端口。

> 下面的应用每一个都能够 / 需要占用端口。

[](#Vless "Vless")Vless
-----------------------

这个肯定是第一时间部署的，每次遇到这样的平台，第一时间总是想着能不能搭建节点。

### [](#① "①")①

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Vless 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下：

折叠代码块 BASH 复制代码

```
# 克隆源仓库
rm -rf public_html && git clone https://github.com/qwer-search/serv00-vless public_html && cd public_html && rm -f README.md
```

使用 vim 编辑或者直接去 Panel 中的 File Manager 选项卡在线编辑 `app.js` 文件，修改端口为刚刚放行的端口。

安装依赖：

折叠代码块 BASH 复制代码

```
npm install
```

安装完毕后，使用 pm2 启动并守护 vless 进程：

折叠代码块 BASH 复制代码

```
pm2 start app.js --name vless
```

接着去你的代理客户端软件中手动添加 vless 配置即可：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>地址</td><td>Panel 中 WWW Websites 选项卡里的你的 Domain name</td></tr><tr><td>端口</td><td>你放行的端口</td></tr><tr><td>用户 id</td><td>37a0bd7c-8b9f-4693-8916-bd1e2da0a817</td></tr><tr><td>传输协议</td><td>ws</td></tr><tr><td>伪装域名</td><td>同地址</td></tr><tr><td>ws path</td><td>/</td></tr></tbody></table>

上表没有给出的可以不填。

### [](#② "②")②

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Vless 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下，再使用 `npm` 命令安装 `@3Kmfi6HP/nodejs-proxy` ：

折叠代码块 BASH 复制代码

```
npm install @3Kmfi6HP/nodejs-proxy
```

> 被删库了可以自己换个源安装，比如：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> npm --registry http://r.cnpmjs.org install @3Kmfi6HP/nodejs-proxy
> ```
> 
> 这个源如果也不行了请自己找其他源替换。

再使用 pm2 启动：

折叠代码块 BASH 复制代码

```
# 记得把 PORT 替换成自己放行的端口。
pm2 start npx --name vless -- nodejs-proxy -p PORT
```

接着访问这个刚刚添加的站点，即可在网页上直接获取配置。  
**哦对，有个小 `bug` ，端口需要改成 443 ，而网页中默认给的配置是 80 。**

> 之所以说这个 `npm` 包不安全，是因为其配置在网页上都可以看到，而且网页设计不太合理，有一个不带 `uuid` 的中转页面，所以可以使用 **fofa** 、 **shodan** 等网络空间扫描工具批量扫出来，而且不止 Serv00 一个平台有人使用，如果你感兴趣，你可以去搜搜看，可以收获一大批 Vless 节点。
> 
> 这里放一个 Serv00 上的，我在 fofa 上搜到的页面作为部署示例：[https://pclwgdwv.serv00.net/](https://pclwgdwv.serv00.net/)

[](#Alist "Alist")Alist
-----------------------

Alist 官方仓库没有构筑 FreeBSD 系统下能够运行的 Alist 可执行文件，但是我在 Github 上发现了一个使用 Github Workflow 自动构筑 FreeBSD 适用的 Alist 的[仓库](https://github.com/uubulb/alist-freebsd)，使用这个仓库就可以很便利的在 Serv00 上部署 Alist。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Alist 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 使用一键命令安装 Alist
wget -O alist-freebsd.sh https://raw.githubusercontent.com/k0baya/alist_repl/main/serv00/alist-freebsd.sh && sh alist-freebsd.sh
```

在 Panel 中进入 MySQL 选项卡，使用 Add database 功能新建一个数据库。

> 密码要求含有大写字母、小写字母和数字三种字符，且长度必须超过 6 个字符。

接下来进入 File manager 选项卡，进入 `~/domains/xxx.USERNAME.serv00.net/public_html/data` 路径，可以看到一个名为 `config.json` 的文件，右键点击，选择 View/Edit > Source Editor ，进行编辑：

我主要修改了 CDN、database、scheme 三个部分，其中 CDN 可以在 [Alist 的官方文档](https://link.zhihu.com/?target=https://alist.nn.ci/zh/config/configuration.html%23cdn)找到，请选择你本地网络连接速度最快的一个。

scheme 部分，我选择修改 adress 为 `127.0.0.1`本地回环，是为了避免被他人使用 `http://ip:port`的方式进行访问。至于自己怎么访问，我在本文后面的部分会进行介绍。port 要改成自己前面放行的端口。

database 部分，type 需要改成 `mysql` ，host 填写你在注册邮件中看到的 mysql 的地址， port 是默认的 3306，用户名、密码、数据库名则按照你创建的情况进行填写。

> 最新版本的 Alist 如果不想开启 S3 Server，请把对应的配置文件中的端口配置为 0 。

改完之后，点击 save 保存，接着回到 SSH 窗口中进行操作：

测试启动 Alist：

折叠代码块 BASH 复制代码

```
./alist server
```

> 确定运行没有问题后，按 `Ctrl+c`即可停止运行。

使用 pm2 启动并管理 alist：

折叠代码块 BASH 复制代码

```
pm2 start ./alist -- server
```

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#Synctv "Synctv")[Synctv](https://synctv.wiki/)
--------------------------------------------------

群友仿照 alist-freebsd 的仓库的 workflow 进行构筑的。部署简单，与 alist 类似。首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Synctv 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载可执行文件
release_info=$(curl -s https://api.github.com/repos/shangskr/synctv-freebsd/releases/latest)
asset_url=$(echo "$release_info" | jq -r '.assets[] | select(.name != "source code") | .browser_download_url')
curl -L -o synctv "$asset_url" && chmod +x synctv
```

新建启动脚本：

折叠代码块 BASH 复制代码

```
cat > start.sh << EOF
#!/bin/sh
# 如果不希望被使用 http://ip:port 的方式访问，取消注释下一行
# export SYNCTV_SERVER_LISTEN=127.0.0.1
# 把下一行的最末的PORT改成自己放行的端口
export SYNCTV_SERVER_PORT=PORT
exec ./synctv server --data-dir ./
EOF
```

添加可执行权限：

折叠代码块 BASH 复制代码

```
chmod +x start.sh
```

测试运行：

折叠代码块 BASH 复制代码

```
./start.sh
```

> 确定运行没有问题后，按 `Ctrl+c`即可停止运行。

使用 pm2 启动并管理：

折叠代码块 BASH 复制代码

```
pm2 start ./start.sh --name synctv
```

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#One-api "One-api")One-api
-----------------------------

~源仓库没有提供 freebsd 平台的二进制文件，需要自己构建，但是很简单~。我已经写了一个仓库用于自动化构建 freebsd 版本的 one-api 二进制文件，可以直接下载使用。

> 如果你想使用 New-API ，可以使用这个仓库 [k0baya/new-api-freebsd](https://github.com/k0baya/new-api-freebsd)，用法与本节介绍的 One-API 基本一致，对比 One-API 添加了一些更方便的功能。也许之后 One-API 也会加入这些功能。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 One-API 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载 one-api
release_info=$(curl -s https://api.github.com/repos/k0baya/one-api-freebsd/releases/latest)
asset_url=$(echo "$release_info" | jq -r '.assets[] | select(.name != "source code") | .browser_download_url')
curl -L -o one-api "$asset_url" && chmod +x one-api
```

新建启动脚本：

折叠代码块 BASH 复制代码

```
cat > start.sh << EOF
#!/bin/sh
# 如果你有设置主题的需要，可以取消注释下一行，然后按照自己的需求设置。
# export THEME="berry"
export TIKTOKEN_CACHE_DIR="$PWD"
# 把下一行的 PORT 改为自己放行的端口
exec ./one-api --port PORT --log-dir ./logs
EOF
```

添加可执行权限：

折叠代码块 BASH 复制代码

```
chmod +x start.sh
```

保存后回到 terminal 中，测试运行：

折叠代码块 BASH 复制代码

```
./start.sh
```

> 确定运行没有问题后，按 `Ctrl+c`即可停止运行。

使用 pm2 启动并管理：

折叠代码块 BASH 复制代码

```
pm2 start ./start.sh --name one-api
```

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#Uptime-Kuma "Uptime-Kuma")Uptime-Kuma
-----------------------------------------

受限于 FreeBSD 的平台限制，1.23 版本内置了 PlayWright ，无法运行，所以只能安装 1.22 版本。切记先去 Panel 中放行 TCP 端口。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Uptime-Kuma 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下：

折叠代码块 BASH 复制代码

```
# 下载 v1.22.1 版本的源代码
cd ~/domains && wget https://github.com/louislam/uptime-kuma/archive/refs/tags/1.22.1.zip && unzip 1.22.1.zip && rm -rf public_html && mv -f uptime-kuma-1.22.1 public_html && rm -f 1.22.1.zip && cd public_html
```

设置生产模式：

折叠代码块 BASH 复制代码

```
npm ci --production
```

下载 dist 文件：

折叠代码块 BASH 复制代码

```
wget https://github.com/louislam/uptime-kuma/releases/download/1.22.1/dist.tar.gz && tar -xzvf dist.tar.gz && rm dist.tar.gz
```

安装补充依赖：

折叠代码块 BASH 复制代码

```
npm install
```

安装过程中多少会有报错，无视就好，实际上最后可以正常运行。内置的 Cloudflared 反向代理在 FreeBSD 平台上无法使用，但是可以使用上述的外置的 Cloudflared 进行反代，使用自己的域名。

测试运行：

折叠代码块 BASH 复制代码

```
node server/server.js --port=PORT
```

> 记得把 PORT 替换成你放行的端口。确定运行没有问题后，按 `Ctrl+c`即可停止运行。

使用 pm2 管理后台运行：

折叠代码块 BASH 复制代码

```
pm2 start server/server.js --name uptime-kuma -- --port=PORT
```

> 记得把 PORT 替换成你放行的端口。

> 如果你不希望自己的 Uptime-Kuma 被人使用 `http://IP:PORT`的方式访问，你可以在最后的执行命令添加 `--host=127.0.0.1`的尾缀，这样就只能通过反向代理的域名进行访问了:
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> pm2 start server/server.js --name uptime-kuma -- --port=PORT --host=127.0.0.1
> ```

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#Bingo（暂时无法正常使用） "Bingo（暂时无法正常使用）")Bingo（暂时无法正常使用）
-----------------------------------------------------

先放行一个端口。在 Panel 中进入 File manager 选项卡，点击左侧的 My Files 进入你的用户根目录，找到 `.profile`文件，右键选择 View/Edit > Choose other >Source Editor 进行编辑，在最末加上以上两行并保存：

折叠代码块 BASH 复制代码

```
alias node='node20'
alias npm='npm20'
```

应用更改：

折叠代码块 BASH 复制代码

```
source ~/.profile
```

> 先新建一个目录用于存放 Bingo 的相关文件，进入目录后执行下述操作。

下载源码：

折叠代码块 BASH 复制代码

```
git clone https://github.com/weaigc/bingo
```

进入源码所在目录：

折叠代码块 BASH 复制代码

```
cd bingo
```

安装依赖：

折叠代码块 BASH 复制代码

```
npm20 install
```

下载 build 好的 `.next`资源：

折叠代码块 BASH 复制代码

```
wget -O next.tar.gz https://pan.saika.free.hr/d/local/next.tar.gz && tar -xzvf next.tar.gz && rm next.tar.gz
```

添加环境变量文件：

折叠代码块 BASH 复制代码

```
cp .env.example .env
```

接着在 Panel 中进入 File manager 选项卡，进入 Bingo 源码所在的目录，找到 `server.js`文件，右键选择 View/Edit > Choose other >Source Editor 进行编辑，修改第 7 行中的端口为你放行的端口。再编辑 `.env`文件，添加你的 `BING_HEADER`。

测试启动：

折叠代码块 BASH 复制代码

```
npm20 run start
```

> 确定运行没有问题后，按 `Ctrl+c`即可停止运行。

使用 pm2 启动并管理：

折叠代码块 BASH 复制代码

```
pm2 start npm --name bingo -- run start
```

[](#Refresh-gpt-chat "Refresh-gpt-chat")Refresh-gpt-chat
--------------------------------------------------------

用来对接 ninja、warpgpt 等能够使用 access_Token 作为 API Key 请求 GPT 的工具，以使用永久有效期的 Refresh_token 来获取更好的体验。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Refresh-gpt-chat 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载 refresh-gpt-chat
wget -O refresh-gpt-chat.jar https://github.com/Yanyutin753/refresh-gpt-chat/releases/download/v0.0.3/refresh-gpt-chat-0.0.3-SNAPSHOT.jar
```

使用 pm2 启动：

折叠代码块 BASH 复制代码

```
pm2 start java19 --name refresh-gpt-chat -- -jar refresh-gpt-chat.jar --server.port=端口 --server.servlet.context-path=/ --getAccessTokenUrl=https://你的ninja地址/auth/refresh_token --chatUrl=https://你的ninja地址/v1/chat/completions
```

再套域名，接下来就可以直接使用 `https://你套的域名/v1/chat/completions/` 当作 API 端点，使用 `refresh_token` 做 API_Keys ，使用 ChatGPT 了。

然后在 one-api 中添加自定义渠道， `Base URL` 填写你 `https://你套的域名`，模型填入你的 refresh_token 对应的账号所支持的模型，如果和我一样手持大把 3.5 的账号想用来做 API 用，可以选择全部 GPT3.5 的相关模型，然后在 `模型重定向`中填入以下内容：

折叠代码块 JSON 复制代码

```
{
  "gpt-3.5-turbo-0301": "gpt-3.5-turbo",
  "gpt-3.5-turbo-0613": "gpt-3.5-turbo",
  "gpt-3.5-turbo-16k": "gpt-3.5-turbo",
  "gpt-3.5-turbo-16k-0613": "gpt-3.5-turbo",
  "gpt-3.5-turbo-1106": "gpt-3.5-turbo",
  "gpt-3.5-turbo-instruct": "gpt-3.5-turbo"
}
```

密钥填写你的 `refresh_token`即可，如果你有多个账号，可以将批量勾选上，然后一行写一个 `refresh_token`。

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#Gpt4-copilot-java "Gpt4-copilot-java")[Gpt4-copilot-java](https://github.com/Yanyutin753/gpt4-copilot-java-sh)
------------------------------------------------------------------------------------------------------------------

支持 cocopilot 的 ccu 和 copilot 的 ghu 调用 copilot 转 GPT-4 的接口转换工具。 Java 写的，可以在 Serv00 运行。

> 目前更推荐这个方法：[lvguanjun/copilot-to-chatgpt4](https://blog.rappit.site/2024/02/07/copilot-to-api-free-temp/#lvguanjun-copilot-to-chatgpt4)  
> 比起 Gpt4-copilot-java 更轻量更强大。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Gpt4-copilot-java 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载 fat jar 包
RELEASE_INFO=$(curl -s "https://api.github.com/repos/Yanyutin753/gpt4-copilot-java-sh/releases/latest")
JAR_DOWNLOAD_URL=$(echo "$RELEASE_INFO" | jq -r '.assets[] | select(.name|test(".jar$")) | .browser_download_url')
curl -L -o gpt4-copilot-java.jar "$JAR_DOWNLOAD_URL"
```

测试运行：

折叠代码块 BASH 复制代码

```
# 把PORT改为自己放行的端口，最后的server.servlet.context-path参数可以改成自己喜欢的尾缀
java19 -jar gpt4-copilot-java.jar --server.port=PORT --server.servlet.context-path=/
```

> 测试没有问题之后，按 `Ctrl+c`即可停止运行。

使用 pm2 启动并管理：

折叠代码块 BASH 复制代码

```
pm2 start java19 --name gpt4-copilot-java -- -jar gpt4-copilot-java.jar --server.port=PORT --server.servlet.context-path=/
```

> 始皇的公车：ghu_ThisIsARealFreeCopilotKeyByCoCopilot （已失效）
> 
> 免费公车白嫖请求示例：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> curl --location 'http(s)://ip:port_or_URL/cocopilot/v1/chat/completions' 
> --header 'Content-Type: application/json' 
> --header 'Authorization: Bearer ghu_ThisIsARealFreeCopilotKeyByCoCopilot' 
> --data '{
> "model": "gpt-4",
> "messages": [{"role": "user", "content": "鲁迅打周树人"}]
> }'
> ```

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#Zfile "Zfile")[Zfile](https://zfile.vip/)
---------------------------------------------

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Zfile 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载 fat jar 包
wget --no-check-certificate -O zfile.jar https://c.jun6.net/ZFILE/zfile-release.jar
```

测试运行：

折叠代码块 BASH 复制代码

```
java19 -jar -Duser.timezone=Asia/Shanghai zfile.jar --zfile.log.path=$PWD/logs --zfile.db.path=$PWD/zfile --server.port=PORT
```

> 记得把端口改成自己的。测试没有问题之后，按 `Ctrl+c`即可停止运行。

使用 pm2 启动并管理：

折叠代码块 BASH 复制代码

```
pm2 start java19 --name zfile -- -jar -Duser.timezone=Asia/Shanghai zfile.jar --zfile.log.path=$PWD/logs --zfile.db.path=$PWD/zfile --server.port=PORT
```

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#Halo "Halo")Halo
--------------------

> **慎重部署，内存会超 100%，不知道会不会封号**

[halo](https://github.com/halo-dev/halo) 自从升级 2.0 版本开始，很长时间内都没有提供构筑好的 jar 包，甚至于在 GitHub 上都出现了第三方的，使用 GitHub workflow 自动化构筑 jar 包的[仓库](https://github.com/Lu7fer/Jar4Halo)。但是，自从 [2.12.0-alpha.1 版本](https://github.com/halo-dev/halo/releases/tag/v2.12.0-alpha.1)开始，halo 的官方仓库又开始提供构筑好的 jar 包了，刚好这些天在玩 Serv00 ，遂尝试部署了一下，成功。现记录一下：

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Halo 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

并在 MySQL 选项卡中中新建 MySQL 数据库，用于填入接入 Halo 。

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载jar包
release_info=$(curl -s https://api.github.com/repos/halo-dev/halo/releases/latest)
jar_url=$(echo "$release_info" | jq -r '.assets[] | select(.name | endswith(".jar")) | .browser_download_url')
curl -L "$jar_url" -o halo.jar
```

在 `halo.jar` 所在路径下新建 `.halo2` 文件夹，进入其中，新建文件 `application.yaml` 然后并配置其内容：

折叠代码块 BASH 复制代码

```
# 新建文件夹
mkdir -p .halo2
# 新建并填入配置
cat > .halo2/application.yaml << EOF
server:
  port: 你在面板中放行的端口
  # Response data gzip.
  compression:
    enabled: false
spring:
  #sql:
  #  init.platform: mysql
  r2dbc:
    url: r2dbc:pool:mysql://数据库地址:3306/数据库名
    username: 数据库用户名
    password: 数据库密码
halo:
  # Your admin client path is https://your-domain/{admin-path}
  admin-path: admin
  # memory or level
  cache: level
EOF
```

在 `halo.jar` 所在路径下新建 `run.sh` 运行脚本：

折叠代码块 BASH 复制代码

```
cat > run.sh << EOF
#!/bin/bash
export HALO_WORK_DIR="$PWD/.halo2"
export HALO_EXTERNAL_URL="https://你的域名"
exec java17 -server -Xms128m -Xmx256m -jar -Duser.timezone=Asia/Shanghai $PWD/halo.jar --spring.config.additional-location=$PWD/.halo2/application.yaml
EOF
```

测试运行：

折叠代码块 BASH 复制代码

```
chmod +x run.sh && ./run.sh
```

> 确定运行没有问题后，按 `Ctrl+c`即可停止运行。

使用 pm2 管理运行：

折叠代码块 BASH 复制代码

```
chmod +x run.sh && pm2 start ./run.sh --name halo
```

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#Go-proxy-bingai "Go-proxy-bingai")Go-proxy-bingai
-----------------------------------------------------

[这个仓库](https://github.com/adams549659584/go-proxy-bingai)是 Bingo 的前身，当初玩 Replit 时我便有在使用，只可惜作者早已弃坑，所以当初我才找到了当时还能用的 Bingo 使用。

在 Bingo 也长期未更新，无法正常使用的如今，我的目光转向了另一个[二改仓库](https://github.com/Harry-zklcdc/go-proxy-bingai)。Harry-zklcdc 维护的 Go-proxy-bingai 的分支仓库目前还能够正常使用。而且在与开发者反馈了几个 bug 之后，开发者都会花时间认真复现，并快速修复，其体验实在是不错。

~虽然原仓库的 Release 中并未提供 FreeBSD 系统适用的二进制文件，但是我们能够自己构建。我已经构建了一份放在这篇博客底部的 QQ 群的群文件中~。~我写了一个仓库用于自动化构建 FreeBSD 版本的 go-proxy-bingai ，可以从我的仓库下载使用。~ 作者已经开始提供 FreeBSD 的构建，故我的仓库已经存档。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Go-proxy-bingai 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载可执行文件
release_info=$(curl -s https://api.github.com/repos/Harry-zklcdc/go-proxy-bingai/releases | jq -r '[.[] | select(.prerelease==false)][0]')
download_url=$(echo "$release_info" | jq -r '.assets[] | select(.) | .browser_download_url')
curl -L "$download_url" -o go-proxy-bingai-freebsd-amd64.tar.gz&& tar -xzf go-proxy-bingai-freebsd-amd64.tar.gz && rm go-proxy-bingai-freebsd-amd64.tar.gz && chmod +x go-proxy-bingai
```

新建启动脚本：

折叠代码块 BASH 复制代码

```
cat > entrypoint.sh << EOF
#!/bin/bash
# 被注释的环境变量请根据自己的需求，按照原仓库的 wiki 中的介绍进行填入。
export BYPASS_SERVER="https://bypass.zklcdc.xyz"  # 作者本人的公共bypass服务，可用性未知。
# export Go_Proxy_BingAI_USER_TOKEN_1="xxx"
# export Go_Proxy_BingAI_USER_TOKEN_2="xxx"
# export USER_KievRPSSecAuth="xxx"
# export USER_RwBf="xxx"
# export USER_MUID="xxx"
# export APIKEY="sk-xxx"
# export BING_BASE_URL="https://www.bing.com"
# export SYDNEY_BASE_URL="https://sydney.bing.com"
# export HTTP_PROXY="http://172.17.0.1:18080"
# export HTTPS_PROXY="http://172.17.0.1:18080"
# export Go_Proxy_BingAI_AUTH_KEY="xxx"
# 请把下一行双引号中的内容替换成你放行的端口。
export PORT="xxx"
chmod +x go-proxy-bingai && exec ./go-proxy-bingai
EOF
```

运行：

折叠代码块 BASH 复制代码

```
# 测试运行
chmod +x entrypoint.sh && ./entrypoint.sh
# 使用 pm2 管理运行
pm2 start ./entrypoint.sh --name go-proxy-bingai
```

> **请注意，如果你需要使用其 web 功能，而不仅仅是 api 功能，请务必使用 https 访问，不然无法打开。你可以选择使用面板自带的 proxy 添加域名并申请 ssl 证书，亦或者直接使用 cloudflared 隧道。**

> 同样的，你还可以使用 Cloudflared 隧道添加域名，而不选择使用 Proxy 。

[](#Pentaract "Pentaract")[Pentaract](https://github.com/Dominux/Pentaract)
---------------------------------------------------------------------------

> 不建议使用，目前 Bug 众多，而且对 Telegram 账号有一定要求，目前暂不清楚 Telegram 限制账号的评定标准。

可以自行构建或者使用使用我构建的成品。由于该应用需要使用具有超级管理员权限的 PostgreSQL ，故不可使用 Serv00 自带的 PostgreSQL ，需要远程连接。

编译成品下载地址：[pentaract-freebsd_X64.tar.gz](https://pan.rappit.site/download/%E6%8D%AF%E9%A5%AC/pentaract-freebsd_X64.tar.gz)

前端构建简单，这里不再赘述，而且由于其 `Dockerfile` 内构建前端使用的是 Node.js 21 而目前 FreeBSD Port 最高只有 Node.js 20 ，故不推荐在 FreeBSD 上直接构建，可以使用 GItHub Actions 进行构建，或是自己在 Node.js 21 的环境下构建再复制，甚至干脆直接从作者预构建的 Docker 镜像内打包出来使用。（经过测试，使用 Nodejs20 构建也可以正常使用。）

Serv00 上的构建法：

折叠代码块 BASH 复制代码

```
# 切换 Node.js 版本为 Nodejs20
alias node=node20
alias npm=npm20
# 全局安装 pnpm
npm install -g pnpm
source ~/.bashrc
# 构建前端
pnpm install
VITE_API_BASE='/api' pnpm run build
# 移动构建产物到工作目录
mkdir -p ~/pentaract/ui && cp -R ./dist/* ~/pentaract/ui
```

后端的构建，可以使用 GItHub Actions ，或者本地 FreeBSD 虚拟机，甚至直接在 Serv00 上构建。这里记录一下在 Serv00 上构建的方法：

折叠代码块 BASH 复制代码

```
# 克隆仓库到 Serv00 上
git clone https://github.com/Dominux/Pentaract && cd Pentaract/pentaract
# 构建
LIBCLANG_PATH=/usr/local/llvm16/lib cpuset -l 0 cargo build --release
# 移动构建产物到工作目录
mkdir -p ~/pentaract && cp ./target/release/pentaract ~/pentaract/pentaract
```

然后去 [supabase](https://supabase.com/) 注册一个免费的 PostgreSQL ，记录下数据库的用户名、密码、数据库名、地址，用于后续填入环境变量。

接着在 `~/pentaract` 路径下新建一个启动脚本，按照要求填入所有的环境变量：

折叠代码块 BASH 复制代码

```
cat > start.sh << EOF
#!/bin/bash
export PORT=xxxx
export WORKERS=4
export CHANNEL_CAPACITY=32
export SUPERUSER_EMAIL=xxxx@xxxx.com
export SUPERUSER_PASS=xxxx
export ACCESS_TOKEN_EXPIRE_IN_SECS=1800
export REFRESH_TOKEN_EXPIRE_IN_DAYS=14
export SECRET_KEY=xxx
export TELEGRAM_API_BASE_URL=https://api.telegram.org
export DATABASE_USER=xxxx
export DATABASE_PASSWORD=xxxx
export DATABASE_NAME=xxxx
export DATABASE_HOST=xxxx
export DATABASE_PORT=5432
chmod +x pentaract && exec ./pentaract
EOF
```

给启动脚本赋权：

折叠代码块 BASH 复制代码

```
chmod +x start.sh
```

~前端的 `index-22eec6d1.js` 文件内的 `http://localhost:8000` 需要更改为 serv00 的 url 或者 ip:port 。你可以去文件管理中编辑，查找替换即可，也可以使用 sed 命令简单更改一下：~ 已经重新构建前端并替换，现无需此步。

测试运行：

折叠代码块 BASH 复制代码

```
./start.sh
```

使用 pm2 管理：

折叠代码块 BASH 复制代码

```
pm2 start ./start.sh --name pentaract
```

[](#OneList "OneList")[OneList](https://github.com/msterzhang/onelist)
----------------------------------------------------------------------

原作者似乎已经弃坑，故我的仓库没有做自动检测构建。但是体验还不错，有 Emby 既视感了，配合小雅的 Alist 岂不美哉。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 OneList 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载 OneList
wget https://github.com/k0baya/onelist-freebsd/releases/download/v2.0.5/onelist
# 初始化配置
chmod +x onelist && ./onelist -run config
```

接着回到 Panel 中，进入 File manager 选项卡，进入 OneList 所在路径，可以看到一个名为 `config.env` 的文件，右键点击，选择 View/Edit > Source Editor，进行编辑：

折叠代码块 复制代码

```
# 服务设置
# 注意要改为未被占用的端口
API_PORT=5245
FaviconicoUrl=https://wework.qpic.cn/wwpic/818353_fizV30xbQCGPQRP_1677394564/0
API_SECRET=fRVvjcNd11gYGI85StVaeCtPVSmJTRRE

# Env有两种模式，Debug及Release，主要用在数据库为mysql时候，需要注意修改Env环境和mysql密码对应
Env=Debug

# 管理员账户设置，用于初始化管理员账户
UserEmail=xxxx.@qq.com
UserPassword=xxxxx

# 数据库设置
DB_DRIVER=sqlite
DB_USER=root
DbName=onelist

# 如果上面DB_DRIVER类型为mysql，就需要正确填下以下参数
DB_PASSWORD_Debug=123456
DB_PASSWORD_Release=123456

# TheMovieDb Key
# 在https://www.themoviedb.org网站申请
KeyDb=22f10ca52f109158ac7fe064ebbcf697
```

你可以按照自己的需求配置端口、管理员账户、数据库。 MySQL 性能更好哦~

测试运行：

折叠代码块 BASH 复制代码

```
./onelist -run server
```

使用 pm2 管理：

折叠代码块 BASH 复制代码

```
pm2 start ./onelist -- -run server
```

[](#WarpGPT "WarpGPT")[WarpGPT](https://github.com/oliverkirk-sudo/WarpGPT)
---------------------------------------------------------------------------

这个没什么多说的，可以使用 access_Token 作为 API Key 请求 ChatGPT 接口，也就是所谓的 chat2api 。配合前文的 Refresh-gpt-chat 就可以把永久有效期的 Refresh_token 作为 API Key 来使用，十分的好用。

源仓库没有 Release ，故[我的仓库](https://github.com/k0baya/warpgpt-freebsd)没有做自动检测构建。如果有更新需求需要手动触发 workflow 。你有需要也可以自己 fork 一份然后手动触发 workflow 。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 WarpGPT 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载二进制文件
wget https://github.com/k0baya/warpgpt-freebsd/releases/download/latest/warpgpt && chmod +x warpgpt
```

添加启动脚本：

折叠代码块 BASH 复制代码

```
cat > start.sh << EOF
#!/bin/bash
export TMPDIR="$PWD"
chmod +x warpgpt && exec ./warpgpt
EOF
```

给启动脚本赋权：

折叠代码块 BASH 复制代码

```
chmod +x start.sh
```

配置环境变量：

折叠代码块 BASH 复制代码

```
cat > .env << EOF
proxy = "http://127.0.0.1:10809"   #代理地址 （选填）
port = 5000                        #程序运行端口
host = '127.0.0.1'                 #可访问ip，0.0.0.0允许所有ip
verify = false                     #是否对访问进行验证
auth_key = ""                      #若开启访问验证，则需要在Header中添加AuthKey字段，且值为auth_key的值才能访问 （选填）
arkose_must = false                #是否强行gpt3.5进行验证
OpenAI_HOST = "chat.openai.com"    #openai网页api接口地址 （选填）
openai_api_host = "api.openai.com" #openai官方api接口 （选填）
proxy_pool_url=""                  #ipidea代理池链接 （选填）
#示例http://api.proxy.ipidea.io/getProxyIp?num=10&return_type=json&lb=1&sb=0&flow=1®ions=us&protocol=http，根据访问频次设置num值
log_level = "debug"                #日志等级

redis_address = "127.0.0.1:6379"   #redis地址（若不开启代理池可选填）
redis_passwd = ""                  #redis密码
redis_db = 0                       #选择的redis数据库
EOF
```

> 如果有 redis 需求，可以查阅官方文档：[Redis](https://docs.serv00.com/Redis/)

使用 pm2 管理运行：

折叠代码块 BASH 复制代码

```
pm2 start bash --name warpgpt -- start.sh
```

[](#Coze-discord-proxy "Coze-discord-proxy")[Coze-discord-proxy](https://github.com/deanxv/coze-discord-proxy)
--------------------------------------------------------------------------------------------------------------

代理 Discord 对话 Coze-Bot ，实现以 API 形式请求 GPT4 模型，提供对话、文生图、图生文、知识库检索等功能。功能不多赘述，详细去源仓库查看。

同样的，我写了一个用于构建 FreeBSD 版本的[仓库](https://github.com/k0baya/coze-discord-proxy-freebsd)。在这里感谢论坛用户 [Reno](https://linux.do/u/reno/summary) 的测试，没有测试人员的测试，也不会有部署的过程记录了。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Coze-discord-proxy 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
# 下载二进制文件
release_info=$(curl -s https://api.github.com/repos/k0baya/coze-discord-proxy-freebsd/releases/latest)
asset_url=$(echo "$release_info" | jq -r '.assets[] | select(.name != "source code") | .browser_download_url')
curl -L -o coze-discord-proxy "$asset_url" && chmod +x coze-discord-proxy
```

添加启动脚本：

折叠代码块 BASH 复制代码

```
cat > start.sh << EOF
#!/bin/bash
# 根据你的需求自行填入环境变量
export PORT="PORT"
export USER_AUTHORIZATION="XXXXXX"
export BOT_TOKEN="XXXXXX"
export GUILD_ID="XXXXXX"
export COZE_BOT_ID="XXXXXX"
export PROXY_SECRET="XXXXXX"
export CHANNEL_ID="XXXXXX"
export TZ="Asia/Shanghai"
export DATA_GYM_CACHE_DIR="$PWD"
chmod +x coze-discord-proxy && exec ./coze-discord-proxy
EOF
```

给启动脚本赋权：

折叠代码块 BASH 复制代码

```
chmod +x coze-discord-proxy
```

添加多机器人配置文件：

折叠代码块 BASH 复制代码

```
mkdir -p app/coze-discord-proxy/data/config
touch app/coze-discord-proxy/data/config/bot_config.json
```

然后回到 Panel 中，进入 File manager 选项卡，进入 `bot_config.json` 所在路径，右键点击它，选择 View/Edit > Source Editor，进行编辑：

折叠代码块 复制代码

```
[
  {
    "proxySecret": "123", // 接口请求密钥(PROXY_SECRET)(注意:此密钥在环境变量PROXY_SECRET中存在时该Bot才可以被匹配到!)
    "cozeBotId": "12***************31", // coze托管的机器人ID
    "model": ["gpt-3.5","gpt-3.5-16k"], // 模型名称(数组格式)(与请求参数中的model对应,如请求中的model在该json中未匹配到则会抛出异常)
    "channelId": "12***************56"  // [可选]discord频道ID(机器人必须在此频道所在的服务器)(目前版本下该参数仅用来活跃机器人)
  },
  {
    "proxySecret": "456",
    "cozeBotId": "12***************64",
    "model": ["gpt-4","gpt-4-16k"],
    "channelId": "12***************78"
  },
  {
    "proxySecret": "789",
    "cozeBotId": "12***************12",
    "model": ["dall-e-3"],
    "channelId": "12***************24"
  }
]
```

使用 pm2 管理运行：

折叠代码块 BASH 复制代码

```
pm2 start bash --name coze-discord-proxy -- start.sh
```

[](#Memos "Memos")[Memos](https://github.com/usememos/memos)
------------------------------------------------------------

一款开源、轻量级的笔记服务。轻松捕捉并分享您的精彩想法。

这个仓库比较难受的是，其在源码的[这个位置](https://github.com/usememos/memos/blob/edc7645086d285f50e484861705ffee3a626f97a/server/server.go#L85)强制要求其 gRPC 服务的端口为 Memos 监听端口 + 1，故这个应用需要占用两个端口，而且必须是两个连续的端口。

同样的，我写了一个用于构建 FreeBSD 版本的[仓库](https://github.com/k0baya/memos-binary)。

首先在 Panel 中放行**两个相邻的端口**，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>刚刚放行的<strong>两个相邻的端口中小的那一个</strong></td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
API_URL="https://api.github.com/repos/k0baya/memos-binary/releases/latest"
DOWNLOAD_URL=$(curl -s $API_URL | jq -r ".assets[] | select(.name == \"memos-freebsd-amd64.tar.gz\") | .browser_download_url")
curl -L $DOWNLOAD_URL -o memos-freebsd-amd64.tar.gz
tar -xzvf memos-freebsd-amd64.tar.gz && rm memos-freebsd-amd64.tar.gz && chmod +x memos
```

关于运行，有两种方式进行：

① SQLite

如果选择使用 SQLite 作为数据库运行，则可以直接运行：

折叠代码块 BASH 复制代码

```
# 假定你的数据文件打算存储在 /home/username/domains/xxx.USERNAME.serv00.net/public_html/data
# 新建数据文件夹
mkdir -p /home/username/domains/xxx.USERNAME.serv00.net/public_html/data
# 测试运行
./memos --mode prod --port PORT --data /home/username/domains/xxx.USERNAME.serv00.net/public_html/data
# 使用 pm2 管理
pm2 start ./memos --name memos -- --mode prod --port PORT --data /home/username/domains/xxx.USERNAME.serv00.net/public_html/data
```

② 外接 MySQL / PostgreSQL

你可以使用面板自带的 MySQL / PostgreSQL 新建数据库，或者使用其他平台提供的远程数据库：

折叠代码块 BASH 复制代码

```
# 假定你的数据文件打算存储在 /home/username/domains/xxx.USERNAME.serv00.net/public_html/data
# 新建数据文件夹
mkdir -p /home/username/domains/xxx.USERNAME.serv00.net/public_html/data
# 测试运行（MySQL）（MySQL需要管理员权限，你可以选择远程连接）
./memos --mode prod --port PORT --data /home/username/domains/xxx.USERNAME.serv00.net/public_html/data --driver mysql --dsn mysql://root:password123@localhost:3306/mydb
# 测试运行（PostgreSQL）
./memos --mode prod --port PORT --data /home/username/domains/xxx.USERNAME.serv00.net/public_html/data --driver postgres --dsn postgresql://user:password123@localhost:5432/mydb?sslmode=disable
# 使用 pm2 管理（MySQL）（MySQL需要管理员权限，你可以选择远程连接）
pm2 start ./memos --name memos -- --mode prod --port PORT --data /home/username/domains/xxx.USERNAME.serv00.net/public_html/data --driver mysql --dsn mysql://root:password123@localhost:3306/mydb
# 使用 pm2 管理（PostgreSQL）
pm2 start ./memos --name memos -- --mode prod --port PORT --data /home/username/domains/xxx.USERNAME.serv00.net/public_html/data --driver postgres --dsn postgresql://user:password123@localhost:5432/mydb?sslmode=disable
```

[](#Frps "Frps")Frps
--------------------

内网穿透嘛，懂的都懂，这里只做服务端的部署记录，客户端可以查看 [Frp 的官方文档](https://gofrp.org/zh-cn/)自行配置。感谢群友的率先测试：[youyi](https://blog.theyouyi.site/archives/serv00-frps)

首先在 Panel 中放行两个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来映射转发内网服务的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

> 同样的，你可以设置多个域名使用 Proxy 指向同一个端口，在 Frpc 客户端配置中使用域名分发不同的服务。具体可以查阅官方文档。

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下

折叠代码块 BASH 复制代码

```
release_info=$(curl -s https://api.github.com/repos/fatedier/frp/releases/latest)
download_url=$(echo "$release_info" | jq -r '.assets[] | select(.name | contains("freebsd_amd64.tar.gz")) | .browser_download_url')
curl -L "$download_url" -o frp_freebsd_amd64.tar.gz 
tar -xzvf frp_freebsd_amd64.tar.gz --strip-components=1
rm -rf frp_freebsd_amd64.tar.gz
```

接着编辑配置文件：

折叠代码块 BASH 复制代码

```
# 按照自己的实际情况和需求进行配置，这里只做最简单的http转发配置示例
cat > frps.toml << EOF
bindPort = 监听端口
vhostHTTPPort = 映射转发端口
auth.token = "密码"
EOF
```

运行：

折叠代码块 BASH 复制代码

```
pm2 start ./frps -- -c frps.toml
```

> 客户端配置示例：
> 
> 折叠代码块 TOML 复制代码
> 
> ```
> serverAddr = "x.x.x.x"
> serverPort = Frps 的监听端口
> auth.token = "密码"
> 
> [[proxies]]
> name = "web"
> type = "http"
> localPort = 80
> customDomains = ["www.yourdomain.com"]
> 
> [[proxies]]
> name = "web2"
> type = "http"
> localPort = 8080
> customDomains = ["www.yourdomain2.com"]
> ```

[](#Rclone "Rclone")[Rclone](https://rclone.org/)
-------------------------------------------------

Rclone 是一款管理云存储文件的命令行程序。它功能丰富，可替代云供应商的网络存储界面。超过 70 种云存储产品支持 Rclone，包括 S3 对象存储、企业和消费者文件存储服务以及标准传输协议。

具体用法与配置请查阅其[官方文档](https://rclone.org/docs/)。

如果你需要使用 Rclone 的 web ui ，你可以按照前文所述的大多数应用一样，先放行端口，添加域名，申请好 SSL 证书，并进入其目录下的 `public_html` 路径下再进行程序本体的下载部署。

下载最新版 Rclone：

折叠代码块 BASH 复制代码

```
release_info=$(curl -s https://api.github.com/repos/rclone/rclone/releases/latest)
download_url=$(echo "$release_info" | jq -r '.assets[] | select(.name | contains("-freebsd-amd64.zip")) | .browser_download_url')
curl -L "$download_url" -o rclone-freebsd-amd64.zip
outer_folder=$(unzip -l rclone-freebsd-amd64.zip | grep '/' | sed -n '1p' | sed 's#^.* \([^/]*\)/.*$#\1#')
unzip rclone-freebsd-amd64.zip
mv "$outer_folder"/* . && rm -rf "$outer_folder" rclone-freebsd-amd64.zip
```

经我测试，目前 v1.63.1 之后的版本的 FreeBSD 版的构建都有无法识别 `mount` 命令的问题，在我查阅其 issue —— [#7432](https://github.com/rclone/rclone/issues/7432) 、 [#5843](https://github.com/rclone/rclone/issues/5843#issuecomment-1784149722) 后，确定这个 bug 已经好几个月没有修复了。所以我建议在此 bug 修复前，使用 v1.63.1 版本。

下载 v1.63.1 版本 Rclone ：

折叠代码块 BASH 复制代码

```
curl -L https://github.com/rclone/rclone/releases/download/v1.63.1/rclone-v1.63.1-freebsd-amd64.zip -o rclone-freebsd-amd64.zip
outer_folder=$(unzip -l rclone-freebsd-amd64.zip | grep '/' | sed -n '1p' | sed 's#^.* \([^/]*\)/.*$#\1#')
unzip rclone-freebsd-amd64.zip
mv "$outer_folder"/* . && rm -rf "$outer_folder" rclone-freebsd-amd64.zip
```

配置 Rclone 的存储：

折叠代码块 BASH 复制代码

```
./rclone config
```

> 启动 web ui：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> ./rclone rcd --rc-web-gui --rc-user 用户名 --rc-pass 密码 --rc-addr :端口
> ```
> 
> pm2 管理 web ui：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> pm2 start ./rclone -- rcd --rc-web-gui --rc-user 用户名 --rc-pass 密码 --rc-addr :端口
> ```

[](#Cloudreve "Cloudreve")[Cloudreve](https://cloudreve.org/)
-------------------------------------------------------------

Cloudreve 可助你即刻构建出兼备自用或公用的网盘服务，通过多种存储策略的支持、虚拟文件系统等特性实现灵活的文件管理体验。

同样的，我编写了一个用于自动化构建 FreeBSD 版本的 Cloudreve 的仓库：[k0baya/cloudreve-freebsd](https://github.com/k0baya/cloudreve-freebsd) 前后端分离构建，前端静态文件在 Cloudreve 本体同路径下的 `static` 文件夹内。

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 Cloudreve 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
release_info=$(curl -s https://api.github.com/repos/k0baya/cloudreve-freebsd/releases/latest)
download_url=$(echo "$release_info" | jq -r '.assets[] | select(.name | contains("freebsd-amd64.tar.gz")) | .browser_download_url')
curl -L "$download_url" -o cloudreve-freebsd-amd64.tar.gz 
tar -xzvf cloudreve-freebsd-amd64.tar.gz
rm -rf cloudreve-freebsd-amd64.tar.gz
```

Cloudreve 在首次启动时，会创建初始管理员账号，请注意保管管理员密码，此密码只会在首次启动时出现。如果您忘记初始管理员密码，需要删除同级目录下的 `cloudreve.db` ，重新启动主程序以初始化新的管理员账户。

Cloudreve 默认会监听 `5212` 端口。首次启动时，Cloudreve 会在同级目录下创建名为 `conf.ini` 的配置文件，你可以修改此文件进行一些参数的配置（比如端口），保存后需要重新启动 Cloudreve 生效。

一个完整的配置文件示例如下：

折叠代码块 INI 复制代码

```
[System]
; 运行模式
Mode = master
; 监听端口
Listen = :5212
; 是否开启 Debug
Debug = false
; Session 密钥, 一般在首次启动时自动生成
SessionSecret = 23333
; Hash 加盐, 一般在首次启动时自动生成
HashIDSalt = something really hard to guss
; 呈递客户端 IP 时使用的 Header
ProxyHeader = X-Forwarded-For

; SSL 相关
[SSL]
; SSL 监听端口
Listen = :443
; 证书路径
CertPath = C:\Users\i\Documents\fullchain.pem
; 私钥路径
KeyPath = C:\Users\i\Documents\privkey.pem

; 启用 Unix Socket 监听
[UnixSocket]
Listen = /run/cloudreve/cloudreve.sock
; 设置产生的 socket 文件的权限
Perm = 0666

; 数据库相关，如果你只想使用内置的 SQLite 数据库，这一部分直接删去即可
[Database]
; 数据库类型，目前支持 sqlite/mysql/mssql/postgres
Type = mysql
; MySQL 端口
Port = 3306
; 用户名
User = root
; 密码
Password = root
; 数据库地址
Host = 127.0.0.1
; 数据库名称
Name = v3
; 数据表前缀
TablePrefix = cd_
; 字符集
Charset = utf8mb4
; SQLite 数据库文件路径
DBFile = cloudreve.db
; 进程退出前安全关闭数据库连接的缓冲时间
GracePeriod = 30
; 使用 Unix Socket 连接到数据库
UnixSocket = false

; 从机模式下的配置
[Slave]
; 通信密钥
Secret = 1234567891234567123456789123456712345678912345671234567891234567
; 回调请求超时时间 (s)
CallbackTimeout = 20
; 签名有效期
SignatureTTL = 60

; 跨域配置
[CORS]
AllowOrigins = *
AllowMethods = OPTIONS,GET,POST
AllowHeaders = *
AllowCredentials = false
SameSite = Default
Secure = lse

; Redis 相关
[Redis]
Server = 127.0.0.1:6379
Password =
DB = 0

; 从机配置覆盖
[OptionOverwrite]
; 可直接使用 `设置名称 = 值` 的格式覆盖
max_worker_num = 50
```

你可以使用 `vim` 或者 Panel 中的 File manager 选项卡，进入 `conf.ini` 所在路径路径，右键点击，选择 View/Edit > Source Editor ，进行编辑。

修改完配置文件后，测试启动：

折叠代码块 BASH 复制代码

```
./cloudreve
```

使用 pm2 管理：

折叠代码块 BASH 复制代码

```
pm2 start ./cloudreve
```

[](#PanIndex "PanIndex")[PanIndex](https://github.com/px-org/PanIndex)
----------------------------------------------------------------------

一个简易的网盘目录列表。

同样的，我编写了一个用于自动化构建 FreeBSD 版本的 PanIndex 的仓库：[k0baya/panindex-freebsd](https://github.com/k0baya/panindex-freebsd)。

> 后台地址（默认）：`http://ip:port/admin`  
> 默认账号：`admin`  
> 默认密码：`PanIndex`

首先在 Panel 中放行一个端口，接着按照下表 Add a New Website ：

<table><thead><tr><th>Key</th><th>Value</th></tr></thead><tbody><tr><td>Domain</td><td><code>xxx.USERNAME.serv00.net</code>（也可以把原有的 USERNAME.serv00.net 删掉后重新添加）</td></tr><tr><td>Website Type</td><td>proxy</td></tr><tr><td>Proxy Target</td><td>localhost</td></tr><tr><td>Proxy URL</td><td>留空</td></tr><tr><td>Proxy port</td><td>你准备用来部署 PanIndex 的端口</td></tr><tr><td>Use HTPPS</td><td>False</td></tr><tr><td>DNS support</td><td>True</td></tr></tbody></table>

添加完新站点后，继续点击上方的 Manage SSL certificates ，接着在出口 IP 的右侧点击 Manage ，再点击 Add certificate ：

<table><thead><tr><th>Type</th><th>Domain</th></tr></thead><tbody><tr><td>Generate Let’s Encrypted certificate</td><td>与刚刚添加的站点域名保持一致（如果是原有的<code>USERNAME.serv00.net</code> ，可以省略此步）</td></tr></tbody></table>

接着 SSH 登入，并进入刚刚你新建的域名目录下的 `public_html` 路径下：

折叠代码块 BASH 复制代码

```
release_info=$(curl -s https://api.github.com/repos/k0baya/panindex-freebsd/releases/latest)
asset_url=$(echo "$release_info" | jq -r '.assets[] | select(.name != "source code") | .browser_download_url')
curl -L -o panindex "$asset_url" && chmod +x panindex
```

创建配置文件：

折叠代码块 BASH 复制代码

```
cat > config.json << EOF
{
  "host": "0.0.0.0",
  "port": 5238,
  "log_level": "info",
  "data_path": "",
  "cert_file": "",
  "key_file": "",
  "config_query": "",
  "db_type": "",
  "dsn": "",
  "ui": ""
}
EOF
```

> 数据库支持 sqlite (默认)、mysql、postgres ，如果需要接入 MySQL 或者 PostgreSQL ，请写成数据库链接的方式填入 dsn 。注意，如果是 Serv00 自带的 PostgreSQL ，请在数据库链接最末加上 `?sslmode=disable` 以禁用 SSL 连接。

编写好配置文件后，测试运行：

折叠代码块 BASH 复制代码

```
./panindex -c=config.json
```

使用 pm2 管理：

折叠代码块 BASH 复制代码

```
pm2 start ./panindex -- -c=config.json
```

[](#Artalk "Artalk")[Artalk](https://github.com/ArtalkJS/Artalk)
----------------------------------------------------------------

似乎有几个群友在用这个，为方便查阅统一收录在本文，具体内容可以去群友的博客查看：  
[![](https://blog.sinzmise.top/img/avatar.png)](https://blog.sinzmise.top/posts/13624/)

> 点击图片进入

[](#收尾工作 "收尾工作")收尾工作
====================

听说 Serv00 会不定时重启机器，所以我们把 pm2 添加开机自启，可以保证每次重启都能由 pm2 调动 Alist 和 Cloudflared 。而且 Serv00 每三个月内必须要有一次登录面板或者 SSH 连接，不然会删号，也可以通过一个脚本解决问题，接下来我会详细说明。

[](#自动续期 "自动续期")自动续期
--------------------

可以用青龙面板的自动任务定期登录 SSH 解决。在青龙面板中添加 Linux 依赖 `sshpass`，然后添加定时任务：名称随意，命令 / 脚本 `sshpass -p '密码' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -tt 用户名@地址 "exit"`，定时规则 `1 1 1 * *`。这样就会每个月自动 ssh 连接一次，实现续期。

> 你还可以使用自身 SSH 自身的方式进行自动续期，操作如下：
> 
> 进入一个自己喜欢的路径，使用 `cat` 命令新建 `auto-renew.sh` 脚本：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> cat > auto-renew.sh << EOF
> #!/bin/bash
> 
> while true; do
>   sshpass -p '密码' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -tt 用户名@地址 "exit" &
>   sleep 259200  #30天为259200秒
> done
> EOF
> ```
> 
> 记得把其中的密码、用户名、ssh 的地址修改为你自己的。
> 
> 给 `auto-renew.sh`添加可执行权限：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> chmod +x auto-renew.sh
> ```
> 
> 使用 pm2 启动：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> pm2 start ./auto-renew.sh
> ```
> 
> 这样就会每隔一个月自动执行一次 SSH 连接，自己 SSH 自己进行续期。

> 上述自动续期脚本，在当时写的时候，犯了两个错误：
> 
> 第一是时间写错了，30 天不是 259200 秒，是 2592000 秒，经群友发现后，就成了一个防伪标志，因为本文在网络上大量被复制粘贴却不标明出处，让太多人赚了流量却不知道真正码字的是谁。抄袭可耻，这样的抄袭行为只会让真正有能力创作的人失去继续创作的热情。
> 
> 第二是不应该在脚本内写成 `while` 循环配合 `sleep` 定时，这样导致大量的 `sleep` 进程把进程数占满而出现异常。正确的做法应该是：  
> 使用下述命令新建 `auto_renew.sh` 脚本：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> cat > auto_renew.sh << EOF
> #!/bin/bash
> 
> sshpass -p '密码' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -tt 用户名@地址 "exit" &
> EOF
> ```
> 
> 记得把其中的密码、用户名、ssh 的地址修改为你自己的。
> 
> 给 `auto_renew.sh`添加可执行权限：
> 
> 折叠代码块 BASH 复制代码
> 
> ```
> chmod +x auto_renew.sh
> ```
> 
> 再去 Panel 中找到 Cron jobs 选项卡，使用 Add cron job 功能添加任务，Specify time 选择 Monthly，Form type 选择 Advanced。Command 写 `auto_renew.sh` 脚本文件的绝对路径，如 `/home/username/auto_renew.sh 2>/dev/null 2>&1` 即可。

[](#自动启动 "自动启动")自动启动
--------------------

听说 Serv00 的主机会不定时重启，所以需要添加自启任务。

在 Panel 中找到 Cron jobs 选项卡，使用 Add cron job 功能添加任务，Specify time 选择 After reboot，即为重启后运行。Form type 选择 Advanced，Command 写：

折叠代码块 BASH 复制代码

```
/home/你的用户名/.npm-global/bin/pm2 resurrect 2>/dev/null 2>&1 && /home/你的用户名/.npm-global/bin/pm2 restart all 2>/dev/null 2>&1
```

> 记得把你的用户名改为你的用户名

添加完之后，在 SSH 窗口保存 pm2 的当前任务列表快照：

折叠代码块 BASH 复制代码

```
pm2 save
```

这样每次 serv00 不定时重启任务时，都能自动调用 pm2 读取保存的任务列表快照，恢复任务列表。**如果在保存了任务列表快照后又改变了任务 pm2 的任务列表，需要重新执行 `pm2 save` 以更新任务列表。**

### [](#检测启动情况 "检测启动情况")检测启动情况

由于自从 S2 开始的新的 Server 推出后屡屡出现使用 Cron Job 的 After reboot 无法恢复任务列表的情况，而过了好几个月也并没有人找到令人满意的解决方案，循环监控网页状态码并在出错时尝试拉起任务是可行的，只可惜我只看到了 Python 脚本，而此等简单的功能在我看来实在没必要使用 Python ，毕竟如果要在没有 Python 环境的情况下运行又成了一种麻烦，所以我决定通过 Shell 实现此功能。

折叠代码块 BASH 复制代码

```
#!/bin/bash

USERNAME=''
PASSWORD=''
SSH_ADDRESS=''
SERVER_ADDRESS=''
SMTP_SERVER_ADDRESS=''
SMTP_SERVER_PORT=''
SMTP_EMAIL=''
SMTP_PASSWORD=''
TARGET_EMAIL=''

check_health() {
    local CODE=$(curl -o /dev/null -s -w "%{http_code}\n" --connect-timeout 10 --max-time 30 ${SERVER_ADDRESS})
    if [ "$CODE" != "200" ]; then
        echo 'Server is down!'
        return 1
    fi
    return 0
}

send_mail() {
    if [ -n "${SMTP_SERVER_ADDRESS}" ] && [ -n "${SMTP_SERVER_PORT}" ] && [ -n "${SMTP_EMAIL}" ] && [ -n "${SMTP_PASSWORD}" ] && [ -n "${TARGET_EMAIL}" ]; then
        curl --ssl-reqd \
          --url "smtps://${SMTP_SERVER_ADDRESS}:${SMTP_SERVER_PORT}" \
          --user "${SMTP_EMAIL}:${SMTP_PASSWORD}" \
          --mail-from "${SMTP_EMAIL}" \
          --mail-rcpt "${TARGET_EMAIL}" \
          -T - <<EOF
From: ${SMTP_EMAIL}
To: ${TARGET_EMAIL}
Subject: ${EMAIL_TITLE}

${EMAIL_DATA}
EOF
    else
        echo 'Cancel send email due to missing or empty configuration.'
    fi
}

restart_server() {
    echo 'Trying to restart server...'
    local REMOTE_COMMAND1="/home/${USERNAME}/.npm-global/bin/pm2 resurrect && exit" \
    && sshpass -p "${PASSWORD}" ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -tt ${USERNAME}@${SSH_ADDRESS} "$(eval echo $REMOTE_COMMAND1)" \
    && local REMOTE_COMMAND2="/home/${USERNAME}/.npm-global/bin/pm2 restart all && exit" \
    && sshpass -p "${PASSWORD}" ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -tt ${USERNAME}@${SSH_ADDRESS} "$(eval echo $REMOTE_COMMAND2)" 
}

check_health
EXIT_CODE=$?
if [ ${EXIT_CODE} -eq 1 ]; then
    EMAIL_TITLE='Server is down!'
    EMAIL_DATA="${SERVER_ADDRESS} is down, trying to restart server..."
    send_mail

    while [ ${EXIT_CODE} -eq 1 ]; do
        restart_server
        sleep 120
        check_health
        EXIT_CODE=$?
    done

    if [ ${EXIT_CODE} -eq 0 ]; then
        EMAIL_TITLE='Server is up!'
        EMAIL_DATA="${SERVER_ADDRESS} has been restarted successfully."
        send_mail
    fi
fi
```

在脚本开头处填好所有的环境变量即可使用：

<table><thead><tr><th>变量名</th><th>是否必须</th><th>备注</th></tr></thead><tbody><tr><td>USERNAME</td><td>是</td><td>Serv00 的账户用户名</td></tr><tr><td>PASSWORD</td><td>是</td><td>Serv00 的账户密码</td></tr><tr><td>SSH_ADDRESS</td><td>是</td><td>Serv00 的账户 SSH 连接的地址（如<code>s4.serv00.com</code>）</td></tr><tr><td>SERVER_ADDRESS</td><td>是</td><td>需要检测的 Web 服务地址（如<code>https://xxx.username.serv00.net</code>）</td></tr><tr><td>SMTP_SERVER_ADDRESS</td><td>否</td><td>发送通知邮件的 SMTP 服务器地址（如<code>mail4.serv00.com</code>、<code>smtp.163.com</code>）</td></tr><tr><td>SMTP_SERVER_PORT</td><td>否</td><td>发送通知邮件的 SMTP 服务器端口，必须使用 SSL（如<code>465</code>、<code>995</code>）</td></tr><tr><td>SMTP_EMAIL</td><td>否</td><td>用于发送通知邮件的邮箱</td></tr><tr><td>SMTP_PASSWORD</td><td>否</td><td>用于发送通知邮件的邮箱的密码或秘钥</td></tr><tr><td>TARGET_EMAIL</td><td>否</td><td>用于接收通知邮件的邮箱</td></tr></tbody></table>

在你的用户目录下任意路径新建一个名为 `check_health.sh` 的文件，把上述脚本填好环境变量后粘贴进去，并在 Console 中使用 `chmod +x check_health.sh` 命令为脚本添加可执行权限。

再去 Panel 中找到 Cron jobs 选项卡，使用 Add cron job 功能添加任务，Specify time 选择 Special manually，Form type 选择 Advanced， 然后把下方的 Minute、Hour、Day of month、Month、Day of week 都改成 Every ，然后在 Minute 后填入 5（即每 5 分钟一次），其他的都填 1。Command 写 `check_health.sh` 脚本文件的绝对路径，如 `/home/username/check_health.sh 2>/dev/null 2>&1` 即可。你也可以根据你的需求自由修改定时。

还有一个加入了循环的版本，有需求的可以移植到其他服务使用：

折叠代码块 BASH 复制代码

```
# 各功能函数与前文一致，主函数加入循环，只展示主函数部分
while true; do
    check_health
    EXIT_CODE=$?
    if [ ${EXIT_CODE} -eq 1 ]; then
        EMAIL_TITLE='Server is down!'
        EMAIL_DATA="${SERVER_ADDRESS} is down, trying to restart server..."
        send_mail

        while [ ${EXIT_CODE} -eq 1 ]; do
            restart_server
            sleep 120
            check_health
            EXIT_CODE=$?
        done

        if [ ${EXIT_CODE} -eq 0 ]; then
            EMAIL_TITLE='Server is up!'
            EMAIL_DATA="${SERVER_ADDRESS} has been restarted successfully."
            send_mail
        fi
    fi
    sleep 300
done
```

[](#常用指令 "常用指令")常用指令
--------------------

折叠代码块 BASH 复制代码

```
# Enables the ability to run your own software
devil binexec on
# Set Devil and shell language to English
devil lang set english
# Get a list of all available IP addresses owned by Serv00.com
devil vhost list public
# Display the list of reserved ports
devil port list
# 伪重置（删除用户所有文件）
chmod -R 755 ~/*
chmod -R 755 ~/.*
rm -rf ~/*
rm -rf ~/.*
# 关闭用户所有进程
killall -u $(whoami)
```

**欢迎进群讨论，一起学习探讨：[受虐滑稽](https://jq.qq.com/?_wv=1027&k=qssjFvAs)**