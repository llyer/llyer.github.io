---
title: Jenkins 学习笔记二：Docker + Jenkins + Nginx
date: 2019-04-08 15:53:54
tags:
    - Jenkins 
    - Docker
    - Nginx
---

本篇文章以 https://liluyang.me 为案例来讲解如何实现`持续集成`。主要涉及的技术有 `Docker`，`Jenkins`，`Nginx`

# 准备工作

- 正确的安装 `Docker`
- 正确的使用 `Docker` 安装了 `Jenkins`，`Nginx`

# 配置过程

## 启动 Nginx，映射宿主机目录

```sh
# 将 html 目录和 conf 目录映射到宿主机
docker run \
--volume "/root/docker_nginx/html":/usr/share/nginx/html \
--volume "/root/docker_nginx/conf":/etc/nginx \
-p 80:80 \
-p 443:443 \
-d \
nginx
```

## 启动 Jenkins，映射宿主机目录

```sh
# 在 jenkins 中定义一个 nginx 的  html 工作目录。将这个目录也映射到宿主机的 html 目录中，这样 jenkins 将 git 中的项目文件拉取到jenkins 的工作目录。然后再拷贝到 nginx 的工作目录就完成了持续集成的过程
docker run \
  -u root \
  -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --volume "/root/docker_nginx/html":/usr/share/jenkins/html \
  jenkinsci/blueocean
```

## 配置 Jenkins，持续集成拉取 Git 项目

安装配置好 `jenkins` 之后，新建项目：
![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/20190403_jenkins_01.png)

配置设置如下：
![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/20190403_jenkins_02.png)

其中将 `git` 地址修改为你自己项目的地址，分支也设置为对应的分支。

最后一步，在构建中选择 `执行Shell`，执行以下脚本命令：

```sh
#!/bin/bash 

# 当前 docker 容器映射的宿主机的地址
www_path=/usr/share/jenkins/html

# jenkins将项目从Git拷贝过来的位置
work_path=/var/jenkins_home/workspace/hexo

rm -rf /usr/share/jenkins/html/*

#将生成的文件拷贝到当前的工作目录，由于当前目录和宿主机中的 nginx 目录对应，所以就自动更新了 nginx 的内容
cp -rf /var/jenkins_home/workspace/hexo/* /usr/share/jenkins/html
```

以上全部执行完成后，返回 `jenkins` 的任务界面，选择立即构建。如下图：

![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/20190403_jenkins_03.png)

构建完成后可以点击构建历史中刚刚构建的项目中选择控制台输出查看日志是否构建成功。

接下来可以修改下文件然后检查确认以下流程是否成功了。

1. 执行修改文件

2. 提交到`Git` 

3. `Jenkins` 触发提交事件的锚点而执行自动构建。

4. 访问 `Nginx` 查看项目的变化

到此为止，如果不出意外，你的 `Jenkins` 持续集成已经完成了。

快去尝试一下吧~

~

***
