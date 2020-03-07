---
title: Linux：如何查看内核版本和系统版本？
date: 2020-03-06 23:34:58
categories:
  - [服务器]
  - [Linux]
tag:
    - Linux
    - 服务器
---

- 当你想要给Linux系统进行安装软件的时候，很多时候都要先查看发行商和版本

- 有些软件甚至还对Linux的内核版本有要求，否则无法正常安装

- 本文主要介绍了多种查看Linux内核和系统版本的方法

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

- 诸如 RHEL、Debian、openSUSE、Arch Linux 这几种主流发行版来说，它们各自拥有不同的包管理器来管理系统上的软件包

- 如果不知道所使用的是哪一个发行版的系统，在软件包安装的时候就会无从下手

- 而且由于大多数发行版都是用 systemd 命令而不是 SysVinit 脚本，在重启服务的时候也难以执行正确的命令。

# 1.查看内核版本

## 1.1 uname 命令

> uname（unix name） 是一个打印系统信息的工具，内容包括内核名称、版本号、系统详细信息以及所运行的操作系统等等。

```shell
$ uname -a
Linux centos 3.10.0-327.el7.x86_64 #1 SMP Thu Nov 19 22:10:57 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
```

以上运行结果说明正在使用的系统版本是 centos，内核版本 3.10.0-327.el7.x86_64

## 1.2 查看 /proc/version 文件

> 这个文件记录了 Linux 内核的版本、用于编译内核的 gcc 的版本、内核编译的时间，以及内核编译者的用户名。

```shell
$ cat /proc/version
Linux version 2.6.32-220.23.1.el6.x86_64 (mockbuild@c6b5.bsys.dev.centos.org) (gcc version 4.4.6 20110731 (Red Hat 4.4.6-3) (GCC) ) #1 SMP Mon Jun 18 18:58:52 BST 2012
```

# 2.查看系统版本

## 2.1 lsb_release 命令

> LSB（Linux Standard Base）能够打印发行版的具体信息，包括发行版名称、版本号、代号等。

```shell
# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description: Ubuntu 16.04.3 LTS
Release: 16.04
Codename: xenial
```

- 这个命令适用于所有的 Linux 发行版，包括 Redhat、SuSE、Debian… 等发行版。

- 有的系统中默认并没有安装 lsb_release，需要安装。

下面介绍一下 CentOS 系统中安装方法。

首先查找 lsb_release 安装包：

```Shell
$ yum provides lsb_release
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * extras: mirrors.btte.net
 * updates: mirrors.btte.net
redhat-lsb-core-4.1-27.el7.centos.1.i686 : LSB Core module support
源    ：base
匹配来源：
文件名    ：/usr/bin/lsb_release
```

安装：

`$ sudo yum install -y redhat-lsb-core`

## 2.2 cat /etc/issue

> 此命令适用于所有的 Linux 发行版。

```
$ cat /etc/issue
Ubuntu 16.04.1 LTS \n \l

```

## 2.3 cat /etc/redhat-release

> 这种方法只适合查看 Redhat 系的 Linux，如：CentOS。

```
$ cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)
```

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)








