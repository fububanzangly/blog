---
title: 安装grid
date: 2018-01-16 16:33:12
tags: Rac
---
# Oracle Rac 11g安装文档
## Rac介绍
RAC是real application clusters的缩写，译为“实时应用集群”， 是Oracle新版数据库中采用的一项新技术，是高可用性的一种，也是Oracle数据库支持网格计算环境的核心技术。
<!-- more --> 
### 优点
Oracle RAC主要支持Oracle9i、10g、11g，12C版本，可以支持24 x 7 有效的数据库应用系统，在低成本服务器上构建高可用性数据库系统，并且自由部署应用，无需修改代码。
在Oracle RAC环境下，Oracle集成提供了集群软件和存储管理软件，为用户降低了应用成本。当应用规模需要扩充时，用户可以按需扩展系统，以保证系统的性能。
(1)多节点负载均衡;
(2)提供高可用：故障容错和无缝切换功能，将硬件和软件错误造成的影响最小化;
(3)通过并行执行技术提高事务响应时间----通常用于数据分析系统;
(4)通过横向扩展提高每秒交易数和连接数----通常对于联机事务系统;
(5)节约硬件成本，可以用多个廉价PC服务器代替昂贵的小型机或大型机，同时节约相应维护成本;
(6)可扩展性好，可以方便添加删除节点，扩展硬件资源。
### 缺点
(1)相对单机，管理更复杂，要求更高;
(2)在系统规划设计较差时性能甚至不如单节点;
(3)可能会增加软件成本(如果使用高配置的pc服务器，Oracle一般按照CPU个数收费)。
在Oracle9i之前，RAC的名称是OPS (Oracle parallel Server)。RAC 与 OPS 之间的一个较大区别是，RAC采用了Cache Fusion(高速缓存合并)技术。在 OPS 中，节点间的数据请求需要先将数据写入磁盘，然后发出请求的节点才可以读取该数据。使用Cache fusion时，RAC的各个节点的数据缓冲区通过高速、低延迟的内部网络进行数据块的传输。
### 组件
在一个应用环境当中，所有的服务器使用和管理同一个数据库，目的是为了分散每一台服务器的工作量，硬件上至少需要两台以上的服务器，而且还需要一个共享存储设备。同时还需要两类软件，一个是集群软件，另外一个就是Oracle数据库中的RAC组件。同时所有服务器上的OS都应该是同一类OS,根据负载均衡的配置策略，当一个客户端发送请求到某一台服务的listener后，这台服务器根据我们的负载均衡策略，会把请求发送给本机的RAC组件处理也可能会发送给另外一台服务器的RAC组件处理，处理完请求后，RAC会通过集群软件来访问我们的共享存储设备.
逻辑结构上看，每一个参加集群的节点有一个独立的instance（数据库实例），这些instance访问同一个数据库。节点之间通过集群软件的通讯层（communication layer）来进行通讯。同时为了减少IO的消耗，存在了一个全局缓存服务，因此每一个数据库的instance，都保留了一份相同的数据库cache
RAC中的特点是：
每一个节点的instance都有自己的SGA
每一个节点的instance都有自己的background process
每一个节点的instance都有自己的redo logs
每一个节点的instance都有自己的undo表空间
所有节点都共享一份datafiles和controlfiles
还提出了一个缓存融合的技术(Cache fusion)
目的有两个
01.保证缓存的一致性
02.减少共享磁盘IO的消耗
因此在RAC环境中多个节点保留了同一份的DB CACHE
缓存融合（Cache fusion）工作原理：
****************************************
01.其中一个节点会从共享数据库中读取一个block到db cache中
02.这个节点会在所有的节点进行交叉db block copy
03.当任何一个节点缓存被修改的时候，就会在节点之间进行缓存修改
04.为了达到存储的一致最终修改的结果也会写到磁盘上
ClusterWare组件
*******************
四种Service
Crsd -集群资源服务
Cssd - 集群同步服务
Evmd - 事件管理服务
oprocd - 节点检测监控
三类Resource
VIP - 虚拟IP地址(Virtual IP)
OCR - Oracle Cluster Registry(集群注册文件),记录每个节点的相关信息
Voting Disk - Establishes quorum (表决磁盘),仲裁机制用于仲裁多个节点向共享节点同时写的行为，这样做是为了避免发生冲突。
RAC的组件
************
提供过了额外的进程，用来维护数据库
LMS - Global Cache Service Process 全局缓存服务进程
LMD - Global Enqueue Service Daemon 全局查询服务守护进程
LMON - Global Enqueue Service Monitor全局查询服务监视进程
LCK0 - Instance Enqueue Process 实例队列进程
Oracle RAC一般也可构建于大型SMP主机，IBM的AIX系列服务器往往是其中高端平台，Intel Linux往往作为其低端平台。当AIXUNIX用来运行Oracle RAC作为大型数据库系统平台时，其集群系统构建、实施、运维、高可用设置，有其平台特点。可以参照《Oracle大型数据库系统在AIX/UNIX上的实战详解》，该书以AIX UNIX平台为主线，以其他UNIX系统为参照，描述了数据库系统Oracle 10g、Oracle 11g的RAC的构架方法和过程。在Linux平台，则《大话OracleRAC集群、高可用性、备份与恢复》有着很好的论述。
### 最佳实践
Oracle RAC节点负载均衡最佳实践
Oracle提出的负载均衡基于最小负载的实现方法，增加了额外的cache fusion。在实际环境中，相似业务的最终用户都将请求发送到同一RAC节点上。如果RAC系统有不同类型的最终用户，我们会希望将负载均衡到不同的数据区域去。举例来说，客户处理可能在节点1上，订单处理在节点2上，而产品处理则在节点3上。将RAC最终用户通过数据需求来分组可以保证cache fusion负载降到最小。
Oracle RAC磁盘存储管理最佳实践
为了实施RAC系统，应该使用共享存储设备因为很多服务器都必须同时存取磁盘。一个单实例数据库可以使用Direct Attached Storage (DAS)这是一种连接到单一服务器上的一组廉价磁盘，而RAC则必须使用Storage Area Network (SAN)，这是更昂贵更复杂的通常使用光纤通道连接到多个服务器的磁盘阵列。这需要一组独立的硬件，从主机总线适配器连接到SAN上。因此DBA具有数据存储层面的完整知识就显得很重要。
## 系统要求&网络架构
需要至少两个节点，一个ISCSI服务器（可由NAS设备提供）
每个节点要求20G存储空间，4G内存，两块网卡（分别配置公共ip和私有ip）
ISCSI设备需要至少2块硬盘，一块作为系统盘 
5G即可），一块10G作为共享盘，这里采用三块。
       <h1>安装版本为11g Release 2 (11.2.0.4)，低版本可能遇到系统软件包版本过高引起的依赖无法验证的情况！！！</h1>
