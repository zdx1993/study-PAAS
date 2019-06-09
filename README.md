# docker镜像配置

https://docker.mirrors.ustc.edu.cn/

亲测,上面这个镜像源,在家里很好用!

对于使用 [systemd](https://www.freedesktop.org/wiki/Software/systemd/) 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

# docker如何修改镜像内容

在开发中我们一定会遇到官方或当前使用的镜像不能满足我们的业务场景这种情况。在这样的情况下,我们不要试图通过修改一个正在运行的容器来满足我们的业务场景,而是通过dockerfile修改镜像完成我们的业务需求。因为容器应该是一个实例对象,不通过修改类而直接修改对象肯定是十分不合理的。总结一下,配置方面的修改可以修改容器,而功能层面的修改应该通过dockerfile重新构建一个镜像,再通过这个镜像run一个容器来实现功能。

# 什么是 GitLab

## [#](https://www.funtl.com/zh/gitlab/%E4%BB%80%E4%B9%88%E6%98%AF-GitLab.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-GitLab-简介与安装](https://www.bilibili.com/video/av27548337)

## [#](https://www.funtl.com/zh/gitlab/%E4%BB%80%E4%B9%88%E6%98%AF-GitLab.html#%E6%A6%82%E8%BF%B0)概述

GitLab 是利用 Ruby on Rails 一个开源的版本管理系统，实现一个自托管的 Git 项目仓库，可通过 Web 界面进行访问公开的或者私人项目。它拥有与 Github 类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序 (Wall) 进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

# 基于 Docker 安装 GitLab

GitLab启动时需要注意最低要分配2个G的内存!

我们使用 Docker 来安装和运行 GitLab 中文版，由于新版本问题较多，这里我们使用目前相对稳定的 10.5 版本，`docker-compose.yml` 配置如下：

```text
version: '3'
services:
    web:
      image: 'twang2218/gitlab-ce-zh:10.5'
      restart: always
      hostname: '192.168.75.145' # 这里本来是需要填写主机名,但是我们在测试的时候并没有域名,所以直接写IP就行了。
      environment:
        TZ: 'Asia/Shanghai' # 环境变量,时区上海
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://192.168.75.145:8080' # 配置docker容器中外部访问地址,Gitlab之所以能够通过web进行访问是使用了nginx
          gitlab_rails['gitlab_shell_ssh_port'] = 2222 # ssh访问端口,也就是说我们的Gitlab是支持ssh访问的。
          unicorn['port'] = 8888
          nginx['listen_port'] = 8080 # nginx的启动(监听)端口,需要与第一个保持一致(external_url),这样才能保证能够正常访问web页面
      ports:
        - '8080:8080'
        - '8443:443'
        - '2222:22'
      volumes:
        - /usr/local/docker/gitlab/config:/etc/gitlab
        - /usr/local/docker/gitlab/data:/var/opt/gitlab
        - /usr/local/docker/gitlab/logs:/var/log/gitlab
```

### [#](https://www.funtl.com/zh/gitlab/%E5%9F%BA%E4%BA%8E-Docker-%E5%AE%89%E8%A3%85-GitLab.html#%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90%E5%90%8E%E7%9A%84%E5%B7%A5%E4%BD%9C)安装完成后的工作

- 访问地址：http://ip:8080
- 端口 8080 是因为我在配置中设置的外部访问地址为 8080，默认是 80
- 初始化安装完成后效果如下：

![img](https://www.funtl.com/assets/Lusifer1511797825.png)

- 设置管理员初始密码，这里的密码最好是 字母 + 数字 组合，并且 大于等于 8 位
- 配置完成后登录，管理员账号是 root

![img](https://www.funtl.com/assets/Lusifer1511798229.png)

**注意：** 如果服务器配置较低，启动运行可能需要较长时间，请耐心等待

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#gitlab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE-2)GitLab 的基本设置

第一次使用时需要做一些初始化设置，点击“管理区域”-->“设置”

![img](https://www.funtl.com/assets/Lusifer1511798480.png)

### [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E8%B4%A6%E6%88%B7%E4%B8%8E%E9%99%90%E5%88%B6%E8%AE%BE%E7%BD%AE)账户与限制设置

关闭头像功能，由于 Gravatar 头像为网络头像，在网络情况不理想时可能导致访问时卡顿

![img](https://www.funtl.com/assets/Lusifer1511798637.png)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E6%B3%A8%E5%86%8C%E9%99%90%E5%88%B6)注册限制

由于是内部代码托管服务器，可以直接关闭注册功能，由管理员统一创建用户即可

![img](https://www.funtl.com/assets/Lusifer1511798763.png)

## 初始化项目

我们选择通过增加一个 README 的方式来初始化项目

![img](https://www.funtl.com/assets/Lusifer1511800836.png)

直接提交修改即可

![img](https://www.funtl.com/assets/Lusifer1511800904.png)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html#%E4%BD%BF%E7%94%A8-ssh-%E7%9A%84%E6%96%B9%E5%BC%8F%E6%8B%89%E5%8F%96%E5%92%8C%E6%8E%A8%E9%80%81%E9%A1%B9%E7%9B%AE)使用 SSH 的方式拉取和推送项目

这里需要注意一点,一个计算机只有一个私钥!在任何一个支持ssh登录的平台只要把当前计算机的公钥,配置到对应帐号的ssh设置中,就能免密登录那个账号,相当于那个账号的密码被私钥代理了!

### [#](https://www.funtl.com/zh/gitlab/GitLab-%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html#%E7%94%9F%E6%88%90-ssh-key)生成 SSH KEY

使用 ssh-keygen 工具生成，位置在 Git 安装目录下，我的是 `C:\Program Files\Git\usr\bin`

输入命令：

```text
ssh-keygen -t rsa -C "your_email@example.com"
```

1

执行成功后的效果：

```text
Microsoft Windows [版本 10.0.14393]
(c) 2016 Microsoft Corporation。保留所有权利。

C:\Program Files\Git\usr\bin>ssh-keygen -t rsa -C "topsale@vip.qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Lusifer/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Lusifer/.ssh/id_rsa.
Your public key has been saved in /c/Users/Lusifer/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:cVesJKa5VnQNihQOTotXUAIyphsqjb7Z9lqOji2704E topsale@vip.qq.com
The key's randomart image is:
+---[RSA 2048]----+
|  + ..=o=.  .+.  |
| o o + B .+.o.o  |
|o   . + +=o+..   |
|.=   .  oo...    |
|= o     So       |
|oE .    o        |
| .. .. .         |
| o*o+            |
| *B*oo           |
+----[SHA256]-----+

C:\Program Files\Git\usr\bin>
```

### [#](https://www.funtl.com/zh/gitlab/GitLab-%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html#%E5%A4%8D%E5%88%B6-ssh-key-%E4%BF%A1%E6%81%AF%E5%88%B0-gitlab)复制 SSH-KEY 信息到 GitLab

秘钥位置在：`C:\Users\你的用户名\.ssh` 目录下，找到 `id_rsa.pub` 并使用编辑器打开，如：

![img](https://www.funtl.com/assets/Lusifer1511801618.png)

登录 GitLab，点击“用户头像”-->“设置”-->“SSH 密钥”

![img](https://www.funtl.com/assets/Lusifer1511801730.png)

成功增加密钥后的效果

![img](https://www.funtl.com/assets/Lusifer1511801884.png)

### [#](https://www.funtl.com/zh/gitlab/GitLab-%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html#%E4%BD%BF%E7%94%A8-tortoisegit-%E5%85%8B%E9%9A%86%E9%A1%B9%E7%9B%AE)使用 TortoiseGit 克隆项目

- 新建一个存放代码仓库的本地文件夹
- 在文件夹空白处按右键
- 选择“Git 克隆...”

![img](https://www.funtl.com/assets/Lusifer1511802101.png)

- 服务项目地址到 URL

![img](https://www.funtl.com/assets/Lusifer1511802242.png)

- 如果弹出连接信息请选择是

![img](https://www.funtl.com/assets/Lusifer1511802354.png)

- 成功克隆项目到本地

![img](https://www.funtl.com/assets/Lusifer1511802402.png)

### [#](https://www.funtl.com/zh/gitlab/GitLab-%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html#%E4%BD%BF%E7%94%A8-tortoisegit-%E6%8E%A8%E9%80%81%E9%A1%B9%E7%9B%AE%EF%BC%88%E6%8F%90%E4%BA%A4%E4%BB%A3%E7%A0%81%EF%BC%89)使用 TortoiseGit 推送项目（提交代码）

- 创建或修改文件（这里的文件为所有文件，包括：代码、图片等）
- 我们以创建 `.gitignore` 过滤配置文件为例，该文件的主要作用为过滤不需要上传的文件，比如：IDE 生成的工程文件、编译后的 class 文件等
- 在工程目录下，新建 `.gitignore` 文件，并填入如下配置：

```text
.gradle
*.sw?
.#*
*#
*~
/build
/code
.classpath
.project
.settings
.metadata
.factorypath
.recommenders
bin
build
target
.factorypath
.springBeans
interpolated*.xml
dependency-reduced-pom.xml
build.log
_site/
.*.md.html
manifest.yml
MANIFEST.MF
settings.xml
activemq-data
overridedb.*
*.iml
*.ipr
*.iws
.idea
.DS_Store
.factorypath
dump.rdb
transaction-logs
**/overlays/
**/logs/
**/temp/
**/classes/
```

- 右键呼出菜单，选择“提交 Master...”

![img](https://www.funtl.com/assets/Lusifer1511802947.png)

- 点击“全部”并填入“日志信息”

![img](https://www.funtl.com/assets/Lusifer1511803046.png)

- 点击“提交并推送”

![img](https://www.funtl.com/assets/Lusifer1511803174.png)

- 成功后的效果图

![img](https://www.funtl.com/assets/Lusifer1511803209.png)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.html#%E6%9F%A5%E7%9C%8B-gitlab-%E7%A1%AE%E8%AE%A4%E6%8F%90%E4%BA%A4%E6%88%90%E5%8A%9F)查看 GitLab 确认提交成功

![img](https://www.funtl.com/assets/Lusifer1511803280.png)

# 什么是 Nexus

## [#](https://www.funtl.com/zh/nexus/#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-Nexus-简介与安装](https://www.bilibili.com/video/av27624502)

## [#](https://www.funtl.com/zh/nexus/#%E6%A6%82%E8%BF%B0)概述

Nexus 是一个强大的仓库管理器，极大地简化了内部仓库的维护和外部仓库的访问。

2016 年 4 月 6 日 Nexus 3.0 版本发布，相较 2.x 版本有了很大的改变：

- 对低层代码进行了大规模重构，提升性能，增加可扩展性以及改善用户体验。
- 升级界面，极大的简化了用户界面的操作和管理。
- 提供新的安装包，让部署更加简单。
- 增加对 Docker, NeGet, npm, Bower 的支持。
- 提供新的管理接口，以及增强对自动任务的管理。

# 基于 Docker 安装 Nexus

我们使用 Docker 来安装和运行 Nexus，`docker-compose.yml` 配置如下：

```text
version: '3.1'
services:
  nexus:
    restart: always
    image: sonatype/nexus3
    container_name: nexus
    ports:
      - 8081:8081
    volumes:
      - /usr/local/docker/nexus/data:/nexus-data
```

*注：* 启动时如果出现权限问题可以使用：`chmod 777 /usr/local/docker/nexus/data` 赋予数据卷目录可读可写的权限

## [#](https://www.funtl.com/zh/nexus/%E5%9F%BA%E4%BA%8E-Docker-%E5%AE%89%E8%A3%85-Nexus.html#%E7%99%BB%E5%BD%95%E6%8E%A7%E5%88%B6%E5%8F%B0%E9%AA%8C%E8%AF%81%E5%AE%89%E8%A3%85)登录控制台验证安装

地址：http://ip:port/ 用户名：admin 密码：admin123

![img](https://www.funtl.com/assets/Lusifer1521047001.png)

# Maven 仓库介绍

## [#](https://www.funtl.com/zh/nexus/Maven-%E4%BB%93%E5%BA%93%E4%BB%8B%E7%BB%8D.html#%E4%BB%A3%E7%90%86%E4%BB%93%E5%BA%93%EF%BC%88proxy-repository%EF%BC%89)代理仓库（Proxy Repository）

意为第三方仓库，如：

- maven-central
- nuget.org-proxy

版本策略（Version Policy）：

- Release: 正式版本
- Snapshot: 快照版本
- Mixed: 混合模式

布局策略（Layout Policy）：

- Strict：严格
- Permissive：宽松

## [#](https://www.funtl.com/zh/nexus/Maven-%E4%BB%93%E5%BA%93%E4%BB%8B%E7%BB%8D.html#%E5%AE%BF%E4%B8%BB%E4%BB%93%E5%BA%93%EF%BC%88hosted-repository%EF%BC%89)宿主仓库（Hosted Repository）

存储本地上传的组件和资源的，如：

- maven-releases
- maven-snapshots
- nuget-hosted

部署策略（Deployment Policy）：

- Allow Redeploy：允许重新部署
- Disable Redeploy：禁止重新部署
- Read-Only：只读

## [#](https://www.funtl.com/zh/nexus/Maven-%E4%BB%93%E5%BA%93%E4%BB%8B%E7%BB%8D.html#%E4%BB%93%E5%BA%93%E7%BB%84%EF%BC%88repository-group%EF%BC%89)仓库组（Repository Group）

通常包含了多个代理仓库和宿主仓库，在项目中只要引入仓库组就可以下载到代理仓库和宿主仓库中的包，如：

- maven-public
- nuget-group

#  在项目中使用 Maven 私服

## [#](https://www.funtl.com/zh/nexus/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-Maven-%E7%A7%81%E6%9C%8D.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-Nexus-在项目中使用 Maven 私服](https://www.bilibili.com/video/av27624534)

## [#](https://www.funtl.com/zh/nexus/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-Maven-%E7%A7%81%E6%9C%8D.html#%E9%85%8D%E7%BD%AE%E8%AE%A4%E8%AF%81%E4%BF%A1%E6%81%AF)配置认证信息

在 Maven `settings.xml` 中添加 Nexus 认证信息(`servers` 节点下)：

```text
<server>
  <id>nexus-releases</id>
  <username>admin</username>
  <password>admin123</password>
</server>

<server>
  <id>nexus-snapshots</id>
  <username>admin</username>
  <password>admin123</password>
</server>
```

### [#](https://www.funtl.com/zh/nexus/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-Maven-%E7%A7%81%E6%9C%8D.html#snapshots-%E4%B8%8E-releases-%E7%9A%84%E5%8C%BA%E5%88%AB)Snapshots 与 Releases 的区别

- nexus-releases: 用于发布 Release 版本
- nexus-snapshots: 用于发布 Snapshot 版本（快照版）

Release 版本与 Snapshot 定义如下：

```text
Release: 1.0.0/1.0.0-RELEASE
Snapshot: 1.0.0-SNAPSHOT
```

- 在项目 `pom.xml` 中设置的版本号添加 `SNAPSHOT` 标识的都会发布为 `SNAPSHOT` 版本，没有 `SNAPSHOT` 标识的都会发布为 `RELEASE` 版本。
- `SNAPSHOT` 版本会自动加一个时间作为标识，如：`1.0.0-SNAPSHOT` 发布后为变成 `1.0.0-SNAPSHOT-20180522.123456-1.jar`

## [#](https://www.funtl.com/zh/nexus/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-Maven-%E7%A7%81%E6%9C%8D.html#%E9%85%8D%E7%BD%AE%E8%87%AA%E5%8A%A8%E5%8C%96%E9%83%A8%E7%BD%B2)配置自动化部署

在 `pom.xml` 中添加如下代码：

```text
<distributionManagement>  
  <repository>  
    <id>nexus-releases</id>  
    <name>Nexus Release Repository</name>  
    <url>http://127.0.0.1:8081/repository/maven-releases/</url>  
  </repository>  
  <snapshotRepository>  
    <id>nexus-snapshots</id>  
    <name>Nexus Snapshot Repository</name>  
    <url>http://127.0.0.1:8081/repository/maven-snapshots/</url>  
  </snapshotRepository>  
</distributionManagement> 
```

注意事项：

- ID 名称必须要与 `settings.xml` 中 Servers 配置的 ID 名称保持一致。
- 项目版本号中有 `SNAPSHOT` 标识的，会发布到 Nexus Snapshots Repository, 否则发布到 Nexus Release Repository，并根据 ID 去匹配授权账号。

## [#](https://www.funtl.com/zh/nexus/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-Maven-%E7%A7%81%E6%9C%8D.html#%E9%83%A8%E7%BD%B2%E5%88%B0%E4%BB%93%E5%BA%93)部署到仓库

```text
mvn deploy
```

1

## [#](https://www.funtl.com/zh/nexus/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-Maven-%E7%A7%81%E6%9C%8D.html#%E4%B8%8A%E4%BC%A0%E7%AC%AC%E4%B8%89%E6%96%B9-jar-%E5%8C%85)上传第三方 JAR 包

Nexus 3.0 不支持页面上传，可使用 maven 命令：

```text
# 如第三方JAR包：aliyun-sdk-oss-2.2.3.jar
mvn deploy:deploy-file 
  -DgroupId=com.aliyun.oss 
  -DartifactId=aliyun-sdk-oss 
  -Dversion=2.2.3 
  -Dpackaging=jar 
  -Dfile=D:\aliyun-sdk-oss-2.2.3.jar 
  -Durl=http://127.0.0.1:8081/repository/maven-3rd/ 
  -DrepositoryId=nexus-releases
```

注意事项：

- 建议在上传第三方 JAR 包时，创建单独的第三方 JAR 包管理仓库，便于管理有维护。（maven-3rd）
- `-DrepositoryId=nexus-releases` 对应的是 `settings.xml` 中 Servers 配置的 ID 名称。（授权）

## [#](https://www.funtl.com/zh/nexus/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-Maven-%E7%A7%81%E6%9C%8D.html#%E9%85%8D%E7%BD%AE%E4%BB%A3%E7%90%86%E4%BB%93%E5%BA%93)配置代理仓库

```text
<repositories>
    <repository>
        <id>nexus</id>
        <name>Nexus Repository</name>
        <url>http://127.0.0.1:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>nexus</id>
        <name>Nexus Plugin Repository</name>
        <url>http://127.0.0.1:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </pluginRepository>
</pluginRepositories>
```

# 安装 Docker Registry 私服

## [#](https://www.funtl.com/zh/registry/#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-Registry-简介与安装](https://www.bilibili.com/video/av27624569)

## [#](https://www.funtl.com/zh/registry/#%E6%A6%82%E8%BF%B0)概述

官方的 Docker Hub 是一个用于管理公共镜像的地方，我们可以在上面找到我们想要的镜像，也可以把我们自己的镜像推送上去。但是，有时候我们的服务器无法访问互联网，或者你不希望将自己的镜像放到公网当中，那么你就需要 Docker Registry，它可以用来存储和管理自己的镜像。

## [#](https://www.funtl.com/zh/registry/#%E5%AE%89%E8%A3%85)安装

在之前的 [Docker 私有仓库](https://www.funtl.com/2018/05/13/docker/Docker-%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93/) 章节中已经提到过如何配置和使用容器运行私有仓库，这里我们使用 `docker-compose` 来安装，配置如下：

```text
version: '3.1'
services:
  registry:
    image: registry
    restart: always
    container_name: registry
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry/data:/var/lib/registry
```

## [#](https://www.funtl.com/zh/registry/#%E6%B5%8B%E8%AF%95)测试

启动成功后需要测试服务端是否能够正常提供服务，有两种方式：

- 浏览器端访问

http://ip:5000/v2/

![img](https://www.funtl.com/assets/Lusifer1520955730.png)

- 终端访问

```text
curl http://ip:5000/v2/
```

![img](https://www.funtl.com/assets/Lusifer1520955773.png)

# 配置 Docker Registry 客户端

## [#](https://www.funtl.com/zh/registry/%E9%85%8D%E7%BD%AE-Docker-Registry-%E5%AE%A2%E6%88%B7%E7%AB%AF.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-Registry-配置 WebUI 与客户端](https://www.bilibili.com/video/av27624593)

## [#](https://www.funtl.com/zh/registry/%E9%85%8D%E7%BD%AE-Docker-Registry-%E5%AE%A2%E6%88%B7%E7%AB%AF.html#%E6%A6%82%E8%BF%B0)概述

我们的教学案例使用的是 Ubuntu Server 16.04 LTS 版本，属于 `systemd` 系统，需要在 `/etc/docker/daemon.json` 中增加如下内容（如果文件不存在请新建该文件）

```text
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [
    "ip:5000"
  ]
}
```

> 注意：该文件必须符合 json 规范，否则 Docker 将不能启动。

之后重新启动服务。

```text
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

## [#](https://www.funtl.com/zh/registry/%E9%85%8D%E7%BD%AE-Docker-Registry-%E5%AE%A2%E6%88%B7%E7%AB%AF.html#%E6%A3%80%E6%9F%A5%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE%E6%98%AF%E5%90%A6%E7%94%9F%E6%95%88)检查客户端配置是否生效

使用 `docker info` 命令手动检查，如果从配置中看到如下内容，说明配置成功（192.168.75.133 为教学案例 IP）

```text
Insecure Registries:
 192.168.75.133:5000
 127.0.0.0/8
```

## [#](https://www.funtl.com/zh/registry/%E9%85%8D%E7%BD%AE-Docker-Registry-%E5%AE%A2%E6%88%B7%E7%AB%AF.html#%E6%B5%8B%E8%AF%95%E9%95%9C%E5%83%8F%E4%B8%8A%E4%BC%A0)测试镜像上传

我们以 Nginx 为例测试镜像上传功能

```text
## 拉取一个镜像
docker pull nginx

## 查看全部镜像
docker images

## 标记本地镜像并指向目标仓库（ip:port/image_name:tag，该格式为标记版本号）
docker tag nginx 192.168.75.133:5000/nginx

## 提交镜像到仓库
docker push 192.168.75.133:5000/nginx
```

## [#](https://www.funtl.com/zh/registry/%E9%85%8D%E7%BD%AE-Docker-Registry-%E5%AE%A2%E6%88%B7%E7%AB%AF.html#%E6%9F%A5%E7%9C%8B%E5%85%A8%E9%83%A8%E9%95%9C%E5%83%8F)查看全部镜像

```text
curl -XGET http://192.168.75.133:5000/v2/_catalog
```

## [#](https://www.funtl.com/zh/registry/%E9%85%8D%E7%BD%AE-Docker-Registry-%E5%AE%A2%E6%88%B7%E7%AB%AF.html#%E6%9F%A5%E7%9C%8B%E6%8C%87%E5%AE%9A%E9%95%9C%E5%83%8F)查看指定镜像

以 Nginx 为例，查看已提交的列表

```text
curl -XGET http://192.168.75.133:5000/v2/nginx/tags/list
```

## [#](https://www.funtl.com/zh/registry/%E9%85%8D%E7%BD%AE-Docker-Registry-%E5%AE%A2%E6%88%B7%E7%AB%AF.html#%E6%B5%8B%E8%AF%95%E6%8B%89%E5%8F%96%E9%95%9C%E5%83%8F)测试拉取镜像

- 先删除镜像

```text
docker rmi nginx
docker rmi 192.168.75.133:5000/nginx
```

- 再拉取镜像

```text
docker pull 192.168.75.133:5000/nginx
```

# 部署 Docker Registry WebUI

私服安装成功后就可以使用 docker 命令行工具对 registry 做各种操作了。然而不太方便的地方是不能直观的查看 registry 中的资源情况。如果可以使用 UI 工具管理镜像就更好了。这里介绍两个 Docker Registry WebUI 工具

- [docker-registry-frontend](https://github.com/kwk/docker-registry-frontend)
- [docker-registry-web](https://hub.docker.com/r/hyper/docker-registry-web/)

两个工具的功能和界面都差不多，我们以 `docker-registry-fontend` 为例讲解

因为一个docker-compose可以管理多个容器的生命周期,所以把registry与front交给同一个docker-compose管理是最佳实践。

## [#](https://www.funtl.com/zh/registry/%E9%83%A8%E7%BD%B2-Docker-Registry-WebUI.html#docker-registry-frontend)docker-registry-frontend

我们使用 docker-compose 来安装和运行，`docker-compose.yml` 配置如下：

```text
version: '3.1'
services:
  frontend:
    image: konradkleine/docker-registry-frontend:v2
    ports:
      - 8080:80
    volumes:
      - ./certs/frontend.crt:/etc/apache2/server.crt:ro
      - ./certs/frontend.key:/etc/apache2/server.key:ro
    environment:
      - ENV_DOCKER_REGISTRY_HOST=192.168.75.133
      - ENV_DOCKER_REGISTRY_PORT=5000
```

> 注意：请将配置文件中的主机和端口换成自己仓库的地址

运行成功后在浏览器访问：http://192.168.75.133:5000

![img](https://www.funtl.com/assets/Lusifer1527005202.png)

![img](https://www.funtl.com/assets/Lusifer1527005783.png)


