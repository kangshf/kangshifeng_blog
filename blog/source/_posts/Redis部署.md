---
title: Redis部署
date: 2018-11-03 20:41:41
tags:
- 学习笔记
mp3: http://link.hhtjim.com/163/29567189.mp3
cover: /image/redis_logo.jpg
---
### 简介
Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。  
Redis是一个开源（BSD许可），内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。它支持字符串、哈希表、列表、集合、有序集合，位图，hyperloglogs等数据类型。  
内置复制、Lua脚本、LRU收回、事务以及不同级别磁盘持久化功能，同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动分区。
![Redis](/image/redis_logo.jpg "Redis")
### 安装
1. 下载redis压缩包  
`# wget http://download.redis.io/releases/redis-3.0.6.tar.gz`
2. 解压压缩包  
`# tar -xvf redis-3.0.6.tar.gz`
3. 编译源码并安装  
`# make`    
`# make install`  
- 注：如发生以下错误  
zmalloc.h:50:31: 致命错误：jemalloc/jemalloc.h：没有那个文件或目录  
执行命令：  
`# make MALLOC=libc`
4. 复制redis配置文件到指定目录  
`# cp redis.conf /usr/local/etc`
5. 编辑redis.conf  
将daemonize属性改为yes  
注掉#bind 127.0.0.1  
添加权限鉴定requirepass passwd  
6. 编辑redis服务脚本
`# vi /etc/init.d/redis`
```
# chkconfig: 2345 10 90
# description: Start and Stop redis

PATH=/usr/local/bin:/sbin:/usr/bin:/bin
REDISPORT=6379
EXEC=/usr/local/bin/redis-server
REDIS_CLI=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_6379.pid
CONF="/usr/local/etc/redis.conf"
case "$1" in
        start)
                if [ -f $PIDFILE ]
                then
                        echo "$PIDFILE exists, process is already running or crashed."
                else
                        echo "Starting Redis server..."
                        $EXEC $CONF
                fi
                if [ "$?"="0" ]
                then
                        echo "Redis is running..."
                fi
                ;;
        stop)
                if [ ! -f $PIDFILE ]
                then
                        echo "$PIDFILE not exists, process is not running."
                else
                        PID=$(cat $PIDFILE)
                        echo "Stopping..."
                       $REDIS_CLI -p $REDISPORT  SHUTDOWN
                        sleep 2
                       while [ -x $PIDFILE ]
                       do
                                echo "Waiting for Redis to shutdown..."
                               sleep 1
                        done
                        echo "Redis stopped"
                fi
                ;;
        restart|force-reload)
                ${0} stop
                ${0} start
                ;;
        *)
               echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2
                exit 1
esac
```
7. redis启停等操作   
启动  
`# service redis start`
停止   
`# service redis stop`
开机自启     
`# chkconfig redis on`
- 注:停止redis实例的其他几种方法
`# /usr/local/redis/bin/redis-cli shutdown`  
或者  
`# pkill redis-server`  
- 注:让redis开机自启的另一种方法
`# vim /etc/rc.local`
加入:/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis-conf

### 相关文件介绍  
接下来我们看看/usr/local/redis/bin目录下的几个文件是什么
- redis-benchmark：redis性能测试工具
- redis-check-aof：检查aof日志的工具
- redis-check-dump：检查rdb日志的工具
- redis-cli：连接用的客户端
- redis-server：redis服务进程
