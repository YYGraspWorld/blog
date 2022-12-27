---
title: 小米AX6S刷OpenWrt和开启OpenClash，及刷回官网固件
toc: true
categories:
  - - 学习 - Linux
  - - 学习 - 代理
tags:
  - OpenWrt
  - 路由器
  - 网络
top: false
date: 2022-12-10 20:09:13
updated: 2022-12-24 21:45:13
---



**简  述:** 闲暇，折腾下新购 小米AX6S，刷下 `OpenWrt`，初次接触记录下。以及开启 OpenClash + clash-rules 的进阶使用

<img src="https://fastly.jsdelivr.net/gh/XMuli/xmuliPic@pic/2022/202212101934164.png" width="80%"/>

<!-- more -->

[TOC]

<br>

> <font color=#D0087E size=4 face="STFangsong">本文初发于 "[**偕臧的小站**](https://ifmet.cn)"，同步转载于此。</font>

<br>

## 背景

> miwifi_rb03_firmware_3e872_1.0.54.bin   [官方 2022.08.24]
> miwifi_rb03_firmware_stable_1.2.7.bin     [官方 2022.03] 可刷机固件，默认开启 telnet 

<br>

## 刷成 OpenWrt

1. 先刷开发版 `miwifi_rb03_firmware_stable_1.2.7.bin` ，默认已开启 Telnet 和 SSH
2. 通过 SN 码计算自己密码，如 36418/K1▇▇▇▇06，密码为 60be9bd0
3. 连接路由器 `telnet 192.168.31.1` (输入 root/60be9bd0)
4. 依次执行命令，执行后无提示
   - nvram set ssh_en=1 && nvram set uart_en=1 && nvram set boot_wait=on && nvram set bootdelay=3 && nvram set flag_try_sys1_failed=0 && nvram set flag_try_sys2_failed=1
   - nvram set flag_boot_rootfs=0 && nvram set "boot_fw1=run boot_rd_img;bootm"
   - nvram set flag_boot_success=1 && nvram commit && /etc/init.d/dropbear enable && /etc/init.d/dropbear start
5. 新开终端页，上传文件`scp .\ax6s-1120\factory.bin root@192.168.31.1:/tmp`
6. 用 ssh 链接路由器 `ssh root@192.168.31.1`；telnet 可以关掉，执行 `mtd -r write /tmp/factory.bin firmware` 刷机。路由器自动重启，默认IP 为 `192.168.6.1` 后，默认账号密码 `root/password` 
7. 输入 ip 进入 openwrt 系统；点击 **“系统 - 备份/升级”** 的 **“刷写新的固件”** 选择 `ax6s-full.bin`  或 `ax6s-mini.bin` 进行刷写固件

<br>

**注：**

- 第二步骤通过 SN 码计算 root 密码：
  - 可在线网站 [miwifi.dev/ssh](miwifi.dev/ssh) 
  - 亦可 [unlock_pwd.py](https://github.com/YangWang92/AX6S-unlock/blob/master/unlock_pwd.py)  脚本计算，`python .\unlock_pwd.py SN码` 

- 此版本发现 `ax6s-full.bin` 实测重启后，WiFi 名称会被重置默认的 bug，而 `ax6s-mini.bin` 不会
- 本篇主要是刷[237176253 ](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=8187405)大佬的固件，新手可参考[此贴](https://www.right.com.cn/forum/thread-8216670-1-1.html)

<br>

## 开启 OpenClash

 简介：[OpenClash](https://github.com/vernesong/OpenClash) ，其有开源内核 foss 和闭源内核 premuium 之分（CFW、OpenClash 都是后者内核），后者通常支持使用规则集 [clash-rules](https://github.com/Loyalsoldier/clash-rules)。初次安装后通常直接运行会失败，LOG 如下，则需要自行安装内核

```bash
2022-12-12 13:26:54 【Dev】版本内核更新失败，请检查网络或稍后再试！
2022-12-12 13:26:52 【Dev】版本内核正在下载，如下载失败请尝试手动下载并上传...
2022-12-12 13:26:50 提示: 检测到内核文件不存在，准备开始下载...
2022-12-12 13:26:49 第二步: 组件运行前检查...
2022-12-12 13:26:49 第一步: 获取配置...
2022-12-12 13:26:49 OpenClash 开始启动...
```

<br>

```bash
cd /etc/openclash/core
wget https://github.com/vernesong/OpenClash/releases/download/Clash/clash-linux-armv8.tar.gz          # OpenWrt首页查看内核平台
tar -zxvf clash-linux-armv8.tar.gz  # 解压为clash文件，tun内核需要下对应文件，后改名clash_tun
chmod 777 clash
chmod 777 clash_tun
# 即可成功开启内核
# 注意优化 dns、ipv6 等操作
```

以下列出 Dev 和 TUN 内核下载地址。

**Dev 内核下载**：https://github.com/vernesong/OpenClash/releases/tag/Clash

**Tun 内核下载**：https://github.com/vernesong/OpenClash/releases/tag/TUN-Premium

**Tun 游戏内核**：https://github.com/vernesong/OpenClash/releases/tag/TUN

<br>

## 刷回官网固件

1. 下载 [小米路由器修复工具](https://www.miwifi.com/miwifi_download.html) 后运行
2. 笔记本网口和路由器 Lan 口网线连接，<u>确保处于同一个网段</u>，选中官网固件
3. 选择 "以太网 -> ip" 后，下一步，此时断电，按住重置按钮直至黄灯闪烁松开，等待几分钟
4. 刷机成功

<br>

## References

- [恩山 AX6S](https://www.right.com.cn/forum/forum.php?mod=forumdisplay&fid=171&filter=typeid&typeid=94)，makr 后续其它固件
- [Redmi AX6S 解锁 SSH、安装 ShellClash、刷入 OpenWRT 教程](https://github.com/lemoeo/AX6S)
