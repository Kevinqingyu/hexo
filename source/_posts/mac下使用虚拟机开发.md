---
title: mac下使用虚拟机开发
date: 2019-06-24 16:22:55
tags: [ruby rails 虚拟机 vagrant]
---

![](https://rails365.oss-cn-shenzhen.aliyuncs.com/uploads/photo/image/1543/2019/d2b5ca33bd970f64a6301fa75ae2eb22.png)

### 背景

在过去的几年间一直使用MacBookPro(2015款13寸最低配，2013款15寸最高配)笔记本开发，使用了大概4年多的时间，非常的熟悉mac下开发。由于性能问题在换到黑苹果之后装载了最新的操作系统，电脑上可能会装有十几个ruby项目以及一部分的vue项目等等，在mac最新系统(10.14.4)下对老版本的ruby和gems的支持有些许问题给我带来了很大困扰，不同的环境交错复杂时间长了很难维护，遂产生了使用虚拟机或者docker开发 的想法。
<!--more-->
### 思考

这篇记录不讨论mac 虚拟机 和docker开发 客观的优缺点，而是带着主观来说优缺点可能某些优点在我使用就不是优点了(或者说不适合我)，所以我希望能达到`和mac下开发的体验保持一致`，`速度快隔离性好`，之前考虑过docker由于docker偏向于一个项目一个docker镜像这对我来说是不可接受的，因为我项目很多镜像搞太多还单独独立于自己的容器中对我来说不方便，最后经由`同事`推荐使用Vagrant + VirtualBox(因为免费) + ubuntu18.04镜像。

### 准备

Vagrant 支持 VirtualBox、HyperV、VMWare 等虚拟机软件，
我安装的是 VirtualBox(因为其他的要么新系统装不上要么收费比如VMWare)

### 安装

下载VirtualBox 安装这个比较简单就和装普通的软件一样，下载安装包安装也无需配置甚至都不用打开。

然后安装Vagrant

```
brew install vagrant
```
安装好之后我们来安装ubuntu18.04 的服务器版本(体积小)
[更多系统镜像点击这里](http://www.vagrantbox.es/)
这个地址的镜像都比较老旧了所以这次我去官网查到了最新的镜像
[ubuntu18.04LTS](https://app.vagrantup.com/generic/boxes/ubuntu1804)

为了方便管理虚拟机的开发环境 共享文件夹还有配置文件等等，我这里在mac 的home目录下新建了一个共享文件夹就叫`vagrant` home目录下执行
```bash
mkdir vagrant
```

创建好之后`cd`进到目录中创建虚拟机配置文件 `Vagrantfile`

```bash
cd vagrant
vim Vagrantfile
```
填入如下内容

```conf
Vagrant.configure(2) do |config|
  # 选择系统镜像
  config.vm.box = "generic/ubuntu1804"
  config.vm.box_check_update = false
  # 时区设置
  if Vagrant.has_plugin?("vagrant-timezone")
    config.timezone.value = "Asia/Shanghai"
  end
  # 固定IP 的私有网络 外部不可访问
  config.vm.network "private_network", ip: "192.168.50.4"
  # 这是对于虚拟机共享文件夹的配置
  config.vm.synced_folder ".", "/vagrant", type: "nfs", disabled: true
  config.vm.synced_folder "~/vagrant", "/vagrant", type: "nfs", mount_options: ["nolock", "vers=3", "udp", "noatime", "actimeo=1"]

  # 虚拟机配置 内存 cpu 等
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048
    vb.cpus = 2
  end
end
```
保存退出

这些配置都配置好之后就可以在vagrant文件夹下使用命令

```
vagrant up
```

来启动虚拟机并会自动从网上下载ubuntu18.04的镜像
这个过程我们需要等待。

启动好了之后可以通过ssh登录到虚拟机并且 可以额设置通过ssh key登录 就和一台普通的服务器是一样的

也可以在mac写入alias

在 `~/.zshrc`(如果是bash请加入 ~/.bashrc) 中加入
`alias va='ssh vagrant@ubuntu.com'`
这样以后就可以快速的使用 `va` 命令进入虚拟机, 当你在虚拟机启动了某项目 让你访问 `0.0.0.0:3000`端口时 你可以通过之前的配置访问 `192.168.50.4:3000`来访问也可以通过修改
`/etc/hosts`

加入

```
192.168.50.4    ubuntu.com
```

这样你就可以在mac浏览器使用 `ubuntu.com:3000` 来访问虚拟机的 `0.0.0.0:3000`地址

### 虚拟机内部配置

我在mac上习惯了使用zsh + oh-my-zsh来工作，那么在ubuntu这里也是一样可以使用的甚至当在虚拟机中使用了同样的样式之后你都分辨不出你是在虚拟机中或是在mac中，所以还是建议使用不同的主题或者带有目录前缀的这种主题，这样一看主题你就知道你在什么位置了。

在ubuntu中也是可以访问mac 的比如 现在mac 的网段就是`192.168.50.1` 也可以用同样的手法修改 ubuntu 的
`etc/hosts` 将mac改为
```
192.168.50.4  mac.com
```

这样在ubuntu下跑的服务数据库连接可以使用
```
database:
  name: root
  password:
  host: mac.com
  port: 3306
```
这样的配置来连接mac下的数据库(前提是mac下的数据库先开启允许远端访问哦)

然后剩下的就是在虚拟机这里搭建你所需要的开发环境，将代码文件夹放置于共享文件夹中，mac上使用你喜欢的编辑器编辑共享文件夹内的代码，而虚拟机负责运行起代码，mac来提交代码。

### 备份镜像

在你将开发环境和配置都搭建好之后可以将自己搭建好的环境作为一个镜像输出出来，以后自己新建另一个虚拟机或是分配给新来的开发伙伴直接使用免于重新搭建都是可以的。
使用命令

```
vagrant package --output boxname.box
```
就可以将现在运行的环境打包为一个镜像

如果下次想要使用可以使用

```
vagrant add box box 名字 box地址
```

虚拟机这里基本就是这些内容了，但是如果使用ruby on rails开发web项目那么还有几点稍微补充一下，我这里使用rvm + ruby 开发，其他都是比较正常的按照ruby-china 的教程你就可以搭建出开发环境这里掠过，要说的一点事首先rails console 的中文问题

需要使用一条命令解决

```
export LC_CTYPE=en_US.UTF-8
```

然后是rails开发在修改了controller代码时 rails(spring)无法获取到代码的修改事件从而导致你需要重启项目才能重新加载变更内容(view部分不受影响)

rails4 项目可以采用安装插件的方式解决
在vagrant共享文件夹下执行

```
vagrant plugin install vagrant-vbguest
vagrant plugin install vagrant-librarian-chef-nochef
```
安装好插件之后 执行

```
vagrant reload
```
重启虚拟机

但是这个方法亲测在rails 5.2中是无效的 不知道rails5其他版本是否也是无效，由于这个原因导致要么使用vim编程要么就开发的时候修改了控制器就要重启项目，如果真的是这样就已经和mac下体验差距过大了，还好现在vscode编辑器支持 remote-ssh编程，也就是说在mac下使用vscode编辑器并安装remote-ssh插件之后可以远程虚拟机中编辑代码，实测在连接项目后延迟几乎感觉不到(可能是由于就在本地的缘故)，几乎和本地开发体验一致，并且也保证了虚拟机的独立性从此算是虚拟机开发圆满成功。

![](https://rails365.oss-cn-shenzhen.aliyuncs.com/uploads/photo/image/1545/2019/d2b5ca33bd970f64a6301fa75ae2eb22.png)


### 总结

在mac下虚拟机开发根据自己的喜好添加，甚至如果自己的配置够高可以 尝试搭建多个虚拟机来模拟集群，由于各自独立的环境也可以模拟多台服务器场景。还有更多玩法等待你去挖掘。
