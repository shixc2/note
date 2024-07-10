> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/m0_45806184/article/details/126408527)

1、概述
----

### 1.1 什么是持续集成，持续交付

*   持续集成（ Continuous integration ， 简称 CI ）指的是，频繁地（一天多次）将代码集成到主干
*   持续交付 / 持续部署（Continuous Delivery (CD) 、Continuous Deployment (CD)）相当于更进一步的 CI，可以在每次推送到仓库默认分支的同时将应用程序部署到生产环境。

![](https://img-blog.csdnimg.cn/c0b4397c59b54dae9386d81e27a78a3f.png)  
**持续集成的组成要素**

*   一个自动构建过程， 从检出代码、 编译构建、 运行测试、 结果记录、 测试统计等都是自动完成的， 无需人工干预。
*   一个代码存储库，即需要版本控制软件来保障代码的可维护性，同时作为构建过程的素材库，一般使用 SVN 或 Git。
*   一个持续集成服务器， Jenkins 就是一个配置简单和使用方便的持续集成服务器。

3、Jenkins 安装启动
--------------

### 3.1 Jenkins 介绍

![](https://img-blog.csdnimg.cn/1ddcb4c8624b42ff8fa073eecc40450b.png)  
Jenkins 是一款流行的开源持续集成（Continuous Integration）工具，广泛用于项目开发，具有自动化构建、测试和部署等功能。官网： http://jenkins-ci.org/。  
Jenkins 的特征：

*   开源的 Java 语言开发持续集成工具，支持持续集成，持续部署。
*   易于安装部署配置：可通过 yum 安装, 或下载 war 包以及通过 docker 容器等快速实现安装部署，可方便 web 界面配置管理。
*   消息通知及测试报告：集成 RSS/E-mail 通过 RSS 发布构建结果或当构建完成时通过 e-mail 通知，生成 JUnit/TestNG 测试报告。
*   分布式构建：Jenkins 支持能够让多台计算机一起构建 / 测试。
*   文件识别： Jenkins 能够跟踪哪次构建生成哪些 jar，哪次构建使用哪个版本的 jar 等。
*   丰富的插件支持：支持扩展插件，可以开发适合自己团队使用的工具，如 git，svn，maven，docker 等。

![](https://img-blog.csdnimg.cn/38dbf61bb2424d6f8a50d4f1c71b866f.png)

1.  首先，开发人员每天进行代码提交，提交到 Git 仓库
2.  然后，Jenkins 作为持续集成工具，使用 Git 工具到 Git 仓库拉取代码到集成服务器，再配合 JDK，Maven 等软件完成代码编译，代码测试与审查，测试，打包等工作，在这个过程中每一步出错，都重新再执行一次整个流程。
3.  最后，Jenkins 把生成的 jar 或 war 包分发到测试服务器或者生产服务器，测试人员或用户就可以访问应用。

### 3.2 Jenkins 与 gitlabCI/CD 的区别

#### 3.2.1 gitlabCI/CD 介绍

gitlab-ci 作为 gitlab 提供的一个持续集成的套件，完美和 gitlab 进行集成，gitlab-ci 已经集成进 gitlab 服务器中，在使用的时候只需要安装配置 gitlab-runner 即可。gitlab-runner 基本上提供了一个可以进行编译的环境，负责从 gitlab 中拉取代码，根据工程中配置的 gitlab-ci.yml，执行相应的命令进行编译。

#### 3.2.2 gitlabCI vs Jenkins

gitlab-runner 配置简单，很容易与 gitlab 集成。当新建一个项目的时候，不需要配置 webhook 回调地址，也不需要同时在 jenkins 新建这个项目的编译配置，只需在工程中配置 gitlab-ci.yml 文件，就可以让这个工程可以进行编译。gitlab-runner 没有 web 页面，但编译的过程直接就在 gitlab 中可以看到，不需要像 jenkins 进入 web 控制台查看编译过程。gitlab-runner 仅仅是提供了一个编译的环境而已，全部的编译都通过 shell 脚本命令进行。当然，jenkins 也可以是全部的编译都通过 shell 脚本命令进行。

jenkins 的好处就是编译服务和代码仓库分离，而且编译配置文件不需要在工程中配置，如果团队有开发、测试、配置管理员、运维、实施等完整的人员配置，那就采用 jenkins，这样职责分明。不仅仅如此，jenkins 依靠它丰富的插件，可以配置很多 gitlab-ci 不存在的功能，比如说看编译状况统计等。如果团队是互联网类型，讲究的是敏捷开发，那么开发 = devOps，肯定是采用最便捷的开发方式，推荐 gitlab-ci。

如果有些敏感的配置文件不方便存放在工程中（例如 nexus 上传 jar 的账户和密码或者是其他配置的账户密码）, 都可以在服务器中配置即可。  
gitlab-ci 对于编译需要的环境，比如 jdk，maven 都需要自行配置。在 jenkins 中，对于编译需要的环境，比如 jdk，maven 都可以在 Web 控制台安装即可。当然，jenkins 也是可以自行配置的（有时候通过控制台配置下载不下来）。

一句话总结就是 gitlab-ci 更简单易用，如果有 gitlab-ci 达不到的要求，可以考虑使用 jenkins。

### 3.3 Jenkins 环境搭建

#### 3.3.1 Jenkins 安装配置

1.  采用 YUM 方式安装

加入 jenkins 安装源：

```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo --no-check-certificate

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

执行 yum 命令安装

```
yum -y install jenkins
```

2.  采用 rpm 安装包方式

```
wget https://pkg.jenkins.io/redhat-stable/jenkins-2.190.1-1.1.noarch.rpm
```

执行安装

```
rpm -ivh jenkins-2.190.1-1.1.noarch.rpm
```

3.  配置

**修改 jenkins 配置文件**

```
vi /etc/sysconfig/jenkins
```

**修改内容**

```
# 修改为对应的目标用户， 这里使用的是root
$JENKINS_USER="root"
# 服务监听端口
JENKINS_PORT="16060"
```

**修改目录权限**

```
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```

**重启 Jenkins**

```
systemctl restart jenkins
```

**如果遇到启动失败， 出现如下错误信息**

```
Starting Jenkins bash: /usr/bin/java: No such file or directory
```

这是由于 Jenkins 缺少 java 环境，可以通过 ln 指令建立一个软连接

```
ln -s /usr/local/jdk/bin/java /usr/bin/java
```

跑题插一下 ln 指令介绍

```
1、命令格式
ln 参数 源文件或目录 目标文件或目录
2、命令功能
ln是linux中非常重要命令，它的功能是为某一个文件在另外一个位置建立一个同步的链接.当我们需要在不同的目录，用到相同的文件时，我们不需要在每一个需要的目录下都放一个必须相同的文件，我们只要在某个固定的目录，放上该文件，然后在 其它的目录下用ln命令链接(link)它就可以，不必重复的占用磁盘空间。
3、命令参数
-b 删除，覆盖以前建立的链接
-d 允许超级用户制作目录的硬链接
-f 强制执行
-i 交互模式，文件存在则提示用户是否覆盖
-n 把符号链接视为一般目录
-s 软链接(符号链接)
-v 显示详细的处理过程
```

或者通过

```
vi /etc/init.d/jenkins
```

![](https://img-blog.csdnimg.cn/7526b3b6910148008c65c13673811401.png)  
加上红色框选的 jdk 配置，然后

```
systemctl daemon-reload
```

重新加载一下配置即可。  
**设置好 java 环境之后，再重启 Jenkins 即可**

4.  管理后台初始化  
    浏览器打开 http://IP:16060/  
    弹出页面需要输入管理密码， 在以下位置查看：

```
cat /var/lib/jenkins/secrets/initialAdminPassword
```

![](https://img-blog.csdnimg.cn/8da3d3993417474d988a661bff348170.png)  
**按默认设置，把建议的插件都安装上**  
![](https://img-blog.csdnimg.cn/81676f5dfaab4d4792682287d0c35228.png)  
安装插件时，如果出现安装失败的情况，有几种情况，1、是网络不行，2、Jenkins 版本过低，有些插件不支持，这时就需要卸载 Jenkins 重新安装新版的。  
**安装完成之后， 就可以创建管理员用户：**  
![](https://img-blog.csdnimg.cn/2584239d17f04bf39fe45fdf769e1a21.png)

**配置访问地址：**  
![](https://img-blog.csdnimg.cn/5a912768b77846fbbc9d727d850551ff.png)  
配置完成之后， 会进行重启， 之后可以看到管理后台：  
![](https://img-blog.csdnimg.cn/3abc5d4faa7647a0b9fa9f205124db59.png)

#### 3.3.2 Jenkins 插件安装

在实现持续集成之前， 需要确保以下插件安装成功。

*   Maven Integration plugin： Maven 集成管理插件。
*   Docker plugin： Docker 集成插件。
*   GitLab Plugin： GitLab 集成插件。
*   Publish Over SSH：远程文件发布插件。
*   SSH: 远程脚本执行插件。

安装方法：

1.  进入【系统管理】-【插件管理】
    
2.  点击标签页的【可选插件】  
    在过滤框中搜索插件名称  
    ![](https://img-blog.csdnimg.cn/77040128be044d7b8d743604873a9dae.png)  
    **勾选插件，点击安装即可**  
    注意：若是没有安装按钮，需要更改配置。在安装插件的高级配置中，修改升级站点的连接为：http://updates.jenkins.io/update-center.json 保存即可  
    ![](https://img-blog.csdnimg.cn/903a9a1dfd7d45a99ccbd2f11281e87a.png)
    

#### 3.3.3 git 安装配置

1.  yum 安装方式

```
yum -y install git
```

2.  采用源码包方式安装  
    安装依赖包

```
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum -y install gcc perl-ExtUtils-MakeMaker
```

如果之前有安装旧版本， 先做卸载， 没有安装则忽略

```
yum remove git
```

下载源码包

```
cd /usr/local
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-1.8.3.1.tar.gz
tar -xvf git-1.8.3.1.tar.gz
```

也可以安装其他版本， 下载地址：https://mirrors.edge.kernel.org/pub/software/scm/git/  
编译安装：

```
cd git-1.8.3.1
make prefix=/usr/local/git all
make prefix=/usr/local/git install
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc  # 将命令输出到bashrc文件中
source /etc/bashrc  # 按顺序执行bashrc中的命令
```

3.  检查 git 版本

```
[root@localhost jenkins]# git version
git version 1.8.3.1
```

#### 3.3.4 安装 maven 配置

1.  下载安装包

```
下载地址： https://maven.apache.org/download.cgi
```

2.  解压安装包

```
cd /usr/local
unzip -o apache-maven-3.6.1.zip
```

3.  上传本地仓库并解压

![](https://img-blog.csdnimg.cn/1fcf592d9c5845348c797b4f7ac56aa2.png)pwd 指令可立刻得知目前所在的工作目录的绝对路径名称。

4.  配置

配置环境变量

```
vi /etc/profile
```

添加配置内容

```
export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.1
export PATH=$PATH:$MAVEN_HOME/bin
```

如果权限不够，则需要增加当前目录的权限

```
chmod 777 /usr/local/maven/apache-maven-3.6.1/bin/mvn
```

**修改镜像仓库配置**

```
vi /usr/local/maven/apache-maven-3.6.1/conf/settings.xml
```

配置内容

```
<mirror>
     <id>alimaven</id>
     <mirrorOf>central</mirrorOf>
     <name>aliyun maven</name>
     <url>https://maven.aliyun.com/repository/central</url>
    </mirror>
```

修改本地仓库配置

```
<localRepository>/path/to/local/repo</localRepository>
```

#### 3.3.5 docker 安装配置

1.  更新软件包版本

```
yum -y update
```

2.  卸载旧版本

```
yum -y remove docker  docker-common docker-selinux docker-engine
```

3.  安装软件依赖包

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

4.  设置 yum 源为阿里云

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

5.  安装 docker 及查看 docker 版本

```
yum install docker-ce -y
```

```
docker -v
```

6.  设置开机启动

```
systemctl enable docker
```

7.  启动 docker

```
systemctl start docker
```

### 3.4 Jenkins 工具配置

1.  进入【系统管理】–> 【全局工具配置】  
    ![](https://img-blog.csdnimg.cn/b0b838d3add2401f9ecef2a4ffb955d3.png)
2.  MAVEN 配置设置  
    ![](https://img-blog.csdnimg.cn/506d8b562d8949779fa7e97f5c601616.png)  
    填写你的 maven 安装路径即可。
3.  JDK 配置  
    ![](https://img-blog.csdnimg.cn/2ef41e2a36b846718cd5f4cbd213ecdc.png)
4.  指定 maven 目录  
    ![](https://img-blog.csdnimg.cn/91bf96749c164d0085101e81dd0c12fd.png)  
    填写你的 maven 安装目录。
5.  指定 docker 目录  
    ![](https://img-blog.csdnimg.cn/0a9ff16613b249a0a07c010e55e57ab2.png)  
    如果不清楚 docker 的安装的目录，可以使用`whereis docker` 命令查看 docker 的安装的目录。

**以上就完成了 Jenkins 的环境搭建准备，可以尝试部署项目了。**

4、项目部署
------

### 4.1 服务集成 docker 配置

目标：部署的每一个微服务都是先创建 docker 镜像后创建对应容器启动

方式一：本地微服务打包以后上传到服务器，编写 Dockerfile 文件完成。

方式二：使用 dockerfile-maven-plugin 插件，可以直接把微服务创建为镜像使用（更省事）  
**配置**

**插件的作用：**

1.  pring-boot-maven-plugin  
    将 springboot 应用打包为可执行的 jar 或者 war 包
2.  maven-compile-plugin  
    用来编译项目代码
3.  dickerfile-maven-plugin  
    使用该插件，必须提供 dockerfile 文件，而且要求放在与 pom 文件同级目录  
    repositoty 标签指定 docker 镜像仓库的名字  
    buildArgs 标签可以指定一个或者多个变量，传递给 Dockerfile，在 dockerfile 中通过 ARG 指令进行引用。

```
<finalName>my-service</finalName>
 <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.3.6</version>
                <configuration>
                    <repository>docker_storage/${project.artifactId}</repository>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
        </plugins>
```

**Dockerfile 文件**

```
# 设置JAVA版本
FROM java:8
# 指定存储卷, 任何向/tmp写入的信息都不会记录到容器存储层
VOLUME /tmp
# 拷贝运行JAR包
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
# 设置JVM运行参数， 这里限定下内存大小，减少开销
ENV JAVA_OPTS="\
-server \
-Xms256m \
-Xmx512m \
-XX:MetaspaceSize=256m \
-XX:MaxMetaspaceSize=512m"
# 空参数，方便创建容器时传参
ENV PARAMS=""
# 入口点， 执行JAVA运行命令
ENTRYPOINT ["sh","-c","java -jar $JAVA_OPTS /app.jar $PARAMS"]
```

### 4.2 Jenkins 基础依赖打包配置

在微服务运行之前需要在本地仓库中先去 install 所依赖的 jar 包，所以第一步应该是从 git 中拉取代码，并且把基础的依赖部分安装到仓库中。

1.  以父工程的名称在 Jenkins 上创建一个构建软件项目  
    ![](https://img-blog.csdnimg.cn/65140f5df2524acba4c6f34923682d3a.png)
2.  找到自己指定的 git 仓库，credentials 设置用户名和密码  
    ![](https://img-blog.csdnimg.cn/cf9a95ff667d4d04a4186efd4d25145d.png)
3.  把基础依赖信息安装到集成服务器上的本地仓库  
    ![](https://img-blog.csdnimg.cn/59637771a641468cac77681d2d644143.png)
4.  执行构建，贴出部分构建日志  
    拉取代码日志  
    ![](https://img-blog.csdnimg.cn/391ffb16600643058fe8feb89d4fde17.png)  
    构建成功日志  
    ![](https://img-blog.csdnimg.cn/c5d05b792d6d4185b666c70f3d0408d1.png)

### 4.3 Jenkins 微服务打包配置

1.  新建任务  
    ![](https://img-blog.csdnimg.cn/967393da2c49449f9d439f863aaa3e94.png)
2.  配置代码的 giturl 地址，设置用户名和密码，设置构建分支  
    ![](https://img-blog.csdnimg.cn/1921686b7a3e480c912fab39fdf225a6.png)
3.  执行 maven 命令  
    ![](https://img-blog.csdnimg.cn/7492bd73abc74d9d870c9417eab57a53.png)  
    ![](https://img-blog.csdnimg.cn/ae6f0bcb16bc4dd59f78a8c136212fc1.png)  
    maven 指令

```
clean install -Dmaven.test.skip=true  dockerfile:build -f 
***/***/pom.xml
```

**命令拆解：**  
*** 代表要构建的模块名称  
-Dmaven.test.skip=true 跳过测试

dockerfile:build 启动 Dockerfile 插件构建容器

-f ***/pom.xml 指定需要构建的文件（必须是 pom）

4.  执行 shell 脚本  
    ![](https://img-blog.csdnimg.cn/cae602e006824833bf3049324574cb6e.png)  
    **shell 命令内容**

```
if [ -n  "$(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )" ]  
# 查看是否有正在运行的与即将要构建的服务一样名字的容器
 then
 #有的话，删除之前的容器
 docker rm -f $(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )
fi
 # 清理镜像
docker image prune -f 
 # 启动docker服务
docker run -d --net=host -e PARAMS="--spring.profiles.active=prod"  --name $JOB_NAME docker_storage/$JOB_NAME
```

5.  开始执行，贴出执行日志

拉取代码![](https://img-blog.csdnimg.cn/240b9bb1b8424588920870e75465701f.png)  
编译打包，构建镜像  
![](https://img-blog.csdnimg.cn/2ba8ab6ba80744d4a1a8a83aa4a57916.png)  
清理容器，创建新的容器

![](https://img-blog.csdnimg.cn/970d3e0b1acf45b6aabafc58c85d5735.png)

### 4.4 部署服务到远程服务器上

目标：使用 jenkins（…100）把微服务打包部署到… 260 服务器上；过程示意图如下。  
![](https://img-blog.csdnimg.cn/17b5e2328c3c41ec8dd64eec3d6addfd.png)

#### 4.4.1 安装配置 docker 镜像仓库

对于持续集成环境的配置，Jenkins 会发布大量的微服务， 要与多台机器进行交互， 可以采用 docker 镜像的保存与导出功能结合 SSH 实现， 但这样交互繁琐，稳定性差， 而且不便管理， 这里我们通过搭建 Docker 的私有仓库来实现， 这个有点类似 GIT 仓库， 集中统一管理资源， 由客户端拉取或更新。  
下面简单介绍 docker 官方提供的镜像仓库 docker registry 的安装配置

##### 4.4.1.1 下载最新 Registry 镜像

```
docker pull registry:latest
```

##### 4.4.1.2 启动 Registry 容器

```
docker run -d -p 5000:5000 --name registry -v /usr/local/docker/registry:/var/lib/registry registry:latest
```

映射 5000 端口； -v 是将 Registry 内的镜像数据卷与本地文件关联， 便于管理和维护 Registry 内的数据。

##### 4.4.1.3 查看仓库资源

若是启动成功，访问地址：http://IP:5000/v2/_catalog；可以看到返回：

```
{"repositories":[]}
```

目前并没有上传镜像， 显示空数据；如果上传成功就可以看到数据。

##### 4.4.1.4 配置 Docker 客户端

正常生产环境中使用， 要配置 HTTPS 服务， 确保安全，内部开发或测试集成的局域网环境，可以采用简便的方式， 不做安全控制。先确保持续集成环境的机器已安装好 Docker 客户端， 然后做以下修改：

```
vi /lib/systemd/system/docker.service
```

修改内容：

```
ExecStart=/usr/bin/dockerd --insecure-registry IP:5000
```

重启 docker 生效：

```
systemctl daemon-reolad
systemctl restart docker.service
```

#### 4.4.2 Jenkins 安装插件

![](https://img-blog.csdnimg.cn/2a6b522aef444799992a43a59e3890dd.png)

#### 4.4.3 jenkins 系统配置远程服务器链接

位置：Manage Jenkins–>Configure System  
![](https://img-blog.csdnimg.cn/884eeee0f86f473ab03445f4c8e013ba.png)  
需要添加凭证  
位置：Manage Jenkins–>Manage CreDentials  
![](https://img-blog.csdnimg.cn/d74d2b423be64dd7a6205a81ca466932.png)  
添加链接到远程服务器的用户名和密码：  
![](https://img-blog.csdnimg.cn/64e75b884b474e2288f9bf2bb9c92b97.png)  
![](https://img-blog.csdnimg.cn/03c3ec096eb3431999b7186adbd236be.png)

#### 4.4.4 Jenkins 设置镜像仓库的参数

![](https://img-blog.csdnimg.cn/e719e7a3bc7c4d068e524de80c37d25b.png)

#### 4.4.5 构建执行 Execute shell

![](https://img-blog.csdnimg.cn/2d43010f91ea4a359dd31eda65321b43.png)  
maven 命令：

```
clean install -Dmaven.test.skip=true dockerfile:build -f ***/***/pom.xml
```

shell 脚本，推送镜像至仓库

```
image_tag=$docker_registry/docker_storage/$JOB_NAME
echo '================docker镜像清理================'
if [ -n  "$(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )" ]
 then
 #删除之前的容器
 docker rm -f $(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )
fi
 # 清理镜像
docker image prune -f 

# 创建TAG
docker tag docker_storage/$JOB_NAME $image_tag
echo '================docker镜像推送================'
# 推送镜像
docker push $image_tag
# 删除TAG
docker rmi $image_tag
echo '================docker tag 清理 ================'
```

#### 4.4.6 Jenkins 配置在远程服务器上执行脚本

![](https://img-blog.csdnimg.cn/7dbdbf32c5824064ba1341b20ccaeb22.png)  
shell 脚本

```
echo '================拉取最新镜像================'
docker pull $docker_registry/docker_storage/$JOB_NAME

echo '================删除清理容器镜像================'
if [ -n  "$(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )" ]
 then
 #删除之前的容器
 docker rm -f $(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )
fi
 # 清理镜像
docker image prune -f 

echo '===============启动容器================'
docker run -d   --net=host -e PARAMS="--spring.profiles.active=prod" --name $JOB_NAME $docker_registry/docker_storage/$JOB_NAME
```

docker run 创建 Docker 容器时，可以用 --net 选项指定容器的网络模式 ：  
host 模式：使用 --net=host 指定。  
none 模式：使用 --net=none 指定。  
bridge 模式：使用 --net=bridge 指定，默认设置。  
container 模式：使用 --net=container:NAME_or_ID 指定  
![](https://img-blog.csdnimg.cn/6fe99e0cf48f4dc0ac28b1fb7e144838.png)  
docker -e 的作用是指定容器内的环境变量。

#### 4.4.7 登录远程服务器，查看是否有相关的镜像和容器和镜像

**查看镜像**

![](https://img-blog.csdnimg.cn/6d6801b2020b44cab7ecbd9efedd2f2f.png)  
查看容器  
![](https://img-blog.csdnimg.cn/7bad671242024045849a696f4399f403.png)

5、自动触发构建配置
----------

### 5.1 URL 触发远程构建

触发远程构建，修改 jenkins 的配置，如下  
![](https://img-blog.csdnimg.cn/7b3657a6a90c4693b65b7d8ece193ceb.png)  
触发构建 url： http://IP:16060/job/code/build?token=88888888

### 5.2 定时构建

定时构建（ Build periodically）  
![](https://img-blog.csdnimg.cn/a8538b4e2e0e468991d37505d26a7fd2.png)  
**定时构建 - 定时表达式**  
定时字符串从左往右分别为： 分 时 日 月 周

![](https://img-blog.csdnimg.cn/c0cff7235cd746d49375326111bbc2f2.png)

*   符号 H 表示一个随机数
*   符号 * 取值范围的任意值

示例：

*   每 30 分钟构建一次：H/30 * * * * 10:02 10:32
*   每 2 个小时构建一次: H H/2 * * *
*   每天的 8 点，12 点，22 点，一天构建 3 次： (多个时间点中间用逗号隔开) 0 8,12,22 * * *
*   每天中午 12 点定时构建一次 H 12 * * *
*   每天下午 18 点定时构建一次 H 18 * * *

### 5.3 轮询

轮询 SCM（Poll SCM）  
轮询 SCM，是指定时扫描本地代码仓库的代码是否有变更，如果代码有变更就触发项目构建。  
![](https://img-blog.csdnimg.cn/f20724b15edb45979036fa82914f2cdc.png)  
Jenkins 会定时扫描本地整个项目的代码，增大系统的开销，不建议使用。  
以上就是 git + Jenkins + docker 实现的持续集成，持续部署方案，欢迎点赞收藏。