![拓扑图](http://192.168.30.5/pic/拓扑图.png  "拓扑图")
## 系统设置

### 修改hostname
```bash
vim  /etc/sysconfig/network
```
节点1
```
NETWORKING=yes
HOSTNAME=rac1
NOZEROCONF=yes
```
节点2

```
NETWORKING=yes
HOSTNAME=rac2
NOZEROCONF=yes
```
*** 临时修改IP直接执行 ```hostname  newhostname```***

### 固定IP（根据实际需求修改IP地址，子网掩码，网关，DNS即可，ONBOOT设置为YES）
```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
节点1
网卡1
```
DEVICE=eth0
TYPE=Ethernet
UUID=a976473d-65a7-428a-a3af-9524fea1415e
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=none
IPADDR=192.168.30.11
PREFIX=16
GATEWAY=192.168.1.10
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME=eth0
HWADDR=00:0C:29:4A:59:BE
LAST_CONNECT=1515762850        
```
网卡2
```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth1
```
```
DEVICE=eth1
TYPE=Ethernet
UUID=e75d39d6-1cb4-4d92-92f3-3005e5a9b003
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=none
IPADDR=10.0.0.11
PREFIX=24
GATEWAY=10.0.0.1
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME=eth1
HWADDR=00:0C:29:4A:59:C8
LAST_CONNECT=1515762850                     
```


节点2
网卡1
```
HWADDR=00:0C:29:8A:DB:16
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.30.12
PREFIX=16
GATEWAY=192.168.1.10
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME=eth0
UUID=29ac348e-b53a-45dc-98be-2bfa0c7d2c3c
ONBOOT=yes
LAST_CONNECT=1515494563
```
网卡2
```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth1
```
```
TYPE=Ethernet
BOOTPROTO=none
IPADDR=10.0.0.12
PREFIX=24
GATEWAY=10.0.0.1
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME=eth1
UUID=bb9b1da8-e41c-4242-8def-1cc2af9417ac
ONBOOT=yes
HWADDR=00:0C:29:8A:DB:20
LAST_CONNECT=1515490460

```
### 修改hosts文件
```bash
vim  /etc/hosts
```
节点1
```
127.0.0.1      localhost 
#Rac1  
192.168.30.11 rac1
192.168.30.13 rac1-vip  
10.0.0.11  rac1-priv  
  
#Rac2  
192.168.30.12 rac2
192.168.30.14 rac2-vip  
10.0.0.12  rac2-priv  
  
#scan-ip  
192.168.30.20 rac-cluster

```
节点2
```
127.0.0.1      localhost 
#Rac1  
192.168.30.11  rac1
192.168.30.13 rac1-vip  
10.0.0.11  rac1-priv  
  
#Rac2  
192.168.30.12 rac2
192.168.30.14 rac2-vip  
10.0.0.12  rac2-priv  
  
#scan-ip  
192.168.30.20 rac-cluster
```
### 关闭NTP服务
节点1
```bash
service ntpd status
chkconfig ntpd off
chkconfig --list | grep ntpd
```
节点2
```bash
service ntpd status
chkconfig ntpd off
chkconfig --list | grep ntpd
```
### 关闭SELINUX
```bash
vi /etc/selinux/config   
```
节点一
```
SELINUX=disabled 
```
节点2
```  
SELINUX=disabled 
```

## 修改相关参数(如果检测阶段无法通过则按照提示修改)
### 修改内核
```bash
vi etc/sysctl.conf
```
注释掉原文相关参数
```
# Controls the maximum shared segment size, in bytes
#kernel.shmmax = 68719476736

# Controls the maximum number of shared memory segments, in pages
#kernel.shmall = 4294967296

```
节点1
```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax =2000721920
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
```
节点2
```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 2001246208
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
```
### 修改 /etc/security/limits.conf文件
```bash
vi /etc/security/limits.conf
```
节点1添加
```
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
grid soft nproc 2047
grid hard nproc 16384
grid soft nofile 1024
grid hard nofile 65536
```
节点2添加
```
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
grid soft nproc 2047
grid hard nproc 16384
grid soft nofile 1024
grid hard nofile 65536
```
### 修改/etc/pam.d/login文件
```bash
vi /etc/pam.d/login
```
节点1添加
```
session required pam_limits.so
```
节点2添加
```
session required pam_limits.so
```
### 修改/etc/profile配置文件
```bash
vi /etc/profile
```
节点1添加
```
if [ $USER = "oracle" ] || [ $USER = "grid" ]; then

        if [ $SHELL = "/bin/ksh" ]; then

                ulimit -p 16384

                ulimit -n 65536

        else

                ulimit -u 16384 -n 65536

        fi

fi
```
节点2添加
```
if [ $USER = "oracle" ] || [ $USER = "grid" ]; then

        if [ $SHELL = "/bin/ksh" ]; then

                ulimit -p 16384

                ulimit -n 65536

        else

                ulimit -u 16384 -n 65536

        fi

fi
```
## 配置用户及用户组
### 新建用户&设置密码
节点1
```bash
groupadd -g 1000 oinstall 
groupadd -g 1001 dba
groupadd -g 1002 asmadmin
groupadd -g 1003 asmdba
groupadd -g 1004 asmoper
useradd -g oinstall -G asmadmin,asmdba,asmoper,dba -m grid
useradd -g oinstall -G dba,asmdba -m oracle
passwd grid
输入密码××××××
passwd oracle
输入密码××××××
```
节点2
```bash
groupadd -g 1000 oinstall 
groupadd -g 1001 dba
groupadd -g 1002 asmadmin
groupadd -g 1003 asmdba
groupadd -g 1004 asmoper
useradd -g oinstall -G asmadmin,asmdba,asmoper,dba -m grid
useradd -g oinstall -G dba,asmdba -m oracle
passwd grid
输入密码××××××
passwd oracle
输入密码××××××
```
### 设置环境变量
#### 设置grid用户变量
```bash
su grid
vim ~/.bash_profile
```
节点1
```
umask 022
TMP=/tmp
TMPDIR=/tmp
PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
ORACLE_BASE=/u02/app/grid
ORACLE_HOME=$ORACLE_BASE/11.2.0
ORACLE_SID=+ASM1
PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_BASE ORACLE_HOME ORACLE_SID PATH TMP TMPDIR
```
节点2
```
umask 022
TMP=/tmp
TMPDIR=/tmp
PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
ORACLE_BASE=/u02/app/grid
ORACLE_HOME=$ORACLE_BASE/11.2.0
ORACLE_SID=+ASM2
PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_BASE ORACLE_HOME ORACLE_SID PATH TMP TMPDIR
```
#### 配置oracle用户的环境变量
```bash
su oracle
vim ~/.bash_profile
```
节点1
```
umask 022
TMP=/tmp
TMPDIR=/tmp
PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
LD_LIBRARY_PATH=/usr/lib:/usr/X11R6/lib
ORACLE_BASE=/u01/app/oracle
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
ORACLE_SID=ora1
LD_LIBRARY_PATH=$ORACLE_HOME/jdk/jre/lib/i386:$ORACLE_HOME/jdk/jre/lib/i386/server:$ORACLE_HOME/rdbms/lib:$ORACLE_HOME/lib:/usr/lib:$LD_LIBRARY_PATH
PATH=$ORACLE_HOME/bin:$PATH
NLS_LANG=American_America.ZHS16GBK
export ORACLE_BASE ORACLE_HOME ORACLE_SID LD_LIBRARY_PATH PATH NLS_LANG TMP TMPDIR
```
节点2
```
umask 022
TMP=/tmp
TMPDIR=/tmp
PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
LD_LIBRARY_PATH=/usr/lib:/usr/X11R6/lib
ORACLE_BASE=/u01/app/oracle
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
ORACLE_SID=ora2
LD_LIBRARY_PATH=$ORACLE_HOME/jdk/jre/lib/i386:$ORACLE_HOME/jdk/jre/lib/i386/server:$ORACLE_HOME/rdbms/lib:$ORACLE_HOME/lib:/usr/lib:$LD_LIBRARY_PATH
PATH=$ORACLE_HOME/bin:$PATH
NLS_LANG=American_America.ZHS16GBK
export ORACLE_BASE ORACLE_HOME ORACLE_SID LD_LIBRARY_PATH PATH NLS_LANG TMP TMPDIR
```

### 创建目录
节点1
```bash
mkdir -p /u01/app/oracle
chown -R oracle:oinstall /u01
chmod -R 775 /u01
mkdir -p /u02/app/grid
chown -R grid:oinstall /u02
chmod -R 775 /u02
```
节点2
```bash
mkdir -p /u01/app/oracle
chown -R oracle:oinstall /u01
chmod -R 775 /u01
mkdir -p /u02/app/grid
chown -R grid:oinstall /u02
chmod -R 775 /u02
```

## 设置共享存储

### 搭建ISCSI服务器
将iscsitarget-1.4.20.2.tar.gz（[点击下载](http://192.168.30.5/grid/iscsitarget-1.4.20.2.tar.gz) ）上传到节点1
解压文件
```
tar -zxvf  iscsitarget-1.4.20.2.tar.gz
```
切换到文件目录
```
cd iscsitarget-1.4.20.2
  ```
编译安装
```
make && make install
```
### 给虚拟机添加三块硬盘
虚拟机会自动识别新添加的硬盘，例如
```
/dev/sdb
/dev/sdc
/dev/sdd
```
给硬盘分区，以sdb为例
```
fdisk /dev/sdb
```
- 输入n新建分区
- 输入e建立扩展分区
- 输入1设置卷标数
-  连续回车
- 输入n建立分区
- 输入l建立逻辑分区
- 连续回车
- 输入w保存

### 编辑iscsi配置
```bash
vi /etc/iet/ietd.conf
```
在文件末尾追加
```
Lun 0 Path=/dev/sdb5,Type=fileio,IOMode=wb 
Alias Lun 0
Lun 1 Path=/dev/sdc5,Type=fileio,IOMode=wb
Alias Lun 1
Lun 2 Path=/dev/sdd5,Type=fileio,IOMode=wb
Alias Lun 2
```
### 启动ISCSI服务
```bash
/etc/init.d/iscsi-target restart
```
设置防火墙地址根据需求填写
```
iptables -A INPUT -p tcp -s192.168.30.0/16 --dport 3260 -j ACCEPT

```
### 客户端配置
安装相关软件包
节点1
```bash
yum install iscsi-initiator-utils
```
发现iscsi服务（IP地址根据实际地址填写）
```
iscsiadm -m discovery -t sendtargets -p 192.168.30.10:3260
```
登录iscsi服务
```
iscsiadm -m node -T iqn名称 -p 服务器地址 -l
```
节点2
```bash
yum install iscsi-initiator-utils
```
发现iscsi服务（IP地址根据实际地址填写）
```
iscsiadm -m discovery -t sendtargets -p 192.168.30.10:3260
```
登录iscsi服务
```
iscsiadm -m node -T iqn名称 -p 服务器地址 -l
```
*常用iscsi命令*
```
1.发现iscsi存储: iscsiadm -m discovery -t st -p ISCSI_IP
2.查看iscsi发现记录 iscsiadm -m node
3.删除iscsi发现记录 iscsiadm -m node -o delete -T LUN_NAME -p ISCSI_IP
4.登录iscsi存储 iscsiadm -m node -T LUN_NAME -p ISCSI_IP -l
5.登出iscsi存储 iscsiadm -m node -T LUN_NAME -p ISCSI_IP -u
6 显示会话情况  iscsiadm -m session
```
## 配置裸设备
### 在节点一上配置裸设备文件
```bash
vi /etc/udev/rules.d/60-raw.rules 
```
```bash
ACTION=="add", KERNEL=="sdb1", RUN+="/bin/raw /dev/raw/raw1 %N"
ACTION=="add", KERNEL=="sdc1", RUN+="/bin/raw /dev/raw/raw2 %N"
ACTION=="add", KERNEL=="sdd1", RUN+="/bin/raw /dev/raw/raw3 %N"
ACTION=="add", ENV{MAJOR}=="8", ENV{MINOR}=="16", RUN+="/bin/raw /dev/raw/raw1 %M %m"
ACTION=="add", ENV{MAJOR}=="8", ENV{MINOR}=="32", RUN+="/bin/raw /dev/raw/raw2 %M %m"
ACTION=="add", ENV{MAJOR}=="8", ENV{MINOR}=="48", RUN+="/bin/raw /dev/raw/raw3 %M %m"
ACTION=="add", KERNEL=="raw1", OWNER="grid", GROUP="asmadmin", MODE="660"
ACTION=="add", KERNEL=="raw2", OWNER="grid", GROUP="asmadmin", MODE="660"
ACTION=="add", KERNEL=="raw3", OWNER="grid", GROUP="asmadmin", MODE="660"
```
其中 ENV{MAJOR}=="8", ENV{MINOR}=="16"中的数值可以通过lsblk命令查询

### 启动raw设备
节点1
```
start_udev 
```
节点2
```
start_udev 
```
### 查看配置情况
节点1
```
raw -qa
```
节点2
```
raw -qa
```

## 设置ssh信任关系
### 设置rsa和dsa加密
节点1
```
su grid
cd ~/
```

```
 ssh-keygen -t rsa
```
 一直回车，直到出现图形
 
```
 ssh-keygen -t dsa
```
  一直回车，直到出现图形
  
```
cd .ssh/
cat   id_rsa.pub  >>  authorized_keys
cat   id_dsa.pub  >>  authorized_keys
```
将 authorized_keys传输到节点2
```
scp  authorized_keys grid@rac2:/home/grid/.ssh/
```
输入密码

```
su oracle
cd ~/
```

```
 ssh-keygen -t rsa
```
 一直回车，直到出现图形
 
```
 ssh-keygen -t dsa
```
  一直回车，直到出现图形
  
```
cd .ssh/
cat   id_rsa.pub  >>  authorized_keys
cat   id_dsa.pub  >>  authorized_keys
```
将 authorized_keys传输到节点2
```
scp  authorized_keys grid@rac2:/home/oracle/.ssh/
```
输入密码，传输完成

节点2
```
su grid
cd ~/
```

```
 ssh-keygen -t rsa
```
 一直回车，直到出现图形
 
```
 ssh-keygen -t dsa
```
  一直回车，直到出现图形
  
```
cd .ssh/
cat   id_rsa.pub  >>  authorized_keys1
cat   id_dsa.pub  >>  authorized_keys1
```
将 authorized_keys1传输到节点1
```
scp  authorized_keys grid@rac1:/home/grid/.ssh/
```
输入密码

```
su oracle
cd ~/
```

```
 ssh-keygen -t rsa
```
 一直回车，直到出现图形
 
```
 ssh-keygen -t dsa
```
  一直回车，直到出现图形
  
```
cd .ssh/
cat   id_rsa.pub  >>  authorized_keys1
cat   id_dsa.pub  >>  authorized_keys1
```
将 authorized_keys1传输到节点1
```
scp  authorized_keys grid@rac1:/home/oracle/.ssh/
```
输入密码，传输完成
ssh到节点1

```
ssh rac2
su grid
mv /home/grid/.ssh/authorized_keys1 /home/grid/.ssh/authorized_keys
su oracle
mv /home/grid/.ssh/authorized_keys1 /home/oracle/.ssh/authorized_keys
```
测试连接
在节点１上执行下面的命令，正确返回日期便表示成功
```
ssh rac1 date
ssh rac1-priv date
ssh rac2 date
ssh rac2-priv date
```
在节点２上执行下面的命令，正确返回日期便表示成功
```
ssh rac1 date
ssh rac1-priv date
ssh rac2 date
ssh rac2-priv date
```
## 安装依赖包
```
binutils (x86_64) 
glibc (x86_64)  
kshx86_64)  
libaio (x86_64)  
libstdc++(x86_64)  
libstdc++ (x86_64)  
libgcc (x86_64)  
make-3.81-128.20 (x86_64)  
linux-kernel-headers
gcc
libstdc
libgomp
gcc43-c++ 
libaio-devel
sysstat
glibc-devel
```
 ***大部分可以通过yum安装，如果安装失败可以采用rpm安装***
## 安装 Grid 11g  
### 将安装包（[点击下载](http://192.168.30.5/grid/p13390677_112040_Linux-x86-64_3of7.zip) ）下载到本地然后上传到服务器或执行如下命令下载到服务器
```
wget http://192.168.30.5/grid/p13390677_112040_Linux-x86-64_3of7.zip
```
将文件解压到grid的home目录
```
unzip p13390677_112040_Linux-x86-64_1of7.zip 
mv grid/  /home/grid
```
### 安装Grid 
切换到grid用户，切换到grid安装目录
运行runInstaller进行安装
```
su grid
cd ~/grid/
./runInstaller
```
- 首先选择第一项
![pic1](http://192.168.30.5/pic/图片1.png  "pic1")
- 选择高级选项
 ![pic2](http://192.168.30.5/pic/图片2.png  "pic2")
 - 将简体中文添加到右侧
 ![pic3](http://192.168.30.5/pic/图片3.png  "pic3")
 - 去掉Configure GNS前面的对勾，SCAN—Name要和hosts文件写入的ip地址一样
 ![pic4](http://192.168.30.5/pic/图片4.png  "pic4")
-  点击Add，配置节点2，名称要和hosts文件里一致，点击SSH connectivity，输入节点密码，点击Setup，提示无错误即可点击Next
 ![pic5](http://192.168.30.5/pic/图片5.png  "pic5")
 - 根据事实选择公共IP和私有IP对应的网卡
 ![pic6](http://192.168.30.5/pic/图片7.png  "pic6")
 - 选择ASM选项
 ![pic 7](http://192.168.30.5/pic/图片8.png  "pic7")
 - 选择第一块磁盘，点击Next
 ![pic8](http://192.168.30.5/pic/图片6.png  "pic8")
 - 选择Use same passswords for these accounts，设置密码，点击Next
 ![pic9](http://192.168.30.5/pic/图片9.png  "pic9")
- 关闭IPM服务
![pic10](http://192.168.30.5/pic/图片14.png  "pic10")
- 如图所示选择，点击Next
![pic11](http://192.168.30.5/pic/图片15.png  "pic11")
- 根据事实填写，点击Next
![pic12](http://192.168.30.5/pic/图片16.png  "pic12")
- 根据事实填写，点击Next
![pic13](http://192.168.30.5/pic/图片10.png  "pic13")
- 运行安装自检，根据提示安装所需包
![pic14](http://192.168.30.5/pic/图片11.png  "pic14")
- 点击Finish
![pic15](http://192.168.30.5/pic/图片12.png  "pic15")
- 运行完成后会提示运行脚本
 ![pic16](http://192.168.30.5/pic/图片13.png  "pic16")

- 在节点1上运行脚本
 
```
	cd /u01/grid/oraInventory/ #根据提示切换目录
	./ orainstRoot.sh
	cd /u01/app/grid/11.2.0/   #根据提示填写
	./root.sh
