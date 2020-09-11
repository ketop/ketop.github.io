---
title: Ubuntu 18.04 LTS美化配置
date: 2020-04-16T16:14:24+08:00
tags: [ubuntu]
draft: false
---
终于又从Archlinux切换到了Ubuntu 18.04 LTS. 原因是发现archlinux确实不稳定，每次滚动升级概率性出现桌面无法启动的现象。

那么这次安装Ubuntu的体验是什么呢？

Problems：
1. 安装Ubuntu时，在分区的时候，如果没有手动建EFI分区，会提示要新建。
    这次终于看清楚了，原来以前Ubuntu总是装不上，就是因为我忽略了这个提示，然后后面安装grub的时候死活装不上。这次在CSDN的某个博主的文章下好好看了这个问题。在新建EFI分区后，成功了。
2. 安装vim插件youcompleteme时，发现现在插件的仓库名变成了'ycm-core/YouCompleteMe'
    这个插件需要下载很多github上的release制品，但是又下不下来（github网站上下载制品特别慢).然后在知乎上看到了一个救星，通过一个网站将所需要的文件下载下来，然后它会给我提供几个下载地址, 这些速度极快，简直不要太爽。附上链接： <https://d.serctl.com/>
3. gnome桌面确实臃肿，但是架不住它可定制能力强啊！看到了McMojave这个主题，真的把持不住了。附上链接: <https://www.gnome-look.org/p/1275087/>

成就：
1. 重新搭建了Linux开发环境，应该说vim开发环境
2. ubuntu现在美化成了MacOS。

附上效果图：
![Ubuntu McMojave](https://s1.ax1x.com/2020/08/16/dEh6rn.png)
