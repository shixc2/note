> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.rappit.site](https://blog.rappit.site/2023/12/31/some_server_on_my_hax/)

> A Reading focusing Blog

前些日子申请了一台 [Hax](https://hax.co.id/)的 IPv6_only 的 VPS，本来只搭了一个 [hysteria2](https://github.com/apernet/hysteria)协议的代理，因为只能使用 IPv6 连入，基本处于放置不用的状态，但是看到后来大家都开始抢注册，又舍不得把机器放出，于是一直在续期。

近些天，在这台机器上部署了一些应用，感觉用起来还比较舒服，于是做个记录。

[](#一些常用程序包的安装 "一些常用程序包的安装")一些常用程序包的安装
--------------------------------------

我选择的是 Debian11 的系统，因为免费的 VPS 只有 1500MB 的内存，为了尽可能部署更多的服务，我没有选择安装 Docker。与之相对应，我安装了 Python 和 Node.js 还有 OpenJDK，以便运行目前和之后我需要部署在 VPS 上的服务。

现得知，由于 Hax 目前使用的 OpenVZ 虚拟化技术的原因，docker 相关功能无法使用，所以只能使用手动部署的方式对应用进行部署。

> 前置需求：  
> 因为 Hax 只有 IPv6 网络，甚至于有时 dns 服务器的默认设置是一个 IPv4 地址，根本无法解析，需要先配置使用 DNS64 才能正常联网。配置 DNS64 可以使用此命令：
> 
> BASH
> 
> ```
> echo "nameserver   2a01:4f9:c010:3f02::1" > /etc/resolv.conf
> ```
> 
> 然后再使用 [fscarmen](https://github.com/fscarmen)大佬的 [Warp](https://gitlab.com/fscarmen/warp)脚本：
> 
> BASH
> 
> ```
> wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh
> ```
> 
> 即可获得 [Cloudflare](https://1.1.1.1/)提供的 IPv4 地址，正常访问 IPv4 的网络资源。
> 
> 另外，可以在 [Warp+ bot](https://t.me/generatewarpplusbot)处生成 Warp + 的秘钥，提升 Warp 的网络速度。

### [](#Python "Python")Python

我就是喜欢追新，所以这里我没有直接使用 `apt-get`安装 Python，而是选择了编译安装目前最新的 Release 版本——Python-3.12.1。编译安装并不难，只是略微有些慢：

先安装一些环境依赖：

BASH

```
apt-get update && apt-get install build-essential gdb lcov pkg-config libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev lzma lzma-dev tk-dev uuid-dev zlib1g-dev
```

接着编译安装 Python：

BASH

```
cd /usr/local          #进入/usr/local路径
mkdir -p soft && mkdir -p source          #创建soft、source路径
cd source && wget https://www.python.org/ftp/python/3.12.1/Python-3.12.1.tgz          #进入source路径并下载Python-3.12.1源码
tar -xzvf Python-3.12.1.tgz && rm Python-3.12.1.tgz && cd Python-3.12.1          #解压
./configure --prefix=/usr/local/soft/python3.12 --enable-optimizations          #检测安装环境，配置安装参数，生成供编译用的Makefile
make && make install          #编译安装
```

再创建软链接：

BASH

```
rm -f /usr/bin/python3          #删除原有软链接
ln -s /usr/local/soft/python3.12/bin/python3.12 /usr/bin/python3          #新建软链接python3
ln -s /usr/local/soft/python3.12/bin/pip3.12 /usr/bin/pip          #新建软链接pip
```

检查安装是否成功：

BASH

```
python3 --version
pip --version
```

### [](#Node-js "Node.js")Node.js

同理追新，我选择安装 Node.js-v21.5.0：

BASH

```
curl -fsSL https://deb.nodesource.com/setup_21.x | bash - &&\
apt-get install -y nodejs
```

检测安装是否成功：

BASH

```
node --version
npm --version
```

### [](#OpenJDK "OpenJDK")OpenJDK

用得不多，所以偷个懒，直接 `apt-get`安装：

BASH

```
apt-get install openjdk-17-jdk
```

检测安装是否成功：

BASH

```
java -version
```

> 如果要安装新一点的版本，以 OpenJDK 21 为例：
> 
> BASH
> 
> ```
> mkdir -p /usr/local/soft
> cd /usr/local/soft
> wget https://download.java.net/java/GA/jdk21.0.1/415e3f918a1f4062a0074a2794853d0d/12/GPL/openjdk-21.0.1_linux-x64_bin.tar.gz
> tar -xzvf openjdk-21.0.1_linux-x64_bin.tar.gz
> rm openjdk-21.0.1_linux-x64_bin.tar.gz
> ln -s /usr/local/soft/jdk-21.0.1/bin/java /usr/bin/java
> ln -s /usr/local/soft/jdk-21.0.1/bin/jar /usr/bin/jar
> ```
> 
> 然后修改 `/etc/profile`，在最末添加一些环境变量：
> 
> BASH
> 
> ```
> export JAVA_HOME=/usr/local/soft/jdk-21.0.1
> export PATH=${JAVA_HOME}/bin:$PATH
> ```
> 
> 可以使用 sftp 上传覆盖修改，也可以使用 vim、nano 之类的工具在线修改。  
> 修改完后应用环境变量：
> 
> BASH
> 
> ```
> source /etc/profile
> ```

### [](#PHP "PHP")PHP

这里我选择了安装 PHP8.2，debian11 的 `apt-get`源里只搜得到低版本，同 Node.js 一样，稍作处理即可用 `apt-get`直接安装：

BASH

```
apt-get update && apt-get install lsb-release ca-certificates curl -y
curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg && sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
apt-get update
apt-get install php
```

如果在执行第二条命令时报错：

BASH

```
Traceback (most recent call last):
  File "/usr/bin/lsb_release", line 25, in <module>
    import lsb_release
ModuleNotFoundError: No module named 'lsb_release'
```

则通过这个办法解决：

BASH

```
cp /usr/lib/python3/dist-packages/lsb_release.py /usr/bin/
```

### [](#MySQL "MySQL")MySQL

创建自己的数据库，再也不用到处找免费的数据库白嫖！充分利用 Hax 的 15G 硬盘。安装方法如下：

BASH

```
wget https://repo.mysql.com//mysql-apt-config_0.8.29-1_all.deb
dpkg -i mysql-apt-config_0.8.29-1_all.deb
rm mysql-apt-config_0.8.22-1_all.deb
apt-get update
apt-get install mysql-server
```

MySQL 默认监听 3306 端口，安装过程中会有一些自定义选项，还会要求你设置数据库 root 用户的密码。使用查看版本号的命令检查 MySQL 是否安装正常：

BASH

```
mysql --version
```

### [](#Redis "Redis")Redis

默认监听 6379 端口，主要用于缓存提速，部分应用可用，不详细介绍，只记录安装方法：

BASH

```
curl https://packages.redis.io/gpg | apt-key add -
echo "deb https://packages.redis.io/deb \$(lsb\_release -cs) main" | tee /etc/apt/sources.list.d/redis.list
apt-get update -y && apt-get install redis-server -y
```

连接测试，会进入到 127.0.0.1:6379 的实例中：

BASH

```
redis-cli
```

### [](#pm2 "pm2")pm2

推荐一个很好用的进程管理工具：[pm2](https://github.com/Unitech/pm2)，安装简单，功能强大。具体功能不细说，源仓库介绍很详细，安装方法也很简单，直接用 `npm`就能安装：

BASH

```
npm install pm2 -g
```

常用指令：

BASH

```
pm2 ls          #查看目前pm2管理的所有进程的简易列表
pm2 monit          #查看pm2管理的所有进程的详细信息
```

[](#一些应用的部署 "一些应用的部署")一些应用的部署
-----------------------------

我首先部署的代理在此就不做介绍了，Hysteria2 和 [tuic](https://github.com/EAimTY/tuic)协议目前在 Hax 上的表现都还不错，GitHub 和 Gitlab 上也都有不少一键部署的脚本。如果希望在本地没有 IPv6 的情况下代理到 Hax，则需要选择使用 Cloudflare 隧道的代理方式，上述的 fscarmen 大佬也有提供，这里不再赘述。

我目前已经在 Hax 上部署了 [PandoraNext](https://github.com/pandora-next/deploy)、[TokensTool](https://github.com/Yanyutin753/PandoraNext-TokensTool)、[One-API](https://github.com/songquanpeng/one-api)、[Bingo](https://github.com/weaigc/bingo)四项服务，并通过 [Cloudflared](https://github.com/cloudflare/cloudflared)隧道链接到了自己的域名，通过 Cloudflare 提供的服务，使得没有 IPv6 的网络情况下也可以访问我部署的服务。以上服务均用 `pm2`启动，以便管理，防止崩溃。

### [](#Bingo "Bingo")Bingo

通过源码部署的方式进行部署。将 Bingo 的代码下载到本地，并测试运行：

BASH

```
git clone https://github.com/weaigc/bingo.git          #下载源码到本地
cd bingo
npm install
npm run build          #构筑
npm run start          #运行
```

其默认端口配置在 `server.js`文件的第 7 行，默认为 `3000`。运行成功后可以访问 http://[IPv6]:PORT 查看是否正常运行。  
正常运行的话，使用 `Ctrl+c`停止运行，如果你希望设置内置账号，可以设置环境变量：

BASH

```
echo "BING_HEADER=Your_Header" > .env          #把Your_Header改成转换的HEADER
```

再使用 pm2 启动：

BASH

```
pm2 start npm -- run start
```

至此，Bingo 就部署完毕了，如果需要修改 `BING_HEADER`然后重启应用的话，修改后直接使用 `ps -ef`命令找到原本运行的 Bingo，接着使用 `kill`将其停止即可。pm2 会自启 Bingo。

### [](#One-API "One-API")One-API

因为源仓库提供了可执行文件，所以部署起来非常简单，下载，赋权，运行即可：

BASH

```
wget -O one-api https://github.com/songquanpeng/one-api/releases/download/v0.5.10/one-api          #下载可执行文件并重命名为one-api
chmod 777 one-api          #赋权
./one-api --port 1333 --log-dir ./logs          #在1333端口运行One-API并将日志输出到当前路径下的logs路径下
```

接着检查一下 1333 端口上 one-api 是否正常运行，运行正常的话，使用 `Ctrl+c`停止运行。再使用 pm2 启动 One-API：

BASH

```
pm2 start ./one-api -- --port 1333 --log-dir ./logs
```

至此，One-API 部署完毕。如果需要升级，与 Bingo 同理，下载覆盖后，使用 kill 停止原本的任务即可，pm2 会自启 One-API。

### [](#PandoraNext "PandoraNext")PandoraNext

与 One-API 相同，PandoraNext 提供了可执行文件，部署简单。需要注意的是，PandoraNext 并不是一个服务端，只是一个客户端，而且并不开源，但是其功能非常强大，且免费使用。以下是部署步骤：

BASH

```
wget https://github.com/pandora-next/deploy/releases/download/v0.6.1/PandoraNext-v0.6.1-linux-amd64-e1cae28.tar.gz
tar -xzvf PandoraNext-v0.6.1-linux-amd64-e1cae28.tar.gz --strip-components=1
chmod 777 PandoraNext
```

接着修改 `config.json`文件，填入自己的配置，再启动 PandoraNext 进行测试运行：

BASH

```
./PandoraNext
```

接着检查一下你在 `config.json`中设置的端口上 PandoraNext 是否正常运行，运行正常的话，使用 `Ctrl+c`停止运行。再使用 pm2 启动 PandoraNext：

BASH

```
pm2 start ./PandoraNext
```

至此，PandoraNext 部署完毕。如果需要升级，与 Bingo 同理，下载覆盖后，使用 kill 停止原本的任务即可，pm2 会自启 PandoraNext。

> 需要注意的是，PandoraNext 的 License 是绑定 IPv4 的，实际上 Hax 的 IPv6_only 的 VPS 都在相近的机房，其套用 Warp 获得的 IPv4 地址数量非常有限，大多数都是共用相同的 IPv4 地址。按照 PandoraNext 的规则来讲，就是先到先得，部署晚的有可能会无法绑定 License。

### [](#TokensTool "TokensTool")TokensTool

TokensTool 是配合 PandoraNext 使用的 tokens 管理工具，能把刷新 tokens、推流 one-api 等工作自动化进行，而且配备了 UI 页面，使用简单方便。目前 TokensTool 还在不断更新中，如今已经支持了 cocopilot 转 OpenAI API 的功能，越来越多的功能将会被添加进去。TokensTool 是用 Java 写的，提供了打包好的 Jar 包，只需要下载 Jar 包运行即可：

BASH

```
wget -O tokenstool.jar https://github.com/Yanyutin753/PandoraNext-TokensTool/releases/download/v0.5.9/pandoraNext-0.5.9-SNAPSHOT.jar          #下载jar包并重命名为tokenstool.jar
java -jar tokenstool.jar --server.port=8081 --deployWay=releases --deployPosition=/root/app --hotReload=true --pandora_Ip=127.0.0.1          #测试运行
```

接着检查一下 8081 端口上 TokensTool 是否正常运行，运行正常的话，使用 `Ctrl+c`停止运行。再使用 pm2 启动 TokensTool：

BASH

```
pm2 start java -- -jar tokenstool.jar --server.port=8081 --deployWay=releases --deployPosition=/root/app --hotReload=true --pandora_Ip=127.0.0.1
```

至此，TokensTool 部署完毕。如果需要升级，与 Bingo 同理，下载覆盖后，使用 kill 停止原本的任务即可，pm2 会自启 TokensTool。

### [](#Cloudflared "Cloudflared")Cloudflared

为了便利的给上述服务添加域名，并使得没有 IPv6 的网络环境下也能访问上述服务，我使用了 Cloudflare 提供的隧道，其使用方法也是非常简单便利：

BASH

```
wget -O cloudflared https://github.com/cloudflare/cloudflared/releases/download/2023.10.0/cloudflared-linux-amd64          #下载Cloudflared客户端
chmod 777 cloudflared          #赋权
./cloudflared tunnel --edge-ip-version auto --protocol http2 --heartbeat-interval 10s run --token ARGO_TOKEN          #测试运行，我选择使用http2协议，ARGO_TOKEN应该替换为自己创建的隧道的那一串ey开头的TOKEN
```

在 Cloudflare 的[面板](https://one.dash.cloudflare.com/)中看到隧道上线，则运行正常。再使用 `Ctrl+c`停止运行，并使用 pm2 启动 Cloudflared：

BASH

```
pm2 start ./cloudflared -- tunnel --edge-ip-version auto --protocol http2 --heartbeat-interval 10s run --token ARGO_TOKEN
```

至此，Cloudflared 部署完毕。如果需要升级，与 Bingo 同理，下载覆盖后，使用 kill 停止原本的任务即可，pm2 会自启 Cloudflared。

在 Cloudflare 的面板中给上述服务添加对应域名即可，比如 3000 端口的 bingo，只要添加 `HTTP`协议的地址 `localhost:3000`即可。

[](#一些我没有部署但可以部署的应用 "一些我没有部署但可以部署的应用")一些我没有部署但可以部署的应用
-----------------------------------------------------

在 Warp 和 Cloudflared 的加持下，Hax 的 IPv6_only 的 VPS 的使用基本上已经和有 IPv4 的服务器无异，仅仅在需要进行 SSH 或者 SCP 连接的时候必须要求本地有 IPv6 网络。而实际上，通过使用 [File Browser](https://github.com/filebrowser/filebrowser)和 [TTYD](https://github.com/tsl0922/ttyd)并使用 Cloudflared 隧道穿透出来，就变相实现了这两个协议所实现的功能在没有 IPv6 网络的情况下的使用。

上述的两个仓库都是单个可执行文件即可实现部署，这里不再赘述部署方式，仅仅记录运行命令，以便查找：

BASH

```
# ttyd运行指令
./ttyd -p 21022 -c admin:password -W bash          #在21022端口运行ttyd，用户名为admin，密码为password，连接的命令行解释器为bash
pm2 start ./ttyd -- -p 21022 -c admin:password -W bash          #使用pm2运行
```

BASH

```
# filebrowser运行指令
./filebrowser -p 21021          #在21021端口运行filebrowser，默认用户名和密码都为admin，登录后可以修改密码
pm2 start ./filebrowser -- -p 21021          #使用pm2运行
```

此外，各种 QQ 机器人、Blog 框架（如 [jar4halo](https://github.com/Lu7fer/Jar4Halo)）、文件目录程序（如 [alist](https://github.com/alist-org/alist)）、图床程序等等。因为此前已经把 Python、Node.js、OpenJDK、PHP 都安装好了，主流应用的环境问题都很好解决，多数只需要按照文档部署即可。

> BASH
> 
> ```
> # jar4halo需要设置环境变量运行
> export HALO_WORK_DIR="/root/halo/.halo2"          #halo的工作目录设置为/root/halo/.halo2
> export HALO_EXTERNAL_URL="https://Your_Domain"          #设置halo的对外域名
> java -jar -Duser.timezone=Asia/Shanghai halo.jar          #使用UTC+8时区
> ```

[](#文件的备份与转移 "文件的备份与转移")文件的备份与转移
--------------------------------

如果你需要将服务器上的文件快速地下载到本地，亦或者是传输到另一台服务器上，可以使用 [ShareList](https://github.com/reruin/sharelist)。  
这是一个一键脚本，用以在当前路径直接启动 sharelist，并且把当前路径挂载到 sharelist 中：

BASH

```
bash <(curl -s https://raw.githubusercontent.com/k0baya/sharelist_repl/main/hax/share.sh)
```

上述命令只能在 AMD64 架构的 Linux 机器上使用。sharelist 固定运行在 33001 端口，因为其监听的是`127.0.0.1`所以无法使用`ip:port`进行访问，你可以选择用 nginx 或者直接用 Cloudflared 绑定 33001 端口到一个域名进行访问。进入 sharelist 的页面后，你可以直接把打包好的文件下载到本地或者在另外一台机器使用`wget -O 文件名 '链接'`的命令进行下载。

> 另外把打包和解压的命令做个备份，我喜欢使用 tar 命令，这里只记录 tar：
> 
> ** 打包：`tar -czvf 文件名.tar.gz 路径`**，比如打包整个`/root/app`路径，命名为`archive.tar.gz`，即为`tar -czvf archive.tar.gz /root/app`
> 
> ** 解压：`tar -xzvf 文件名`**，比如解压刚刚打包的`archive.tar.gz`即为`tar -xzvf archive.tar.gz`

未完待续……