---
title: shell
date: 2017-08-31 11:35:41
tags:
---

vps 设置系统定时重启， ssserver 开机启动

首先先编写开启启动ssserver 脚本
<!--more-->
```
sudo ssserver -p 443 -k password -m rc4-md5 --user nobody -d start
```
P 服务端口
K密码m 加密方式 我们选择 比较快速的rc4-md5 放弃了aes-256
user 选择 nobody 然后 把这段话保存 为 shadowsocks.sh

将其放在 /opt 目录下

打开开机自启配置文件 `vim /etc/rc.local`

将 `/opt/./shadowsocks.sh` 这句放在最下方 保存退出
执行 `source ~/etc/rc.local` 让配置生效

crontab -l 查看当前在执行的 定时任务

crontab -e 编辑定时任务

在最下方加 `0 13 * * * /sbin/reboot`

每天的13点重启vps， 因为vps 是美国服务器 有时差 所以13点应该是中国的凌晨 刚刚好

然后 `*/5 * * * * ./shadowsocks.sh`

每5分钟就检测一次 ssserver 如果没有进程 就启动一遍，有就不执行

保存退出

重启vps

登录 vps 输入 top

查看到 ssserver 服务已经启动

完结撒花