```
 
- 运行完成后在节点2执行
```
	cd /u01/grid/oraInventory/  #根据提示切换目录
	./ orainstRoot.sh
	cd /u01/app/grid/11.2.0/   #根据提示填写
	./root.sh
```
-  运行之后点击ok, 继续安装
- 安装之后会有两个错误，在/etc/hosts中配置了SCAN的地址，尝试ping这个地址信息，如果可以成功，则这个错误可以忽略。
![pic17](http://192.168.30.5/pic/图片18.png  "pic17")
- 安装完成

## 安装oracle 11g
下载安装包1（[下载](wget http://192.168.30.5/grid/p13390677_112040_Linux-x86-64_1of7.zip) ）和安装包2（[下载](wget http://192.168.30.5/grid/p13390677_112040_Linux-x86-64_2of7.zip) ）或者在节点上执行以下命令
```
wget wget http://192.168.30.5/grid/p13390677_112040_Linux-x86-64_1of7.zip
wget wget http://192.168.30.5/grid/p13390677_112040_Linux-x86-64_2of7.zip
```
解压文件
```
unzip  wget http://192.168.30.5/grid/p13390677_112040_Linux-x86-64_1of7.zip
unzip  wget http://192.168.30.5/grid/p13390677_112040_Linux-x86-64_2of7.zip
```
将文件放入oracle的home目录下
```
mv database /home/oracle
```
切换到ORACLE用户进行安装操作
```
su oracle
cd  ~/database/
./runInstaller
```
- 去掉复选框，点击Next，有错误提示直接或略即可
![db1](http://192.168.30.5/pic/db1.png  "db1")
- 选择“install database software only”并点击Next
![db2](http://192.168.30.5/pic/db2.png  "db2")
- 选择“Real Application Clusters datebase installation”，并且勾选两个节点，完成后选择“SSH connectivity”，点击Setup，完成后，点击Next
![db3](http://192.168.30.5/pic/db3.png  "db3")
- 将简体中文添加到右侧，点击Next
![db4](http://192.168.30.5/pic/db4.png  "db4")
- 选择商业版，点击Next
![db5](http://192.168.30.5/pic/db5.png  "db5")
- 选择保存位置，点击Next
![db6](http://192.168.30.5/pic/db6.png  "db6")
- 如图所示选择，点击Next
![db7](http://192.168.30.5/pic/db7.png  "db7")
- 运行安装检查，根据提示安装依赖包以及解决警告，完成后点击Next
![db8](http://192.168.30.5/pic/db8.png  "db8")
点击Finish
![db9](http://192.168.30.5/pic/db9.png  "db9")
根据提示，运行脚本
![db10](http://192.168.30.5/pic/db10.png  "db10")

- 在节点1上运行脚本 
```
	cd /u01/app/oracle/product/11.2.0/db_1   #根据提示填写
	./root.sh
