---
title: Nginx 在 windows 下的常用命令
date: 2019-03-25 21:25:36
tags:
    - Windows
    - Nginx
---

`Windows` 下安装 `Nginx` 推荐使用绿色安装的方式，在 `Nginx` 官网下载安装包，直接解压压缩文件就可以了。
安装成功后，进入 `Nginx` 文件夹的安装目录，`Ctrl + 鼠标右键` 打开 `CMD` 或者 `Powershell` 窗口即可。打开的窗口一般如下图所示：

![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/2019331_nginx01.png)

# 1. 启动 Nginx

执行以下命令，启动成功后打开浏览器输入 `127.0.0.1` or `localhost` 访问

``` bash
# 第一种方法，nginx在前台运行，关闭当前命令行窗口时 nginx 自动关闭
PS D:\Program Files\nginx-1.13.7> ./nginx.exe

# 后台运行 nginx
PS D:\Program Files\nginx-1.13.7> start nginx
```

![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/2019331_nginx02.png)

![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/2019331_nginx03.png)

# 2. 关闭 Nginx

```bash
PS D:\Program Files\nginx-1.13.7> ./nginx.exe -s stop

PS D:\Program Files\nginx-1.13.7> ./nginx.exe -s quit
```
注：stop是快速停止 Nginx，可能并不保存相关信息；quit是完整有序的停止 Nginx，并保存相关信息。

# 3、重新载入 Nginx

```bash
# 当配置信息修改，需要重新载入这些配置时使用此命令。
PS D:\Program Files\nginx-1.13.7> ./nginx.exe -s reload
```

# 4、查看 Nginx 版本：

```
PS D:\Program Files\nginx-1.13.7> ./nginx.exe -v
```

~~~

***
