---
title: CentOS7系统下MongoDB部署
date: 2018-11-05 18:59:41
tags:
- 学习笔记
mp3: http://link.hhtjim.com/163/486814412.mp3
cover: /image/mongodb.jpg
---
### 简介
MongoDB（来自于英文单词“Humongous”，中文含义为“庞大”）是可以应用于各种规模的企业、各个行业以及各类应用程序的开源数据库。  
作为一个适用于敏捷开发的数据库，MongoDB的数据模式可以随着应用程序的发展而灵活地更新。  
与此同时，它也为开发人员 提供了传统数据库的功能：二级索引，完整的查询系统以及严格一致性等等。  
MongoDB能够使企业更加具有敏捷性和可扩展性，各种规模的企业都可以通过使用MongoDB来创建新的应用，提高与客户之间的工作效率，加快产品上市时间，以及降低企业成本。
MongoDB是专为可扩展性，高性能和高可用性而设计的数据库。  
它可以从单服务器部署扩展到大型、复杂的多数据中心架构。  
利用内存计算的优势，MongoDB能够提供高性能的数据读写操作。  
MongoDB的本地复制和自动故障转移功能使您的应用程序具有企业级的可靠性和操作灵活性。
![mongodb](/image/mongodb.jpg "mongodb")
### 安装
1. 下载MongoDB安装包  
`# wget http://downloads.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.6.3.tgz`
- 注:CentOS6下载
`# wget http://downloads.mongodb.org/linux/mongodb-linux-x86_64-rhel62-3.2.8.tgz`
2. 解压程序压缩包  
`# tar xf mongodb-linux-x86_64-3.6.3.tgz`
3. 创建程序安装目录  
`# mkdir -p /usr/local/mongodb`  
`# cd /usr/local/mongodb/`   
`# mkdir -p bin conf log data`
4. 拷贝二进制程序到安装目录  
`# cd -`  
`# cp mongodb-linux-x86_64-3.6.3/bin/* /usr/local/mongodb/bin`
5. 配置环境变量  
`# vi /etc/profile.d/mongodb_env.sh`  
写入以下内容：
> #!/bin/bash
export MONGODB_HOME=/usr/local/mongodb
PATH=$PATH:$MONGODB_HOME/bin

6. 使环境变量生效：  
`# source /etc/profile`
7. 修改mongodb配置文件:
`# vi /usr/local/mongodb/conf/mongodb.conf`
> ##content
systemLog:
&#8194;&#8194;destination: file
&#8194;&#8194;logAppend: true
&#8194;&#8194;path: /data1/application/mongodb/log/mongodb.log
>#Where and how to store data.
storage:
&#8194;&#8194;engine: wiredTiger
&#8194;&#8194;dbPath: /data1/application/mongodb/data
&#8194;&#8194;directoryPerDB: true
&#8194;&#8194;journal:
&#8194;&#8194;&#8194;&#8194;enabled: true
>#how the process runs
processManagement:
&#8194;&#8194;fork: true
&#8194;&#8194;pidFilePath: /data1/application/mongodb/log/mongodb.pid
>#network interfaces
net:
&#8194;&#8194;port: 27017
&#8194;&#8194;bindIp: 0.0.0.0
&#8194;&#8194;maxIncomingConnections: 2000
security:
&#8194;&#8194;authorization: disabled
&#8194;&#8194;javascriptEnabled: false
setParameter:
&#8194;&#8194;failIndexKeyTooLong: false

8. mongodb启停
启动：
`mongod -f conf/mongodb.conf`
关闭：
`mongod -f conf/mongodb.conf  --shutdown`
在数据库中关闭数据库的方法:
`# mongo`
```
MongoDB shell version: 3.2.8
connecting to: test
> db.shutdownServer()
shutdown command only works with the admin database; try 'use admin'
> use admin
> db.shutdownServer()
server should be down...
```
>mongod进程收到SIGINT信号或者SIGTERM信号，会做一些处理
- 关闭所有打开的连接
- 将内存数据强制刷新到磁盘
- 当前的操作执行完毕
- 安全停止
&#8194;&#8194;切忌kill -9
&#8194;&#8194;数据库直接关闭，数据丢失，数据文件损失，修复数据库（成本高，有风险）

9. 用脚本管理mongodb服务
```
#!/bin/bash
#
# chkconfig: 2345 80 90
# description:mongodb
#################################
MONGODIR=/usr/local/mongodb
MONGOD=$MONGODIR/bin/mongod
MONGOCONF=$MONGODIR/conf/mongod.conf
InfoFile=/tmp/start.mongo
. /etc/init.d/functions

status(){
  PID=`awk 'NR==2{print $NF}' $InfoFile`
  Run_Num=`ps -p $PID|wc -l`
  if [ $Run_Num -eq 2 ]; then
    echo "MongoDB is running"
  else
    echo "MongoDB is shutdown"
    return 3
  fi
}

start() {
  status &>/dev/null
  if [ $? -ne 3 ];then
    action "启动MongoDB,服务运行中..."  /bin/false
    exit 2
  fi
  sudo su - mongod -c "$MONGOD -f $MONGOCONF" >$InfoFile 2>/dev/null
  if [ $? -eq 0 ];then
    action "启动MongoDB"  /bin/true
  else
    action "启动MongoDB"  /bin/false
  fi
}

stop() {
  sudo su - mongod -c "$MONGOD -f $MONGOCONF --shutdown"  &>/dev/null
  if [ $? -eq 0 ];then
    action "停止MongoDB"  /bin/true
  else
    action "停止MongoDB"  /bin/false
  fi
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart)
    stop
    sleep 2
    start
    ;;
  status)
    status
    ;;
  *)
    echo $"Usage: $0 {start|stop|restart|status}"
    exit 1
esac
```
10. 开机自启
`# chkconfig redis on`
### 几个建议
- mongodb建议以非root用户启动
建议创建名为mongodb的用户，修改mongodb文件属主，用mongodb用户启动
- 禁用THP
Transparent Huge Pages (THP)，通过使用更大的内存页面，可以减少具有大量内存的机器上的缓冲区（TLB）查找的开销。
但是，数据库工作负载通常对THP表现不佳，因为它们往往具有稀疏而不是连续的内存访问模式。
您应该在Linux机器上禁用THP，以确保MongoDB的最佳性能。
以root用户做以下操作：
```
cat >> /etc/rc.local <<'EOF'
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
EOF
```
- 使用XFS
在Linux上运行MongoDB时，官方建议使用Linux内核版本2.6.36或更高版本，使用XFS或EXT4文件系统。
如果可能，最好使用XFS，因为它通常与MongoDB表现更好。
使用WiredTiger存储引擎，强烈建议使用XFS，以避免在使用EXT4与WiredTiger时可能发生的性能问题。