```
- 在节点2上运行脚本 
```
	cd  /u01/app/oracle/product/11.2.0/db_1  #根据提示填写
	./root.sh
```
- 安装完成后点击close
![db11](http://192.168.30.5/pic/db11.png  "db11")
***安装完成***
## 建立数据库
- 在节点1上用oracle用户执行dbca，打开配置工具
```
dbca
```
 - 选择第一项点击Next
![ndb1](http://192.168.30.5/pic/ndb1.png  "ndb1")
- 选择第一项，点击Next
![ndb2](http://192.168.30.5/pic/ndb2.png  "ndb2")
- 选择第一项，点击Next
![ndb3](http://192.168.30.5/pic/ndb3.png  "ndb3")
- oracle数据库名称按照环境变量里的SID填写（env | grep SID）
![ndb4](http://192.168.30.5/pic/ndb4.png  "ndb4")
- 取消“Configure Enterprise Manager”复选框，点击Next
![ndb5](http://192.168.30.5/pic/ndb5.png  "ndb5")
- 选择"Use the Same Administrative Password for All Accounts",设置密码并点击Next
![ndb6](http://192.168.30.5/pic/ndb6.png  "ndb6")
- 如图所示选择，点击Next
![ndb7](http://192.168.30.5/pic/ndb7.png  "ndb7")
- 按照如图所示填写（输入框保持默认即可），点击Next
![ndb8](http://192.168.30.5/pic/ndb8.png  "ndb8")
- 按照如图所示选择，点击Next
![ndb9](http://192.168.30.5/pic/ndb9.png  "ndb9")
- 按照如图所示选择，点击Next
![ndb10](http://192.168.30.5/pic/ndb10.png  "ndb10")
- 点击“File Location Viriables”查看无误点击Next
![ndb11](http://192.168.30.5/pic/ndb11.png  "ndb11")
- 如图所示选择,,点击Finish
![ndb12](http://192.168.30.5/pic/ndb12.png  "ndb12")
- 软件会自动执行，等待完成即可
![ndb13](http://192.168.30.5/pic/ndb13.png  "ndb13")

***运行完成后数据库建立完成***
## 启动数据库
执行如下命令
```
sqlplus / as sysdba
```
在SQL界面执行以下命令
```
startup
shutdown immediate
exit
```
## 验证集群、数据库
```
cd /u02/app/grid/11.2.0/bin
crsctl stat res -t
```
。。。。。。
更多内容参考[Rac相关命令](192.168.30.5/2018/01/16/%E5%AE%89%E8%A3%85grid/) 
# 常见问题解答
## hosts 文件以及网卡配置规则
- 网卡１（eth0）设置为公共ip（和本地计算机所在局域网在相同网段）
- 网卡１（eth0）设置为私有ip（和局域网任意一台机器都不处在同一网段）
- hosts文件在两台机器上应相同
- hosts文件中虚拟ip应和公共ip在同一网段
- **虚拟ＩＰ不要与网卡绑定**

## iscsi服务无法连接问题
- 首先检查ＩＰ和端口是否正确
- 检查防火请是否允许ISCSI端口通过,如果没通过则执行下面的命令

```
iptables -A INPUT -p tcp -s192.168.30.0/16 --dport 3260 -j ACCEPT
```
- 或者直接关闭防火墙
```
service iptables stop 
chkconfig iptables off
```
拓展知识：[iptables详解](http://localhost:4000/2018/01/19/%E9%98%B2%E7%81%AB%E5%A2%99%E8%AE%BE%E7%BD%AE/#more) 

## ssh连接失败
- authorized_keys不存在或者内容有误，重新执行ssh信任关系
- 权限问题，执行
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
 
## 执行root.sh脚本报错
- 查看/etc/hosts配置，是否配置localhost回环地址
```
127.0.0.1    localhost
```
- 如果节点2报错
则编辑如下文件
```
vi /etc/sysconfig/oracleasm
```
```
ORACLEASM_ENABLED=true 
ORACLEASM_UID=grid 
ORACLEASM_GID=asmdba 
ORACLEASM_SCANBOOT=true 
ORACLEASM_SCANORDER="dm" 
ORACLEASM_SCANEXCLUDE="sd" 
ORACLEASM_USE_LOGICAL_BLOCK_SIZE=false
```
然后执行
```
/etc/init.d/oracleasm restart
```

第二节点先执行清除配置，再执行root.sh即可:
```
/u01/app/11.2.0/grid/crs/install/roothas.pl -delete -force -verbose    
/u01/app/11.2.0/grid/root.sh
```
## 安装过程中提示swap交换分区不足
可以新建文件作为交换文件使用,执行下列命令(count后数字为ＧＢ数×１０２４×１０００)

```
dd if=/dev/zero of=/home/swap bs=1024 count=2048000
mkswap /home/swap
/sbin/swapon /home/swap
```
