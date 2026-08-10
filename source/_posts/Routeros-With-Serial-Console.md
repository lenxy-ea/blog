---
title: 给Routeros安装程序加入Serial Output
date: 2025-04-13 13:54:57
tags: 随笔
---

# 给Routeros安装程序加入Serial Output

> 单纯一个随笔，方便日后查询

在近日折腾C3558卵路由的时候，安装Routeros总是出现问题。载入硬盘后就变得没有输出。索性手动折腾一下。

![](../img/GlobalCare-Zi0j930q.png)

解压后获取安装img，双击挂载。

![](../img/GlobalCare-ZW67IhGq.png)

修改syslinux.cfg

```
default system
LABEL system
	KERNEL linux
	APPEND load_ramdisk=1 -install -hdd
```

修改加入console输出即可

```
SERIAL 0 115200
CONSOLE 0
default system
LABEL system
	KERNEL linux
	APPEND console=ttyS0,115200n8 load_ramdisk=1 -install -hdd

```

保存配置文件并umount，随后使用dd或rufus将img文件写入U盘即可正常引导。
