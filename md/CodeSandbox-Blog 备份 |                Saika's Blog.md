> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.rappit.site](https://blog.rappit.site/2024/01/03/codesandbox-backup/)

> A Reading focusing Blog

[](#在CodeSandBox上部署docker并使用Cloudflare-Tunnel打通自己的域名——以青龙面板为例 "在CodeSandBox上部署docker并使用Cloudflare Tunnel打通自己的域名——以青龙面板为例")在 [CodeSandBox](https://codesandbox.io/)上部署 docker 并使用 Cloudflare Tunnel 打通自己的域名——以[青龙面板](https://github.com/whyour/qinglong)为例
=========================================================================================================================================================================================================================================================

[](#前言 "前言")前言
--------------

长期以来白嫖 [Replit](https://replit.com/~)已经成为习惯，突然 Replit 的政策改变让人猝不及防。

虽然此前就已经知道了 CodeSandBox 这个平台，但苦于其公开性质，以及不可更换的会直接暴露公开容器的域名分配，一直以来对 CodeSandBox 的使用也是离不开 Replit 的——在 CodeSandBox 部署好服务之后，在 Replit 部署 NGINX 服务反代 CodeSandBox 提供的域名，一定程度上避免 CodeSandBox 容器的暴露。

那么 Replit 即将无法白嫖的现状下，怎么快乐的使用 CodeSandBox 呢？第一时间我想到的是 Cloudflare 的 Workers。但是 Workers 对于免费用户来讲额度太少，就此浪费掉未免也太可惜，那怎么办呢？

虽说家中的软路由一直以来都使用 Cloudflared 打通 Tunnel 进行穿透，但是还得是 fscarman2 的各种 PaaS 平台搭建代理的仓库给了我灵感，可以在 PaaS 平台使用 Cloudflared，像家里的软路由内网穿透一样，使用自己的域名。

经过我反复尝试，总算找到了在 CodeSandBox 上能够比较稳定搭建各种服务的办法。如果你会操作，也可以自行构筑各种镜像进行部署，只需要在 `docker-compose.yaml`文件末添加 Cloudflared 部分即可。

这里就以青龙面板为例，讲述一下 CodeSandBox 的白嫖方法。

[](#部署过程 "部署过程")部署过程
--------------------

### [](#注册CodeSandBox "注册CodeSandBox")注册 CodeSandBox

CodeSandBox 支持 Google、Github 等多种方式登录，任意方式注册登录即可，这里不再赘述。

### [](#编写docker-compose文件 "编写docker-compose文件")编写 docker-compose 文件

以青龙面板为例，在[这里](https://github.com/whyour/qinglong/blob/develop/docker/docker-compose.yml)可以找到其 docker-compose 文件：

折叠代码块 YAML 复制代码

```
version: '2'
services:
  web:
    # alpine 基础镜像版本
    image: whyour/qinglong:latest
    # debian-slim 基础镜像版本
    # image: whyour/qinglong:debian  
    volumes:
      - ./data:/ql/data
    ports:
      - "0.0.0.0:5700:5700"
    environment:
      # 部署路径非必须，以斜杠开头和结尾，比如 /test/
      QlBaseUrl: '/'
    restart: unless-stopped
```

稍作修改：

折叠代码块 YAML 复制代码

```
version: '2'
services:
    qinglong:
      image: whyour/qinglong:latest
      volumes:
        - /project/sandbox/ql/data:/ql/data
      ports:
       - "0.0.0.0:5700:5700"
      environment:
        QlBaseUrl: '/'
      restart: always
```

主要修改了其 `volumes`的部分，因为 CodeSandBox 的文件目录在 `/project/sandbox/`下，所以我在此路径下创建 `./ql/data`来存储青龙面板的数据，实现数据持久化。

而 Cloudflared 部分，我是这样写的：

折叠代码块 YAML 复制代码

```
cloudflared:
      restart: always
      network_mode: host
      environment:
          - TZ=Asia/Shanghai
      command: tunnel --edge-ip-version auto --protocol quic --heartbeat-interval 10s run --token [ARGO_TOKEN]
      container_name: cloudflared
      image: cloudflare/cloudflared:latest
```

其中的 `[ARGO_TOKEN]`要替换成自己的 token，后文将附上详细教程。

所以整个 `docker-compose.yaml`的内容如下：

折叠代码块 YAML 复制代码

```
version: '2'
services:
    qinglong:
      image: whyour/qinglong:latest
      volumes:
        - /project/sandbox/ql/data:/ql/data
      ports:
       - "0.0.0.0:5700:5700"
      environment:
        QlBaseUrl: '/'
      restart: always

    cloudflared:
      restart: always
      network_mode: host
      environment:
          - TZ=Asia/Shanghai
      command: tunnel --edge-ip-version auto --protocol quic --heartbeat-interval 10s run --token [ARGO_TOKEN]
      container_name: cloudflared
      image: cloudflare/cloudflared:latest
```

### [](#获取-ARGO-TOKEN "获取[ARGO_TOKEN]")获取 [ARGO_TOKEN]

详见[群晖套件：Cloudflare Tunnel 内网穿透中文教程 支持 DSM6、7](https://imnks.com/5984.html)。

此处需要的 [ARGO_TOKEN] 就是 ey 开头的那一串。

然后在 Cloudflare 中设置好相应的域名和本地端口对应，一般协议用 http 即可。像此处，按照我的 `docker-compose.yaml`来看，我应该映射 5700 端口，所以我填的是 `0.0.0.0:5700`。

### [](#创建Devbox "创建Devbox")创建 Devbox

在 CodeSandBox 中进入 dashboard，然后点击”Create a Devbox”

选择 docker

进入 Devbox 之后，在左侧的文件管理 EXPLORER 中可以看到有 `.codesandbox`和 `.devcontainer`两个文件夹

先修改 `.codesandbox`中的 `tasks.json`文件，在 `setupTasks`中添加部署命令：

折叠代码块 JSON 复制代码

```
"setupTasks": [
    {
      "name": "Deploy",
      "command": "cd /project/sandbox/.devcontainer/ && docker compose up -d"
    }
  ],
```

然后在 `.devcontainer`文件夹下新建 `docker-compose.yaml`文件，把上述的 `docker-compose.yaml`内容全部复制进去。

最后点击左上角的方框打开菜单，选择”Restart Devbox”，等待容器重启后，你就可以看到青龙运行好了。再试一下刚刚在 Cloudflare 中映射的域名，也可以用了。

### [](#一些问题 "一些问题")一些问题

不知道为什么，当我使用 http2 模式运行 Cloudflared 时，隧道断连后并不会重新连接，auto 模式在 quic 模式断连后自动切换 http2 模式，再断连又会出现 http2 模式一样的问题。所以我只能指定其使用 quic 模式进行运行。原因未知，希望有人能找到原因。

然后，必须保持容器活跃，Cloudflared 掉线后才会重连，为保证其活跃，得使用网站监控器监控 CodeSandBox 自带的映射域名，形同 `https://t6a4m-8080.csb.app/`的那个。至于网站监控器，[Uptime-Kuma](https://github.com/louislam/uptime-kuma)、[cron-job](https://cron-job.org/en/)、[UptimeRobot](https://uptimerobot.com/)之类的都行。

注意，有些 PaaS 平台（比如 Glitch）已经禁止了自身服务器 ip、公用网站监控器 ip 进行访问，所以上述的 cron-job 和 UptimeRobot 是否能行请自行测试，更加建议自己搭建 Uptime-Kuma。

[](#后话 "后话")后话
--------------

在 CodeSandBox 上我已经尝试使用此方法部署了 [Alist](https://github.com/alist-org/alist)、[qinglong](https://github.com/whyour/qinglong)、[Halo](https://www.halo.run/)（本站就是）、[Pandora_Next](https://github.com/pandora-next/deploy)以及自己构筑的个别镜像等多个应用，尽数成功。可玩性还是很高的。

但是不得不提醒，这种方法只是避免了从域名上找到自己的容器而已，并不能确保自己的数据不会泄露，毕竟使用的是公共容器，请注意信息安全，不要存储私密信息，否则后果自负。

另，宣传一下 QQ 群：[受虐滑稽](https://jq.qq.com/?_wv=1027&k=qssjFvAs)

[](#一些可以在CodeSandBox上配合Cloudflared使用的docker-compose "一些可以在CodeSandBox上配合Cloudflared使用的docker-compose")一些可以在 CodeSandBox 上配合 Cloudflared 使用的 docker-compose
==========================================================================================================================================================

> **以下任何一个配置，如果不需要 Cloudflared 打隧道，就把 cloudflared 部分删掉。**

> **另外，除了 Cloudflared 打隧道，现在有新的使用自己的域名的方法，详见仓库：[k0baya/reserve-vercel](https://github.com/k0baya/reserve-vercel)。**
> 
> 现在已经将 Cloudflared 部分从各配置中删除，如有需要请自行添加：
> 
> 折叠代码块 复制代码
> 
> ```
> version: '3.3'
> services:
>    cloudflared:
>      restart: always
>      network_mode: host
>      environment:
>          - TZ=Asia/Shanghai
>      command: tunnel --edge-ip-version auto --protocol quic --heartbeat-interval 10s run --token [ARGO_TOKEN]
>      container_name: cloudflared
>      image: cloudflare/cloudflared:latest
> ```

[](#Qinglong "Qinglong")[Qinglong](https://github.com/whyour/qinglong)
----------------------------------------------------------------------

> 映射 5700 端口

折叠代码块 YAML 复制代码

```
version: '2'
services:
    qinglong:
      image: whyour/qinglong:latest
      volumes:
        - /project/sandbox/ql/data:/ql/data
      ports:
       - "0.0.0.0:5700:5700"
      environment:
        QlBaseUrl: '/'
      restart: always
```

[](#Alist "Alist")[Alist](https://github.com/alist-org/alist)
-------------------------------------------------------------

> 映射 5244 端口
> 
> 如果需要使用 Aria2 实现离线下载，把第 14 行的 `image`指定的镜像更改为 `xhofe/alist-aria2:main`即可

折叠代码块 YAML 复制代码

```
version: '3.3'
services:
    alist:
        restart: always
        volumes:
            - '/project/sandbox/alist:/opt/alist/data'
        ports:
            - '5244:5244'
        environment:
            - PUID=0
            - PGID=0
            - UMASK=022
        container_name: alist
        image: 'xhofe/alist:latest'
```

[](#Halo "Halo")[Halo](https://www.halo.run/)
---------------------------------------------

> 映射 8090 端口

折叠代码块 YAML 复制代码

```
version: "3"
services:
  halo:
    image: halohub/halo:2.11
    container_name: halo
    restart: on-failure:3
    depends_on:
      halodb:
        condition: service_healthy
    networks:
      halo_network:
    volumes:
      - /project/sandbox/data/halo2:/root/.halo2
    ports:
      - "8090:8090"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8090/actuator/health/readiness"]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 30s      
    command:
      - --spring.r2dbc.url=r2dbc:pool:postgresql://halodb/halo
      - --spring.r2dbc.username=halo
      # PostgreSQL 的密码，请保证与下方 POSTGRES_PASSWORD 的变量值一致。
      - --spring.r2dbc.password=12345678
      - --spring.sql.init.platform=postgresql
      # 外部访问地址，请根据实际需要修改
      - --halo.external-url=http://localhost:8090/
  
  halodb:
    image: postgres:15.4
    container_name: halodb
    restart: on-failure:3
    networks:
      halo_network:
    volumes:
      - /project/sandbox/data/db:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
    environment:
      - POSTGRES_PASSWORD=12345678
      - POSTGRES_USER=halo
      - POSTGRES_DB=halo
      - PGUSER=halo

networks:
  halo_network:
```

[](#Microsoft-365-E5-RenewX "Microsoft_365_E5_RenewX")[Microsoft_365_E5_RenewX](https://hub.docker.com/r/gladtbam/ms365_e5_renewx)
----------------------------------------------------------------------------------------------------------------------------------

> 映射端口 1066

折叠代码块 YAML 复制代码

```
version: '3.5'
services:
  renewx:
    image: gladtbam/ms365_e5_renewx:latest
    container_name: renewx
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /project/sandbox/E5RenewX/Deploy:/renewx/Deploy
      - /project/sandbox/E5RenewX/Appdata:/renewx/appdata
    ports:
      - "1066:1066"
    restart: unless-stopped
```

> 需要新建 `Config.xml`文件放在 `Deploy`内，默认的 `Config.xml`文件内容如下：

折叠代码块 TXT 复制代码

```
<?xml version="1.0" encoding="utf-8" ?>
<Configuration>
	<!--站点服务器基本配置-->
	<Serivce>
		<!--服务访问端口-->
		<Port>1066</Port>
		<!--管理员密码(管理员登录路由/Admin/Login) 重要：首次启动前必须更改-->
		<LoginPassword>12345678</LoginPassword>
		<!--是否启用内核多线程支持-->
		<CoreMultiThread>true</CoreMultiThread>
		<!--网站备案（选填）-->
		<ICP>
			<!--备案显示文本-->
			<Text></Text>
			<!--备案管理查询机构跳转链接-->
			<Link>https://beian.miit.gov.cn</Link>
		</ICP>
		<!--Bootstrap CDN 若要更改请务必使用bootstrap@5.1.3版本（选填）-->
		<CDN>
			<!--Bootstrap CSS文件CDN bootstrap.min.css-->
			<CSS>https://cdn.staticfile.org/bootstrap/5.1.3/css/bootstrap.min.css</CSS>
			<!--Bootstrap JS文件CDN bootstrap.bundle.min.js-->
			<JS>https://cdn.staticfile.org/bootstrap/5.1.3/js/bootstrap.bundle.min.js</JS>
		</CDN>
	</Serivce>
	<!--站点Kestrel服务器HTTPS配置 （只支持IIS证书类型 即PFX格式的证书）-->
	<HTTPS>
		<!--Kestrel是否启用HTTPS(SSL加密传输)-->
		<Enable>false</Enable>
		<!--SSL证书文件名 (需要将PFX格式的SSL证书放置于该配置文件的同级目录Deploy文件夹下) 如e5.sundayrx.net.pfx-->
		<!--不填则默认使用Dev localhost 本地证书-->
		<Certificate></Certificate>
		<!--SSL证书密钥(PFX证书的访问密钥)-->
		<Password></Password>
	</HTTPS>
	<!--共享站点配置,不共享可无视以下内容 (若要共享站点 请自备以下所需的配置信息 且配置中HTTPS必须启用)-->
	<ShareSite>
		<!--是否启用站点共享-->
		<Enable>false</Enable>
		<!--SMTP邮件发送支持-->
		<SMTP>
			<!--发件邮箱-->
			<Email></Email>
			<!--邮箱密钥-->
			<Password></Password>
			<!--SMTP服务器地址-->
			<Host></Host>
			<!--SMTP服务器端口-->
			<Port>587</Port>
			<!--SMTP服务器是否使用SSL传输-->
			<EnableSSL>true</EnableSSL>
		</SMTP>
		<!--第三方OAuth登录支持(至少启用以下一种OAuth否则其他用户无法注册)-->
		<OAuth>
			<!--微软登录授权-->
			<Microsoft>
				<!--是否启用该OAuth-->
				<Enable>true</Enable>
				<!--应用程序Id-->
				<ClientId></ClientId>
				<!--应用程序访问机密-->
				<ClientSecret></ClientSecret>
			</Microsoft>
			<!--GitHub登录授权-->
			<Github>
				<!--是否启用该OAuth-->
				<Enable>true</Enable>
				<!--应用程序Id-->
				<ClientId></ClientId>
				<!--应用程序访问机密-->
				<ClientSecret></ClientSecret>
			</Github>
		</OAuth>
		<!--站点系统设置-->
		<System>
			<!--站点启动后默认是否允许用户注册 建议为false-->
			<AllowRegister>false</AllowRegister>
			<!--站点启动后默认公告（换行符请使用 
 进行换行）-->
			<Notice></Notice>
			<!--站点运营者-->
			<Master></Master>
			<!--站点运营者推广链接-->
			<MasterLink></MasterLink>
			<!--站点新用户默认配额数-->
			<DefaultQuota>1</DefaultQuota>
			<!--站点自动特赦时间间隔 （单位：天 至少30天）-->
			<AutoSpecialPardonInterval>30</AutoSpecialPardonInterval>
		</System>
	</ShareSite>
</Configuration>
```

[](#Pandora-Next "Pandora-Next")[Pandora-Next](https://github.com/pandora-next/deploy)
--------------------------------------------------------------------------------------

> 映射端口 8181

折叠代码块 YAML 复制代码

```
version: '3.3'
services:
    pandora-next:
        image: pengzhile/pandora-next
        container_name: PandoraNext
        network_mode: bridge
        restart: always
        ports:
            - "8181:8181"
        volumes:
            - /project/sandbox/pandora-next/data:/data
            - /project/sandbox/pandora-next/sessions:/root/.cache/PandoraNext
```

[](#Uptime-Kuma "Uptime-Kuma")[Uptime-Kuma](https://github.com/louislam/uptime-kuma)
------------------------------------------------------------------------------------

> 映射端口 3001

折叠代码块 YAML 复制代码

```
version: '3.8'
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - /project/sandbox/uptimekuma:/app/data
    ports:
      - "3001:3001"  # <Host Port>:<Container Port>
    restart: always
```

[](#KodBox "KodBox")[KodBox](https://kodcloud.com/)
---------------------------------------------------

> 映射端口 8080

折叠代码块 YAML 复制代码

```
version: '3.5'
services:
  db:
    image: mariadb
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - "/project/sandbox/db:/var/lib/mysql"   
    environment:
      - "TZ=Asia/Shanghai"
      - "MYSQL_ROOT_PASSWORD=jiehdo!25165n"
      - "MYSQL_DATABASE=kodbox"
      - "MYSQL_USER=bodbox"
      - "MYSQL_PASSWORD=jiehdo!25165n"
    restart: always
  
  app:
    image: kodcloud/kodbox
    ports:
      - 8080:80                
    links:
      - db
      - redis
    volumes:
      - "/project/sandbox/site:/var/www/html"  
    restart: always

  redis:
    image: redis:alpine
    environment:
      - "TZ=Asia/Shanghai"
    restart: always
```

> 如果照搬上面的设置，初始化时数据库和 redis 的设置应该如图填写
> 
> 数据库类型：MySQL  
> 服务器：db  
> 用户名：root  
> 密码：jiehdo!25165n  
> 数据库：kodbox  
> 存储引擎：InnoDB  
> 系统缓存类型：Redis  
> 服务器：redis

[](#Zfile "Zfile")[Zfile](https://zfile.vip/)
---------------------------------------------

> 映射端口 8080

折叠代码块 YAML 复制代码

```
version: '3.3'
services:
    zfile:
        container_name: zfile
        restart: always
        ports:
            - '8080:8080'
        volumes:
            - '/project/sandbox/zfile/db:/root/.zfile-v4/db'
            - '/project/sandbox/zfile/logs:/root/.zfile-v4/logs'
            - '/project/sandbox/zfile/file:/data/file'
        image: zhaojun1998/zfile
```

[](#PanIndex "PanIndex")[PanIndex](https://github.com/px-org/PanIndex)
----------------------------------------------------------------------

> 映射端口 5238

折叠代码块 YAML 复制代码

```
version: '3.3'
services:
    panindex:
        container_name: panindex
        restart: always
        ports:
            - '5238:5238'
        volumes:
            - '/project/sandbox/PanIndex/data:/app/data'
        environment:
            - "PORT=5238"
        image: iicm/pan-index:latest
```

[](#ShareList "ShareList")[ShareList](https://github.com/reruin/sharelist)
--------------------------------------------------------------------------

> 映射端口 33001

折叠代码块 YAML 复制代码

```
version: '3.3'
services:
    sharelist:
        container_name: sharelist
        restart: always
        ports:
            - '33001:33001'
        volumes:
            - '/project/sandbox/sharelist:/sharelist/cache'
        image: reruin/sharelist:next
```

[](#Cloudreve "Cloudreve")[Cloudreve](https://cloudreve.org/)
-------------------------------------------------------------

> 映射端口 5212
> 
> 进入容器需要先手动创建几个文件，在 Terminal 中输入以下指令即可：
> 
> 折叠代码块 SHELL 复制代码
> 
> ```
> mkdir -p cloudreve && cd cloudreve \
> && mkdir -vp cloudreve/{uploads,avatar} \
> && touch cloudreve/conf.ini \
> && touch cloudreve/cloudreve.db \
> && mkdir -p aria2/config \
> && mkdir -p data/aria2 \
> && chmod -R 777 data/aria2
> ```

折叠代码块 YAML 复制代码

```
version: "3.8"
services:
  cloudreve:
    container_name: cloudreve
    image: cloudreve/cloudreve:latest
    restart: unless-stopped
    ports:
      - "5212:5212"
    volumes:
      - /project/sandbox/cloudreve/data:/data
      - /project/sandbox/cloudreve/cloudreve/uploads:/cloudreve/uploads
      - /project/sandbox/cloudreve/cloudreve/conf.ini:/cloudreve/conf.ini
      - /project/sandbox/cloudreve/cloudreve/cloudreve.db:/cloudreve/cloudreve.db
      - /project/sandbox/cloudreve/cloudreve/avatar:/cloudreve/avatar
    depends_on:
      - aria2
  aria2:
    container_name: aria2
    image: p3terx/aria2-pro
    restart: unless-stopped
    environment:
      - RPC_SECRET=jif1568dw87
      - RPC_PORT=6800
    volumes:
      - /project/sandbox/cloudreve/aria2/config:/config
      - /project/sandbox/cloudreve/data:/data
```

> Aria2 的 token 默认为 `jif1568dw87`，如有需要自行修改
> 
> 初始账号和密码，请在 Terminal 中输入命令 `docker logs cloudreve`后在日志中查找。

[](#Typecho "Typecho")[Typecho](https://github.com/typecho/typecho)
-------------------------------------------------------------------

> 映射端口 8080

折叠代码块 YAML 复制代码

```
version: '3.7'
services:
  typecho:
    image: joyqi/typecho:nightly-php7.4-apache
    container_name: typecho-server
    restart: always
    environment:
      - TIMEZONE=Asia/Shanghai
    ports:
      - 8080:80
    volumes:
      - /project/sandbox/typecho:/app/usr
    depends_on:
      - mariadb

  mariadb:
    image: mariadb
    container_name: mariadb
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - "/project/sandbox/db:/var/lib/mysql"   
    environment:
      - "TZ=Asia/Shanghai"
      - "MYSQL_ROOT_PASSWORD=jiehdo!25165n"
      - "MYSQL_DATABASE=typecho"
      - "MYSQL_USER=typecho"
      - "MYSQL_PASSWORD=jiehdo!25165n"
    restart: always
```

> 如果照搬此 docker-compose.yaml，初始设置数据库时应该如此填写：
> 
> 数据库适配器：Mysql 原生函数适配器  
> 数据库前缀：typecho_  
> 数据库地址：mariadb  
> 数据库用户名：root  
> 数据库密码：jiehdo!25165n  
> 数据库名：typecho

[](#Baota-Panel "Baota_Panel")[Baota_Panel](https://hub.docker.com/r/gettionhub/baota-docker)
---------------------------------------------------------------------------------------------

> 端口 8888 为面板，2022 为 SSH 端口，2021 为 FTP 端口，2080 和 2443 为网页服务预留端口，2888 是官方给的，不知道作用。

折叠代码块 YAML 复制代码

```
version: '3'
services:
  baota:
    image: gettionhub/baota-docker:ltd
    container_name: baota
    volumes:
      - /project/sandbox/www/website_data:/www/wwwroot 
      - /project/sandbox/www/mysql_data:/www/server/data
      - /project/sandbox/www/vhost:/www/server/panel/vhost 
    ports:
      - "8888:8888"
      - "2022:22"
      - "2021:21"
      - "2443:443"
      - "2080:80"
      - "2888:888"
    restart: unless-stopped
```

> 面板入口为 8888 端口的那个网址后面加上 `/baota`，形同 `https://t6a4m-8888.csb.app/baota`。
> 
> 初始用户名、密码都为 baota
> 
> 容器内 root 用户的 ssh 密码也是 baota

[](#WordPress "WordPress")[WordPress](https://wordpress.org/)
-------------------------------------------------------------

> 映射端口 8080

折叠代码块 YAML 复制代码

```
version: '3.1'
services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - /project/sandbox/wordpree/app:/var/www/html

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - /project/sandbox/wordpree/db:/var/lib/mysql
```

[](#Memos "Memos")[Memos](https://www.usememos.com/)
----------------------------------------------------

> 映射端口 5230

折叠代码块 YAML 复制代码

```
version: "3.0"
services:
  memos:
    image: neosmemo/memos:latest
    container_name: memos
    volumes:
      - /project/sandbox/memos/:/var/opt/memos
    ports:
      - 5230:5230
```

[](#Ghost "Ghost")[Ghost](https://ghost.org/)
---------------------------------------------

> 映射端口 8080
> 
> 首次启动会反复重启几次等待数据库创建文件，是正常的。

折叠代码块 YAML 复制代码

```
version: '3.1'
services:
  ghost:
    image: ghost:4-alpine
    restart: always
    ports:
      - 8080:2368
    environment:
      database__client: mysql
      database__connection__host: db
      database__connection__user: root
      database__connection__password: example
      database__connection__database: ghost
      url: http://localhost:8080
    volumes:
      - /project/sandbox/ghost/app:/var/lib/ghost/content

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
    volumes:
      - /project/sandbox/ghost/db:/var/lib/mysql
```

[](#NGINX-ui "NGINX-ui")[NGINX-ui](https://nginxui.com/)
--------------------------------------------------------

> 面板端口为 8080。网页服务端口预留为 8443。

折叠代码块 YAML 复制代码

```
version: '3.3'
services:
    nginx-ui:
        stdin_open: true
        tty: true
        container_name: nginx-ui
        restart: always
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - '/project/sandbox/appdata/nginx:/etc/nginx'
            - '/project/sandbox/appdata/nginx-ui:/etc/nginx-ui'
            - '/project/sandbox/www:/var/www'
        ports:
            - 8080:80
            - 8443:443
        image: 'uozi/nginx-ui:latest'
```

[](#继续补充ing…… "继续补充ing……")继续补充 ing……
====================================

[](#在CodeSandBox中部署其他应用——以小雅为例 "在CodeSandBox中部署其他应用——以小雅为例")在 CodeSandBox 中部署其他应用——以[小雅](http://alist.xiaoya.pro/)为例
====================================================================================================================

[](#对小雅容器的一些分析 "对小雅容器的一些分析")对小雅容器的一些分析
--------------------------------------

相信部署过小雅的朋友都一直苦恼于小雅的更新频繁，以及阿里网盘更新后的容量限制，给日常观影带来了很不好的体验。随着 xiaoyakeeper 的诞生，才迎来了转机——自动更新容器 + 自动清理转存文件，解决了小雅日常使用的痛点。

但是，不论是小雅本身还是 xiaoyakeeper，其部署方式都是一键式的脚本，而且部署方式限制的死死的——指定了端口、挂载卷等应该容许自定义的参数，使得其难以部署在各种 PaaS 平台上。

此前我曾有过最简单的把小雅部署在 PaaS 平台上的方法构思——基于小雅原镜像进行修改。直接查看[小雅的一键部署脚本](http://docker.xiaoya.pro/update_new.sh)不难发现，是通过提前创建 `/etc/xiaoya`路径，再把需要使用到的各项 token、id 等参数储存为 TXT 文本文件，放入 `/etc/xiaoya`后，使用 `volume`挂载进容器的 `/data`路径进行读取的。所以只要在原有镜像上进行修改，准备好自己的这些 token、id 后，将其全部复制到小雅镜像的 / data 路径, 就应当可以在不使用 volume 挂载的情况下直接部署一个能够正常使用的小雅容器。

但是，首先这个方案比起一键脚本操作复杂太多，而且在我仔细查看了 [xiaoyakeeper 的脚本](https://xiaoyahelper.zengge99.eu.org/aliyun_clear.sh)后，我发现这样操作会使得 xiaoyakeeper 无法读取 docker 命令的 `-v`项，就需要对 xiaoyakeeper 的脚本进行修改了。但是 xiaoyakeeper 的脚本又自带自动更新功能，如果一个更新覆盖了我的修改，又不得偿失；如果删掉自动更新的功能，那么如果阿里网盘的政策出现了改变，还得重新手动修改最新的 xiaoyakeeper 脚本进行替换，难以做到一劳永逸。

通过观察 xiaoyakeeper 的脚本内容，其更新小雅容器的方法并不是运行小雅的一键更新脚本，而是读取当前小雅容器的各项参数之后，重新进行 `docker pull`、`docker run`，也就是说，小雅容器被安装好之后，通过 xiaoyakeeper 进行更新就能完全摆脱小雅一键脚本。而且，如果小雅容器没有被启动的话，xiaoyakeeper 运行之后就会直接启动小雅容器。

[](#对CodeSandBox的分析 "对CodeSandBox的分析")对 CodeSandBox 的分析
-------------------------------------------------------

摸索暂时比较少，Devbox（Docker）大概理解为一个 `Ubuntu`为底的，`zsh`为默认 shell 执行器的，有 root 权限的，预装了 Docker 的虚拟机。在 `/project/sandbox/`路径下的文件，即便重启也不会丢失。

[](#在CodeSandBox上部署小雅、xiaoyakeeper、Cloudflared "在CodeSandBox上部署小雅、xiaoyakeeper、Cloudflared")在 CodeSandBox 上部署小雅、xiaoyakeeper、Cloudflared
----------------------------------------------------------------------------------------------------------------------------------------

### [](#部署小雅 "部署小雅")部署小雅

通过上述的分析，不难得出，可以修改小雅的一键部署脚本，然后直接在 CodeSandBox 的 Devbox 的 Terminal 中运行部署脚本，以将小雅部署到 CodeSandBox 上。小雅的一键脚本功能很简答，无非就是要求你输入 `token`、`opentoken`、`folder_id`，然后把他们储存在一个路径，再 `Docker run`，把这个路径挂载进容器。基本上不需要什么修改，唯一需要修改的就是把创建文件和挂载卷的路径改成在 CodeSandBox 中能够永久储存的路径，操作如下：

#### [](#下载小雅的部署脚本并替换其中的路径然后执行 "下载小雅的部署脚本并替换其中的路径然后执行")下载小雅的部署脚本并替换其中的路径然后执行

在 Terminal 中运行以下命令：

折叠代码块 SHELL 复制代码

```
wget -O update.sh http://docker.xiaoya.pro/update_new.sh && sed -i 's|/etc/xiaoya|/project/sandbox/xiaoya|g' update.sh && chmod +x update.sh && bash update.sh && rm update.sh
```

按照提示输入各种要求的变量即可。运行完毕后可以看到右侧弹窗窗口中对应 5678 端口的那个就是小雅了。

注意，如果你遇到了运行后小雅路径下内没有 `mytoken.txt`、`myopentoken.txt`、`temp_transfer_folder_id.txt`，请不要一个劲重试浪费时间，自己把这三个文件建好并填入应有的内容再放入 xiaoya 路径下即可。小雅的配置文档中本身就有好几个文件需要手动建立并填入内容。

### [](#部署xiaoyakeeper "部署xiaoyakeeper")部署 xiaoyakeeper

经我尝试，xiaoyakeeper 部署在 CodeSandBox 上，如果使用 Docker 模式，会有很多问题，大概与 CodeSandBox 的 `docker.sock`配置有关，所以只能使用模式 0：

折叠代码块 SHELL 复制代码

```
bash -c "$(curl -s https://xiaoyahelper.zengge99.eu.org/aliyun_clear.sh | tail -n +2)" -s 0 -tg
```

如果需要 TG 推送，先把上述命令在 Terminal 中运行一遍，会要求接入 TG 推送，按照要求操作即可。接入完成后使用 `Ctrl+c`结束脚本。

如果你不需要 TG 推送，或者已经运行了刚刚的命令接入了 TG，那么就在左侧文件中新建一个名为”clear.sh” 的文件，把上述命令最后的 `-tg`参数删除，粘贴进去，即为：

折叠代码块 SHELL 复制代码

```
bash -c "$(curl -s https://xiaoyahelper.zengge99.eu.org/aliyun_clear.sh | tail -n +2)" -s 0
```

然后把执行这个脚本添加进容器启动时的任务中，即修改 `.codesandbox`下的 `tasks.json`中的 `tasks`部分：

折叠代码块 SHELL 复制代码

```
"tasks": {
    "xiaoyakeeper": {
      "name": "aliyun_clear",
      "command": "bash /project/sandbox/clear.sh",
      "runAtStart": true
    }
  }
```

这样，当 Devbox 启动时，xiaoyakeeper 就会启动，而 xiaoyakeeper 启动时就会启动小雅容器。

### [](#部署Cloudflared "部署Cloudflared")部署 Cloudflared

如果你就个人使用，实际上没有必要加 Cloudflared 了，只要 `.csb.app`的域名和容器名不泄露，就和使用 Cloudflared 加自己的域名也没太大区别。

这里就简单偷个懒，虽然也可以直接运行 Cloudflared 客户端解决，但是可以用 docker，而且又早都写好了 `docker-compose.yaml`，那就继续用 docker-compose 部署吧：

折叠代码块 YAML 复制代码

```
version: '3'
services:

    cloudflared:
      restart: always
      network_mode: host
      environment:
          - TZ=Asia/Shanghai
      command: tunnel --edge-ip-version auto --protocol quic --heartbeat-interval 10s run --token [ARGO_TOKEN]
      container_name: cloudflared
      image: cloudflare/cloudflared:latest
```

docker-compose.yaml 文件放在. devcontainer 路径下，详细的设置请参考之前写的文章。

这里依旧是修改 `.codesandbox`下的 `tasks.json`文件进行启动，加上启动 xiaoyakeeper 的修改，最终的的 `task.json`为：

折叠代码块 JSON 复制代码

```
{
  // These tasks will run in order when initializing your CodeSandbox project.
  "setupTasks": [
    {
      "name": "Deploy",
      "command": "cd /project/sandbox/.devcontainer/ && docker compose up -d"
    }
  ],

  // These tasks can be run from CodeSandbox. Running one will open a log in the app.
  "tasks": {
    "xiaoyakeeper": {
      "name": "aliyun_clear",
      "command": "bash /project/sandbox/clear.sh",
      "runAtStart": true
    }
  }
}
```

注意，只有 5678 端口需要映射出来，另外两个端口是配合一些 TV 软件使用，这里用不上。如果你确实有需求且自己会配置，请自行研究，这里不做探讨。

到这里就全部部署完成了，只需要 Restart Devbox 检查一下是否正常运行即可。

附上 [Cloudflared](https://github.com/cloudflare/cloudflared/releases)的可执行文件使用命令行直接建立 Tunnel 的命令：

折叠代码块 SHELL 复制代码

```
./cloudflared tunnel --edge-ip-version auto --protocol quic --heartbeat-interval 10s run --token [ARGO_TOKEN]
```

[](#其他应用的部署 "其他应用的部署")其他应用的部署
-----------------------------

通过以上的捣鼓可以发现，`.codesandbox`下的 `tasks.json`文件实际上与 [Replit](https://replit.com/~)的 `replit.nix`和 `.replit`两个文件的功能相近。

在 `setupTasks`部分中你可以使用 `apt-get install`、`npm install`或者 `pip install`等命令进行软件包的安装，亦或者其他的部署指令。

在 `tasks`部分你可以添加一些快捷命令，需要在启动时自动运行的，就把 `runAtStart`设置为 `true`即可。

上述命令，无论是 `setupTasks`还是 `tasks`，都是在 Terminal 中直接运行的命令，区别是，如果 `setupTasks`没有执行完，Devbox 不会启动，而 `tasks`只有在启动完成之后才能执行。

那么就可以很简单的把 Replit 上之前部署的应用搬到 CodeSandBox 上了，`replit.nix`里需要的软件包，就去 `setupTasks`添加命令安装，`.replit`里定义的 `Run`的行为，添加到 `tasks`里去，并且把 `runAtStart`设置为 `true`，其他的文件照搬一下，原本使用 `${REPL_SLUG}`、`${REPL_OWNER}`之类的变量确定的文件路径，统一修改为 `/project/sandbox/`下的任意路径即可。

当然不建议直接照搬，因为 CodeSandBox 配置比 Replit 高不少而且支持 docker，这里建议能用 docker 的都用 docker 部署，源仓库不提供用 docker 部署的，写 dockerfile 构筑 docker 部署。实在不会用 docker，再考虑这样的照搬或者直接运行的方法。

[](#在CodeSandBox中使用Mirai-Console-Loader "在CodeSandBox中使用Mirai-Console-Loader")在 CodeSandBox 中使用 [Mirai-Console-Loader](https://github.com/iTXTech/mirai-console-loader)
=======================================================================================================================================================================

刚好发现了 [ttyd](https://github.com/tsl0922/ttyd)和 [filebrowser](https://github.com/filebrowser/filebrowser)两款神器，感觉使用这两个工具就可以愉快便利的在 CodeSandBox 上使用 Mirai-Console-Loader，遂试了一下，果然可行，记录一下部署过程。

[](#部署过程-1 "部署过程")部署过程
----------------------

### [](#下载Mirai-Console-Loader "下载Mirai-Console-Loader")下载 Mirai-Console-Loader

Terminal 中运行一行命令解决：

折叠代码块 SHELL 复制代码

```
wget -O mcl.zip https://github.com/iTXTech/mirai-console-loader/releases/download/v2.1.2/mcl-2.1.2.zip && unzip -o mcl.zip && rm mcl.zip && chmod +x mcl
```

如果和我一样有强迫症，看着时区不对很烦，毕竟 CodeSandBox 的服务器并不在 UTC+8 区，所以通过手动修改其启动脚本，即名为 `mcl`的那个文件，在执行 jar 包的命令前添加时区参数，即把最后一行改成：

折叠代码块 SHELL 复制代码

```
$JAVA_BINARY -jar -Duser.timezone=Asia/Shanghai mcl.jar $*
```

### [](#修改Devbox的启动任务 "修改Devbox的启动任务")修改 Devbox 的启动任务

修改 `.codesandbox`路径下的 `tasks.json`文件，添加软件包的安装与所需工具的下载与运行命令，修改后的 `tasks.json`文件如下：

折叠代码块 JSON 复制代码

```
{
  // These tasks will run in order when initializing your CodeSandbox project.
  "setupTasks": [
    {
      "name": "Installing java",
      "command": "apt-get update && echo | apt-get install openjdk-21-jre"
    },
    {
      "name": "Downloading ttyd",
      "command": "mkdir -p /project/sandbox/ttyd && cd /project/sandbox/ttyd && wget -O ttyd https://github.com/tsl0922/ttyd/releases/download/1.7.4/ttyd.x86_64 && chmod +x ttyd"
    },
    {
      "name": "Downloading filebrowser",
      "command": "mkdir -p /project/sandbox/filebrowser && cd /project/sandbox/filebrowser && wget -O filebrowser.tar.gz https://github.com/filebrowser/filebrowser/releases/download/v2.26.0/linux-amd64-filebrowser.tar.gz && tar -xzvf filebrowser.tar.gz && chmod +x filebrowser && rm -f filebrowser.tar.gz"
    },
    {
      "name": "Downloading Cloudflared",
      "command": "wget -O cloudflared https://github.com/cloudflare/cloudflared/releases/download/2023.10.0/cloudflared-linux-amd64 && chmod +x cloudflared"
    }
  ],

  // These tasks can be run from CodeSandbox. Running one will open a log in the app.
  "tasks": {
    "ttyd": {
      "name": "ttyd",
      // 在21022端口打开ttyd终端，用户名为admin，密码为password，可以自行修改。
      "command": "./ttyd/ttyd -p 21022 -c admin:password -W zsh",
      "runAtStart": true
    },
    "filebrowser": {
      "name": "filebrowser",
      // 在21021端口打开filebrowser，默认用户名和密码都为admin，登入后请自行在设置中修改用户名和密码，确保安全。
      "command": "./filebrowser/filebrowser -p 21021",
      "runAtStart": true
    },
    "Cloudflared": {
      "name": "Cloudflared",
      // 运行Cloudflare Tunnels，请把最后的[ARGO_TOKEN]替换成自己的。
      "command": "./cloudflared tunnel --edge-ip-version auto --protocol quic --heartbeat-interval 10s run --token [ARGO_TOKEN]",
      "runAtStart": true
    }
  }
}
```

请不要直接复制粘贴，务必看一眼我写的批注，改成自己的信息后再复制粘贴。接着 Restart Devbox，等待重启完毕即可。

记得在 Cloudflare 面板里设置相应的端口对应，这里我使用的是 21021 和 21022 两个端口。

接着打开自己映射的域名，输入自己设置的用户名和密码即可开始畅玩。

[](#后话-1 "后话")后话
----------------

Mirai-Console-Loader 是目前最为成熟的 QQ 机器人使用方法之一了，其插件生态之完善带来了很高的可玩性，不论是之前部署的 ChatGPT 机器人、New Bing 机器人，在 Mirai-Console-Loader 中基本都有相应的插件实现。也是得益于 CodeSandBox 比起 Replit 更为强大的机能，才使得 Mirai-Console-Loader 能够在平台上流畅的运行。

关于 Mirai-Console-Loader 的使用方法、插件安装、登录问题等，其官方文档和论坛也已经记录的非常详细了，这里不多作讨论，只说这一点：

我大概尝试发现的可行的登录方法就是在本地电脑也安装一个 Mirai-Console-Loader，通过手表协议扫码登录之后能够获得一个登录记录的文件夹，把整个文件夹上传到 CodeSandBox 上的相应位置即可。原理和 [go-cqhttp](https://github.com/Mrs4s/go-cqhttp)通过上传 `session.token`文件进行登录是类似的。

讨论群：[738386033](https://jq.qq.com/?_wv=1027&k=qssjFvAs)

[](#在CodeSandBox上搭建代理服务器 "在CodeSandBox上搭建代理服务器")在 CodeSandBox 上搭建代理服务器
======================================================================

使用的是 [3Kmfi6HP/nodejs-proxy](https://github.com/3Kmfi6HP/nodejs-proxy)这个仓库。

[](#部署方法 "部署方法")部署方法
--------------------

新建 Node.js 为 Template 的 Devbox，然后使用以下 `tasks.json`的内容覆盖原本的 `tasks.json`：

折叠代码块 JSON 复制代码

```
{
  // These tasks will run in order when initializing your CodeSandbox project.
  "setupTasks": [
    {
      "name": "Install Dependencies",
      "command": "npm i -g @3kmfi6hp/nodejs-proxy"
    }
  ],
  // These tasks can be run from CodeSandbox. Running one will open a log in the app.
  "tasks": {
    "dev": {
      "name": "Start Dev Server",
      "command": "npx @3kmfi6hp/nodejs-proxy",
      "runAtStart": true,
      "preview": {
        "port": 7860
      },
      "restartOn": {
        "files": ["./package-lock.json"]
      }
    }
  }
}
```

然后 Restart Devbox，等待网页弹出即可。

[](#使用注意 "使用注意")使用注意
--------------------

有个 Bug，实际端口应该是 443 而不是 80，所以要在导出的代理配置中把服务端口从 80 改为 443 才能正常连接，不然连不上。

[](#在CodeSandBox上模拟健康码 "在CodeSandBox上模拟健康码")在 CodeSandBox 上模拟健康码
================================================================

仓库是 [health-code-simulator](https://codeberg.org/ilovexjp/health-code-simulator)。

听说健康码要回来了，咱提前做好准备，有备无患。

[](#部署流程 "部署流程")部署流程
--------------------

### [](#下载源码 "下载源码")下载源码

新建 Template 为 Node.js 的 Devbox。在 Terminal 中输入以下命令把源码下载到 Devbox 中：

折叠代码块 SHELL 复制代码

```
wget -O repo.zip https://codeberg.org/mito/health-code-simulator/archive/main.zip && unzip repo.zip && rm repo.zip && mv -b health-code-simulator/* ./ && mv -b health-code-simulator/.[^.]* ./ && rm -rf *~ && rm -rf health-code-simulator
```

### [](#修改启动任务 "修改启动任务")修改启动任务

修改. codesandbox 路径下的 tasks.json 文件，加入依赖安装和启动的任务：

折叠代码块 JSON 复制代码

```
{
  // These tasks will run in order when initializing your CodeSandbox project.
  "setupTasks": [
    {
      "name": "Install Dependencies",
      "command": "npm install"
    },
    {
      "name": "Building",
      "command": "npm build"
    }
  ],

  // These tasks can be run from CodeSandbox. Running one will open a log in the app.
  "tasks": {
    "start": {
      "name": "start",
      "command": "node build.mjs --serve",
      "runAtStart": true
    }
  }
}
```

接着 Restart Devbox 即可。

[](#在一个容器内同时运行PandoraNext和TokensTool "在一个容器内同时运行PandoraNext和TokensTool")在一个容器内同时运行 PandoraNext 和 TokensTool
===========================================================================================================

在 ChatGPT 的辅助下，我写了一个 Dockerfile，用以构筑 PandoraNext 和 TokensTool 二合一的镜像。

该 Dockerfile 在构筑时会自动从 PandoraNext 和 TokensTool 的 release 中检测最新且符合当前系统架构的版本进行下载，省去了很多麻烦。

折叠代码块 DOCKERFILE 复制代码

```
FROM debian:11.8-slim

WORKDIR /app

# 安装所需的工具
RUN apt-get update && apt-get install -y openjdk-11-jdk curl jq && rm -rf /var/lib/apt/lists/*

# 下载 PandoraNext 的最新版本
FROM debian:11.8-slim

WORKDIR /app

# 安装所需的工具
RUN apt-get update && apt-get install -y openjdk-11-jdk curl jq wget && rm -rf /var/lib/apt/lists/*

# 下载 PandoraNext 的最新版本
RUN ARCH=$(dpkg --print-architecture) && \
    VERSION=$(curl -s https://api.github.com/repos/pandora-next/deploy/releases/latest | jq -r ".tag_name") && \
    DOWNLOAD_URL=$(curl -s https://api.github.com/repos/pandora-next/deploy/releases/latest | jq -r ".assets[] | select(.name | contains(\"linux-${ARCH}\")) | .browser_download_url") && \
    curl -sL ${DOWNLOAD_URL} -o pandoranext.tar.gz && \
    tar -xzf pandoranext.tar.gz --strip-components=1 && \
    chmod +x /app/PandoraNext && \
    rm pandoranext.tar.gz tokens.json config.json

# 下载 PandoraNext-TokensTool 的最新版本
RUN JAR_URL=$(curl -s https://api.github.com/repos/Yanyutin753/PandoraNext-TokensTool/contents/simplyDeploy?ref=main | \
    jq -r ".[] | select(.name | endswith(\".jar\")) | .download_url"); \
    echo "Download URL: $JAR_URL"; \
    wget -O tokenstool.jar "$JAR_URL"

# 创建一个脚本来同时运行 PandoraNext 和 tokenstool.jar，并处理配置文件
RUN echo '#!/bin/sh\nln -sf /data/config.json /app/config.json\nln -sf /data/tokens.json /app/tokens.json\njava -jar tokenstool.jar --server.port=8081 --deployWay=releases --deployPosition=/app --hotReload=true --pandora_Ip=127.0.0.1 &\nsleep 10\n./PandoraNext' > start.sh && \
    chmod +x start.sh

EXPOSE 8081 8181

ENTRYPOINT ["/app/start.sh"]
```

[](#使用方法——以CodeSandBox为例 "使用方法——以CodeSandBox为例")使用方法——以 CodeSandBox 为例
----------------------------------------------------------------------

### [](#构筑镜像 "构筑镜像")构筑镜像

在 DevBox 中新建一个 Dockerfile，并且把上述内容粘贴进去。接着在 Terminal 中输入命令：

折叠代码块 SHELL 复制代码

```
docker build -t pandoranext:latest .
```

### [](#运行容器 "运行容器")运行容器

准备好已经填好的 `config.json`和 `tokens.json`（`TokensTool.json`可以先不修改，之后在 _TokensTool_ 提供的页面内进行操作也可以。），新建一个名为 `pandora`的文件，将这两个文件放入其中。

接着新建 `docker-compose.yaml`文件，并填入以下内容：

折叠代码块 YAML 复制代码

```
version: '3.3'
services:
    pandora-next:
        image: pandoranext:latest
        container_name: PandoraNext
        network_mode: bridge
        restart: always
        ports:
            - "8181:8181"
            - "8081:8081"
        volumes:
            - /project/sandbox/pandora:/data
            - /project/sandbox/sessions:/root/.cache/PandoraNext
```

再去 Terminal 中执行命令：

折叠代码块 SHELL 复制代码

```
docker compose up -d
```

容器即可完成启动。

### [](#使用我构筑的镜像 "使用我构筑的镜像")使用我构筑的镜像

如果你嫌麻烦，也可以直接使用我构筑的镜像，省去自己构筑的时间。同理，新建 pandora 文件夹后放入已经填好的 `config.json`和 `tokens.json`文件，然后把 `docker-compose.yaml`的内容改成如下内容：

折叠代码块 YAML 复制代码

```
version: '3.3'
services:
    pandora-next:
        image: saika2077/pandoranext:latest
        container_name: PandoraNext
        network_mode: bridge
        restart: always
        ports:
            - "8181:8181"
            - "8081:8081"
        volumes:
            - /project/sandbox/pandora:/data
            - /project/sandbox/sessions:/root/.cache/PandoraNext
```

然后直接 `docker compose up -d`即可。我构筑的镜像截止到这篇博客成文为止，_PandoraNext_ 的版本号为 `0.5.2`，_TokensTool_ 的版本号为 `0.4.8.2`。

[](#注意事项 "注意事项")注意事项
--------------------

该容器的使用注意事项与上一篇博客是完全一样的，只是更新方式发生了改变。如果你是自己构筑的方式使用的，`docker compose down`然后执行 `docker build --no-cache -t pandoranext:latest .`再 `docker compose up -d`即可。如果你使用的是我构筑的镜像，在我的镜像更新后，你只需要 `docker compose down && docker compose pull && docker compose up -d`即可完成升级。