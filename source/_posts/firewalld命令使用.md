---
title: firewalld命令使用
date: 2017-02-08 22:20:40
tags: firewalld
---
### 1、firewalld的基本使用  
启动： `systemctl start firewalld`  
查看状态： `systemctl status firewalld`  
停止： `systemctl disable firewalld`  
禁用： systemctl stop firewalld  

### 2.配置firewalld-cmd

查看版本： `firewall-cmd --version`  
查看帮助： `firewall-cmd --help`  
显示状态： `firewall-cmd --state`  
查看所有打开的端口： `firewall-cmd --zone=public --list-ports`  
更新防火墙规则： `firewall-cmd --reload`  
查看区域信息:  `firewall-cmd --get-active-zones`  
查看指定接口所属区域： `firewall-cmd --get-zone-of-interface=eth0`  
拒绝所有包：`firewall-cmd --panic-on`  
取消拒绝状态： `firewall-cmd --panic-off`  
查看是否拒绝： `firewall-cmd --query-panic`  

### 开启或者关闭一个端口
添加  
`firewall-cmd --zone=public --add-port=80/tcp --permanent`    （`--permanent`永久生效，没有此参数重启后失效）  
重新载入  
`firewall-cmd --reload`  
查看  
`firewall-cmd --zone= public --query-port=80/tcp`  
删除  
`firewall-cmd --zone= public --remove-port=80/tcp --permanent`  