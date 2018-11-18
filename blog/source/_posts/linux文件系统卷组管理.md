---
title: Linux文件系统卷组管理
date: 2018-11-06 08:39:41
tags:
- 学习笔记
mp3: http://link.hhtjim.com/163/27867449.mp3
cover: /image/红叶.jpg
---
### 简介
LVM（Logical Volume Manager），即逻辑卷管理，它是Linux环境下对磁盘分区进行管理的一种机制。  
一般来说，物理磁盘或分区之间是分隔的，数据无法跨盘或分区，而各磁盘或分区的大小固定，重新调整比较麻烦。  
LVM可以将这些底层的物理磁盘或分区整合起来，抽象成容量资源池，以划分成逻辑卷的方式供上层使用，其最主要的功能即是可以在无需关机无需重新格式化（准确地说，原来的部分无需格式化，只格式化新增的部分）的情况下弹性调整逻辑卷的大小。  
![lvm](/image/lvm.jpg "lvm")
### LVM基本术语
前面谈到，LVM是在磁盘分区和文件系统之间添加的一个逻辑层，来为文件系统屏蔽下层磁盘分区布局，提供一个抽象的盘卷，在盘卷上建立文件系统。首先我们讨论以下几个LVM术语：
- 物理存储介质（The physical media）
这里指系统的存储设备：硬盘，如：/dev/hda、/dev/sda等等，是存储系统最低层的存储单元。
- 物理卷（physicalvolume）
物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。
- 卷组（Volume Group）
LVM卷组类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或多个物理卷组成。
- 逻辑卷（logicalvolume）
LVM的逻辑卷类似于非LVM系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home或者/usr等)。
- PE（physical extent）
每一个物理卷被划分为称为PE(Physical Extents)的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是可配置的，默认为4MB。
- LE（logical extent）
逻辑卷也被划分为被称为LE(Logical Extents) 的可被寻址的基本单位。在同一个卷组中，LE的大小和PE是相同的，并且一一对应。  

### 异同
和非LVM系统将包含分区信息的元数据保存在位于分区的起始位置的分区表中一样，逻辑卷以及卷组相关的元数据也是保存在位于物理卷起始处的VGDA(卷组描述符区域)中。VGDA包括以下内容： PV描述符、VG描述符、LV描述符、和一些PE描述符。  
系统启动LVM时激活VG，并将VGDA加载至内存，来识别LV的实际物理存储位置。当系统进行I/O操作时，就会根据VGDA建立的映射机制来访问实际的物理位置。
### 安装LVM
首先确定系统中是否安装了lvm工具：
`# rpm -qa|grep lvm`
如果没有则安装lvm：
`# yum install lvm`
### 具体命令
1. 查看磁盘信息
`# fdisk -l`
2. 创建基于磁盘的物理卷
`# pvcreate /dev/sdb`
>Physical volume "/dev/sdb"successfully created

3. 查看物理卷创建是否成功
`# pvdisplay`
4. 创建data卷组
`# vgcreate data /dev/sdb`
5. 查看卷组是否创建成功
`# vgdisplay`
6. 激活卷组
`# vgchange -a y data`
7. 创建新的物理卷
`# pvcreate /dev/sdc`
>Physical volume "/dev/sdc"successfully created

8. 将新的物理卷添加到现有卷组
`# vgextend data /dev/sdc`
9. 查看卷组信息
`# vgdisplay data`
10. 从现有的卷组中删除一个物理卷
`# vgreduce data /dev/sdc`
删除成功显示
>Removed "/dev/sdc" fromvolume group "data"

11. 创建逻辑卷
创建逻辑卷的命令为lvcreate，分为两种：
- 创建指定大小的逻辑卷LV
`# lvcreate -L200M -n data001 data`
>Logical volume"data001” created

- 创建卷组全部大小的逻辑卷LV
如果希望创建一个使用全部卷组的逻辑卷，则需要首先通过vgdisplay察看该卷组的Total PE数，然后在创建逻辑卷时指定
`# vgdisplay data`
`# lvcreate -l127999 -n dataall data`
- 创建剩余空间所有大小的LV
查看LV剩余空间的大小：
`# vgdisplay data`
创建分区：
`# lvcreate -l127949 -n data003 data`
>Logical volume "data003” created

- 在现有的空间中添加500M
`# lvextend -L+500M /dev/data/data001 ---向LV01添加500M的空间`
- 在现有的空间中添加到分区的总大小为2G
`# lvextend -L2G /dev/data/data002`
- 扩容文件系统
`# resize2fs /dev/data/data001`

### lvm创建销毁流程
#### LV建立流程：
1. 建立PV
2. 建立VG，将PV加入到VG中。
3. 建立LV，并设置LV大小。
4. 格式化LV或LP成你需要的文件系统。  

#### LV删除流程：
1. umountFS
2. 删除LV。
3. 将PV从所在的VG中删除。
4. 删除VG。  

#### LV扩容流程：
1. 建立PV
2. 将PV加入VG
3. 修改lv大小
4. 扩容文件系统

### 卷管理常用命令集合：

|    |查看显示     |创建     |删除   |扩容       |激活     |扫描查找|
|----|------------|--------|-------|---------|---------|-------|
|LV  |lvdisplay  |lvcreate |lvremove|lvextend|lvchange|lvscan|
|PV  |pvdisplay  |pvcreate |pvremove|        |pvchange|pvscan|
|VG  |vgdisplay  |vgcreate |vgremove|vgextend|vgchange|vgscan|
### 备注：
1. 请将创建的lvm逻辑卷加入/etc/fstab文件，使其开机自动挂载
2. 一个问题：
Linux resize2fs: Bad magic number in super-block错误的解决方法：
确认文件系统是xfs:
`# cat /etc/fstab | grep centos-home`
>/dev/mapper/centos-home /home                   xfs     defaults        0 0

- xfs用以下命令来扩磁盘空间:
`# xfs_growfs /dev/mapper/centos-home`
