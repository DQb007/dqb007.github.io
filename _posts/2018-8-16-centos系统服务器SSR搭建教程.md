---
layout: post
title: "centos系统SSR服务器搭建教程"
description: "centos系统SSR服务器搭建教程"
categories: [工具]
cover: 'https://i.loli.net/2018/08/22/5b7cd5e857bbd.jpg'
tags: [工具, SSR]
---


>本脚本适合centos6.0,此系统是基于Redhat

###  一、安装脚本
#### 1、安装wget

```Java
yum install wget -y
```

#### 2、下载SSR一键安装脚本并执行

```Java
wget --no-check-certificate https://freed.ga/github/shadowsocksR.sh; bash shadowsocksR.sh
```
然后按照提示操作

### 二、SSR速度优化
#### 1、使用锐速加速
- 更换系统内核，更换系统内核需要几分钟请耐心等待，不要做其他操作

```Java
wget -N --no-check-certificate https://freed.ga/kernel/ruisu.sh && bash ruisu.sh
```

- 使用脚本安装锐速

```Java
wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh
```
- 备用脚本

```Java
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
```

#### 2、使用BBR加速
- 安装BBR

```Java
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod a+x bbr.sh && bash bbr.sh -f
```

### 三、SSR的相关使用命令
#### 1、修改shadowsocks.json配置文件

```Java
vim /etc/shadowsocks.json
```
#### 2、SSR配置命令行方式
- 启动SSR

```Java
/etc/init.d/shadowsocks start
```

- 停止SSR

```Java
/etc/init.d/shadowsocks stop
```

-  重启SSR

```Java
/etc/init.d/shadowsocks restart
```

- 查看SSR的状态

```Java
/etc/init.d/shadowsocks status
```
