---
layout: post
title: Linux Shell 常用命令集锦（1）
category: 2017年5月
tags: Shell
keywords: Shell
description: 主要介绍 Linux Shell 常用的命令
---

## 基础

`pwd`：显示当前所在目录名称

`cd`：跳转目录

`ls`：显示文件列表

`cat`：查看文件内容

---

## 关机

立即关机命令：~~`init0`~~、`poweroff`、`half`、`shutdown -h now`

定时关机命令：

- `shutdown -h +30`：30分钟后关机
- `shutdown -h 23:30 '晚上11时30分执行关机操作'`：晚上11时30分将会执行关机操作，末尾的引号内容为关机的提示信息---

---

## 重启

~~`init6`~~、`reboot`、`shutdown -r now`

---

## 用户管理

###  添加用户

`useradd ${option} ${accountName}`，常用可选参数：`-d`指定账户的家目录；`-g`指定账户的用户组。

### 修改用户

`usermod ${option} ${accountName}`，常用可选参数：`-d`指定新的家目录；`-m`将旧家目录的内容移动到新的家目录里，该参数必须与`-d`搭配使用。

### 修改密码

`passwd ${accountName}`

### 删除用户

`userdel ${option} ${accountName}`，常用可选参数：`-r`删除家目录。

## 用户统计

`cat /etc/passwd | wc -l `

### 用户列表

` cat /etc/passwd | cut -d : -f 1` 或者 `awk -F : '{print $1}' /etc/passwd`

---

## 用户组管理

### 添加用户组

`groupadd ${option} ${groupName}`

### 修改用户组

`groupmod ${option} ${groupName}`，常用可选参数：`-n`指定新的组名称。

### 删除用户组

`grouopdel ${option} ${accountName}`

## 用户组统计

`cat /etc/group | wc -l `

### 用户组列表

` cat /etc/group | cut -d : -f 1,3` 或者 `awk -F : '{print $1" - "$3}' /etc/group`

---

## 文件系统与磁盘

### 查看某个（些）文件的空间占用

`du ${option} ${files}`

常用选项：

- `-h`：以`K, M, G, T, P, E, Z, Y`作为计数单位（1024）
- `-s`：只显示最后的目录大小而不显示子目录或者子文件的大小
- `-c`：对多个文件的大小进行求和
- `-a`：显示子目录的文件大小

例子：`du -sh /etc`：统计`/etc`目录的大小；`du -csh /etc/ /root/`：显示`/etc`和`/root`的目录大小以及两者的总大小。

### 查看（文件所在）文件系统的使用情况

`df ${option} ${files}`

常用选项：

- `-h`：以`K, M, G, T, P, E, Z, Y`作为计数单位（1024）
- `-a`：显示所有文件系统

例子：`df -h`：显示文件系统使用情况；`df -h /etc`：查看`/etc`目录所在文件系统的使用情况。

---

## 文件操作

### 清空文件内容

`true > /path/to/file`





|                 |
| --------------- |
| 编写日期：2017-05-23 |
| 发布日期：2017-05-23 |