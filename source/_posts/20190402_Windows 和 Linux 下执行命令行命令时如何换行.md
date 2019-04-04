---
title: Windows 和 Linux 下执行命令行命令时如何换行
date: 2019-04-02 14:24:47
tags:
    - Windows
    - Linux
---

注意，如果命令需要换行，请添加换行字符 `windows` 为  “`^`”和 “\`”，`linux` 为 “`\`”

```sh
# 不换行
ffmpeg  -i ./input.mp4 -vcodec copy -acodec copy -ss 00:00:10 -to 00:00:15 ./cut01.mp4 -y
```

```sh

# linux 环境
ffmpeg  -i ./output.mp4 \
        -vcodec copy \
        -acodec copy \
        -ss 00:00:10 -to 00:00:15 \
        ./cutout1.mp4 \
        -y
```

```sh
# windows CMD环境
ffmpeg  -i ./output.mp4 -vcodec copy ^
        -acodec copy ^
        -ss 00:00:10 ^
        -to 00:00:15 ^
        ./cutout1.mp4 ^
        -y

# windows Powershell环境
ffmpeg  -i ./output.mp4 -vcodec copy `
        -acodec copy `
        -ss 00:00:10 `
        -to 00:00:15 `
        ./cutout1.mp4 `
        -y
```

~

***