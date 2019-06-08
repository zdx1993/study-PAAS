# 什么是 GitLab

## [#](https://www.funtl.com/zh/gitlab/%E4%BB%80%E4%B9%88%E6%98%AF-GitLab.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-GitLab-简介与安装](https://www.bilibili.com/video/av27548337)

## [#](https://www.funtl.com/zh/gitlab/%E4%BB%80%E4%B9%88%E6%98%AF-GitLab.html#%E6%A6%82%E8%BF%B0)概述

GitLab 是利用 Ruby on Rails 一个开源的版本管理系统，实现一个自托管的 Git 项目仓库，可通过 Web 界面进行访问公开的或者私人项目。它拥有与 Github 类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序 (Wall) 进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

# 基于 Docker 安装 GitLab

我们使用 Docker 来安装和运行 GitLab 中文版，由于新版本问题较多，这里我们使用目前相对稳定的 10.5 版本，`docker-compose.yml` 配置如下：

```text
version: '3'
services:
    web:
      image: 'twang2218/gitlab-ce-zh:10.5'
      restart: always
      hostname: '192.168.75.145'
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://192.168.75.145:8080'
          gitlab_rails['gitlab_shell_ssh_port'] = 2222
          unicorn['port'] = 8888
          nginx['listen_port'] = 8080
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

# GitLab 的基本设置

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-GitLab-基本设置](https://www.bilibili.com/video/av27548356)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#gitlab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE-2)GitLab 的基本设置

第一次使用时需要做一些初始化设置，点击“管理区域”-->“设置”

![img](https://www.funtl.com/assets/Lusifer1511798480.png)

### [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E8%B4%A6%E6%88%B7%E4%B8%8E%E9%99%90%E5%88%B6%E8%AE%BE%E7%BD%AE)账户与限制设置

关闭头像功能，由于 Gravatar 头像为网络头像，在网络情况不理想时可能导致访问时卡顿

![img](https://www.funtl.com/assets/Lusifer1511798637.png)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E6%B3%A8%E5%86%8C%E9%99%90%E5%88%B6)注册限制

由于是内部代码托管服务器，可以直接关闭注册功能，由管理员统一创建用户即可

![img](https://www.funtl.com/assets/Lusifer1511798763.png)

# GitLab 的基本设置

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-GitLab-基本设置](https://www.bilibili.com/video/av27548356)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#gitlab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE-2)GitLab 的基本设置

第一次使用时需要做一些初始化设置，点击“管理区域”-->“设置”

![img](https://www.funtl.com/assets/Lusifer1511798480.png)

### [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E8%B4%A6%E6%88%B7%E4%B8%8E%E9%99%90%E5%88%B6%E8%AE%BE%E7%BD%AE)账户与限制设置

关闭头像功能，由于 Gravatar 头像为网络头像，在网络情况不理想时可能导致访问时卡顿

![img](https://www.funtl.com/assets/Lusifer1511798637.png)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E6%B3%A8%E5%86%8C%E9%99%90%E5%88%B6)注册限制

由于是内部代码托管服务器，可以直接关闭注册功能，由管理员统一创建用户即可

![img](https://www.funtl.com/assets/Lusifer1511798763.png)

# GitLab 的基本设置

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)本节视频

- [【视频】平台即服务-GitLab-基本设置](https://www.bilibili.com/video/av27548356)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#gitlab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE-2)GitLab 的基本设置

第一次使用时需要做一些初始化设置，点击“管理区域”-->“设置”

![img](https://www.funtl.com/assets/Lusifer1511798480.png)

### [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E8%B4%A6%E6%88%B7%E4%B8%8E%E9%99%90%E5%88%B6%E8%AE%BE%E7%BD%AE)账户与限制设置

关闭头像功能，由于 Gravatar 头像为网络头像，在网络情况不理想时可能导致访问时卡顿

![img](https://www.funtl.com/assets/Lusifer1511798637.png)

## [#](https://www.funtl.com/zh/gitlab/GitLab-%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AE%BE%E7%BD%AE.html#%E6%B3%A8%E5%86%8C%E9%99%90%E5%88%B6)注册限制

由于是内部代码托管服务器，可以直接关闭注册功能，由管理员统一创建用户即可

![img](https://www.funtl.com/assets/Lusifer1511798763.png)