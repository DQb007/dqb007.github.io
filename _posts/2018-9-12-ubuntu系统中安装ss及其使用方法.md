---
layout: post
title: "ubuntu系统中安装ss及其使用方法"
description: "ubuntu系统中安装ss及其使用方法"
categories: [工具]
tags: [工具, SSR]
---

### 一、在ubuntu系统中安装shadowsocks客户端

```Java
sudo apt-get update 
sudo apt-get install python-gevent python-pip
pip install shadowsocks
```

### 二、配置文件的编写（根据此配置文件来连接远程的ss代理服务器）

#### 1、进入 /etc/shadowsocks.json 编辑配置文件

```Java
sudo vim /etc/shadowsocks.json
```
#### 2、配置文件内容

```Java
{
    "server":"1.1.1.1",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"your passwd",
    "timeout":300,
    "method":"aes-256-cfb"
}
```
##### 参数说明

- server 服务端监听地址(IPv4或IPv6)
- server_port 服务端端口，一般为443
- local_address 本地监听地址，缺省为127.0.0.1
- local_port 本地监听端口，一般为1080
- assword 用以加密的密匙
- timeout 超时时间（秒）
- method 加密方法，默认的table是一种不安全的加密，此处首推aes-256-cfb

### 三、运行shadowsocks客户端

#### 1、直接运行ss,该方法必须保证终端窗口一直开着才有效
```Java
sslocal -c /etc/shadowsocks.json
```

#### 2、将ss服务设置成开机自启
把以下命令写到**/etc/rc.local**中

```Java
sudo sslocal -c /etc/shadowsocks/config.json -d start
```

### 四、Chrome插件的配置
  下载安装Proxy SwitchyOmega插件及其使用详情请见：https://www.qcgzxw.cn/2988.html
  


