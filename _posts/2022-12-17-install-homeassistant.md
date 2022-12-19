---
title: 使用树莓派通过 Docker 安装 Home Assistant
author: sidxul
date: 2022-12-17 11:36:59 +0800
categories: [自我学习, 自力更生]
tags: [树莓派, Docker, Home Assistant, 米家, HomeKit, HomePod]
---

## 序

几年前从朋友那里转租房子的时候扣留了他的两个小米智能插座，之后陆续又买了一些小米生态链下的智能产品，开启了智能家居的生活。由于一直无法~~完整~~顺心接入 HomePod Mini 中，查了一些资料后，遂即入手一套树莓派设备，开始折腾。起初觉得 Homebridge 简洁方便，容易上手。无奈插件质量问题，提交 issue 至少大半年时间仍未解决，开始转投 Home Assistant 怀中。<br>

我并不会编译程序，对各种代码更是一窍不通。网上教程繁多，花了一些时间踩了一些坑，归纳了一份适合我自己的操作方法。<br>

目前借助 Home Assistant 使用 Siri 控制米家产品未出现异常。
<hr>

## 安装树莓派 OS

我使用的是官方 [Raspberry Pi Imager](https://downloads.raspberrypi.org/imager/imager_latest.dmg) 工具安装最新 [Raspberry Pi OS with desktop 64bit](https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit) 版本。Raspberry Pi Imager 可以提前进行打开 ssh 、连接 WIFI 等高级设置，方便后续无需使用显示器操控。<br>

> 在搜索文档时，发现有些人使用另一款好用的工具 [balenaEtcher](https://www.balena.io/etcher/) 写入镜像。有方法配置 WIFI如下：
> 
> * 在 boot 分区新建 CONFIG 文件夹
> * 在 CONFIG 文件夹在新建 network 文件夹
> * 在 network 文件夹创建一个 my-network 文件，参考内容如下：
>
>```plaintext
>[connection]
>id=my-network
>uuid=<UUID>
>type=802-11-wireless
>
>[802-11-wireless]
>mode=infrastructure
>ssid=<WIFISSID>
># Uncomment below if your SSID is not broadcasted
>#hidden=true
>
>[802-11-wireless-security]
>auth-alg=open
>key-mgmt=wpa-psk
>psk=<WIFIPASSWORD>
>
>[ipv4]
>method=auto
>
>[ipv6]
>addr-gen-mode=stable-privacy
>method=auto
>```
>
> * 生成 Version 4 [UUID](https://www.uuidgenerator.net)
> 
> 可能出现由于时区问题导致无法安装，在 CONFIG 文件夹新建文件 timesyncd.conf 内容如下：
>
>```plaintext
>[Time]
>NTP=cn.ntp.org.cn hk.ntp.org.cn
>FallbackNTP=0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org
>```
>
> **未验证** 在 boot 分区新建或 touch 一个 ssh 空白文件可打开 ssh 服务
>
> ```console
>touch ssh
>```

安装完系统的几个常规操作如下：

* 更新系统

  ```console
  sudo apt-get update && sudo apt-get upgrade
  ```

* ssh 终端开启 VNC

  ```console
  sudo raspi-config
  ```

* 安装谷歌拼音输入法

  ```console
  sudo apt-get install fcitx fcitx-googlepinyin fcitx-module-cloudpinyin
  ```

## 安装 Docker

* 安装 Docker

  ```console
  curl -sSL https://get.docker.com | sh
  ```

* 启动 Docker

  ```console
  sudo systemctl enable docker
  ```

  ```console
  sudo systemctl start docker
  ```

* 将当前用户加入 Docker 组

  ```console
  sudo usermod -aG docker pi
  ```

* 退出当前终端并重新登录，并执行以下命令测试 Docker 是否安装正确

  ```console
  docker run --rm hello-world
  ```

## 安装 Home Assistant

* 安装 Home Assistant

  ```console
  sudo docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Asia/Shanghai \
  -v /home/pi/ha:/config \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
  ```

* 安装完毕后重启 Home Assistant

  ```console
  docker restart homeassistant
  ```

## 安装 HACS

* cd 到 home Assistant 文件夹目录 ha

```console
wget -O - https://hacs.vip/get | sudo bash -
```

* 配置过程

略

* 通过 HACS 安装温湿度计

略
* 配置温湿度计（ha目录下）

```console
sudo nano configuration.yaml
```

添加以下信息：

```plaintext
# Passive BLE Monitor integration
ble_monitor:
```
* 通过 HACS 安装 hass-xiaomi-miot 插件

在 HACS 中搜索 Xiaomi Miot Auto 安装并配置。

<hr>

*1. 2022-12-18 15:04:36 增加安装树莓派 OS*

*1. 2022-12-17 12:28:33 添加「序」。*

*2. 2022-12-13 17:31:07 创建。*