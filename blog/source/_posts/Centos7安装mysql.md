---
title: Centos7安装mysql
date: 2018-11-03 16:59:41
tags:
- 学习笔记
mp3: http://link.hhtjim.com/163/29567193.mp3
cover: /image/mysql_logo.jpg
---
### 简介
MySQL是一个关系型数据库管理系统，使用的 SQL 语言是用于访问数据库的最常用标准化语言。  
MySQL 软件采用了双授权政策，分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。  
在服务器上部署mysql数据库服务，通常服务器操作系统为CentOS  
本篇记录一下在CentOS7上安装mysql的常规步骤
![mysql](/image/mysql_logo.jpg "MySQL")
### 安装
1. 下载mysql的repo源  
`# wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm`
2. 安装mysql-community-release-el7-5.noarch.rpm包  
`# rpm -ivh mysql-community-release-el7-5.noarch.rpm`  
安装这个包后，会获得两个mysql的yum repo源：  
/etc/yum.repos.d/mysql-community.repo  
/etc/yum.repos.d/mysql-community-source.repo   
检查mysql源是否安装成功  
`# yum repolist enabled | grep "mysql.*-community.*"`  
可以修改/etc/yum.repos.d/mysql-community.repo源，改变默认安装的mysql版本。  
比如要安装5.6版本，将5.7源的enabled=1改成enabled=0。  
然后再将5.6源的enabled=0改成enabled=1即可。  
3. 安装mysql  
安装mysql服务端    
`# yum install mysql-server`  
安装mysql客户端  
`# yum install mysql`
4. 启动MySQL服务  
`# systemctl start mysqld`  
或者  
`# service mysqld start`
5. 查看MySQL的启动状态  
`# systemctl status mysqld`  
或者  
`# service mysqld status`
6. 停止MySQL服务
`# systemctl stop mysqld`    
或者  
`# service mysqld stop`
7. 开机启动  
`# systemctl enable mysqld`
`# systemctl daemon-reload`

### 默认配置文件路径  
- 配置文件：/etc/my.cnf  
- 日志文件：/var/log//var/log/mysqld.log  
- 服务启动脚本：/usr/lib/systemd/system/mysqld.service  
- socket文件：/var/run/mysqld/mysqld.pid  

### 修改密码及权限设置
1. 查看mysql默认临时密码  
`# cat /var/log/mysqld.log | grep pass`
2. 连接mysql   
`# mysql -u root -p  `
然后输入临时密码
3. 修改密码  
用临时密码连上数据库后发现执行命令报错：  
所以要修改mysql生成的随机密码为自己的密码  
`mysql> alter user user() identified by "your_passwd”;`
4. 修改mysql远程访问用户及权限：  
    默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须修改root允许远程连接，或者添加一个允许远程连接的帐户:  
```
mysql > GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'passwd' WITH GRANT OPTION;  
mysql > GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'passwd' WITH GRANT OPTION;  
mysql > flush privileges;
```
### 注意事项  
安装前请检查系统版本为CentOS7  
CentOS6用户请第一步执行  
`# wget https://dev.mysql.com/get/mysql80-community-release-el6-1.noarch.rpm`
第二步安装也将安装文件改为el6  
`# rpm -ivh mysql80-community-release-el6-1.noarch.rpm`  
后面步骤基本一致  
