---
title: Rac常用命令
date: 2018-01-22 09:46:57
tags: Rac
---
# Rac
### 检查集群状态：
```
crsctl check cluster
	CRS-4537: Cluster Ready Services is online
	CRS-4529: Cluster Synchronization Services is online
	CRS-4533: Event Manager is online
```
<!-- more --> 
### 所有 Oracle 实例 —（数据库状态）:
```
srvctl status database -d vcpdb
　　Instance racdb1 is running on node vcpdb1
　　Instance racdb2 is running on node vcpdb2
```
### 检查单个实例状态：
```
srvctl status instance -d racdb -i racdb1
　　Instance racdb1 is running on node rac01
```
### 节点应用程序状态：
```
srvctl status nodeapps
　　VIP rac01-vip is enabled
　　VIP rac01-vip is running on node: rac01
　　VIP rac02-vip is enabled
　　VIP rac02-vip is running on node: rac02
　　Network is enabled
　　Network is running on node: rac01
　　Network is running on node: rac02
　　GSD is disabled
　　GSD is not running on node: rac01
　　GSD is not running on node: rac02
　　ONS is enabled
　　ONS daemon is running on node: rac01
　　ONS daemon is running on node: rac02
　　eONS is enabled
　　eONS daemon is running on node: rac01
　　eONS daemon is running on node: rac02
```
### 列出所有的配置数据库：
```
srvctl config database
　　racdb
```
### 数据库配置：
```
srvctl config database -d racdb -a
　　Database unique name: racdb
　　Database name: racdb
　　Oracle home: /u01/app/oracle/product/11.2.0/dbhome_1
　　Oracle user: oracle
　　Spfile: +RACDB_DATA/racdb/spfileracdb.ora
　　Domain: xzxj.edu.cn
　　Start options: open
　　Stop options: immediate
　　Database role: PRIMARY
　　Management policy: AUTOMATIC
　　Server pools: racdb
　　Database instances: racdb1,racdb2
　　Disk Groups: RACDB_DATA,FRA
　　Services:
　　Database is enabled
　　Database is administrator managed
```
### ASM状态以及ASM配置：
```
srvctl status asm
　　ASM is running on rac01,rac02
srvctl config asm -a
　　ASM home: /u01/app/11.2.0/grid
　　ASM listener: LISTENER
　　ASM is enabled.
```
### TNS监听器状态以及配置：
```
srvctl status listener
　　Listener LISTENER is enabled
　　Listener LISTENER is running on node(s): rac01,rac02
srvctl config listener -a
　　Name: LISTENER
　　Network: 1, Owner: grid
　　Home: <CRS home>
　　/u01/app/11.2.0/grid on node(s) rac02,rac01
　　End points: TCP:1521
```
### SCAN状态以及配置：
```
srvctl status scan
　　SCAN VIP scan1 is enabled
　　SCAN VIP scan1 is running on node rac02
srvctl config scan
　　SCAN name: rac-scan.xzxj.edu.cn, Network: 1/192.168.1.0/255.255.255.0/eth0
　　SCAN VIP name: scan1, IP: /rac-scan.xzxj.edu.cn/192.168.1.55
```
### VIP各个节点的状态以及配置：
```
srvctl status vip -n rac01
　　VIP rac01-vip is enabled
　　VIP rac01-vip is running on node: rac01
srvctl status vip -n rac02
　　VIP rac02-vip is enabled
　　VIP rac02-vip is running on node: rac02
srvctl config vip -n rac01
　　VIP exists.:rac01
　　VIP exists.: /rac01-vip/192.168.1.53/255.255.255.0/eth0
srvctl config vip -n rac02
　　VIP exists.:rac02
　　VIP exists.: /rac02-vip/192.168.1.54/255.255.255.0/eth0
```
### 节点应用程序配置 —（VIP、GSD、ONS、监听器）
```
srvctl config nodeapps -a -g -s -l
　　-l option has been deprecated and will be ignored.
　　VIP exists.:rac01
　　VIP exists.: /rac01-vip/192.168.1.53/255.255.255.0/eth0
　　VIP exists.:rac02
　　VIP exists.: /rac02-vip/192.168.1.54/255.255.255.0/eth0
　　GSD exists.
　　ONS daemon exists. Local port 6100, remote port 6200
　　Name: LISTENER
　　Network: 1, Owner: grid
　　Home: <CRS home>
　　/u01/app/11.2.0/grid on node(s) rac02,rac01
　　End points: TCP:1521
```
### 验证所有集群节点间的时钟同步:
```
cluvfy comp clocksync -verbose
　　Verifying Clock Synchronization across the cluster nodes
　　Checking if Clusterware is installed on all nodes...
　　Check of Clusterware install passed
　　Checking if CTSS Resource is running on all nodes...
　　Check: CTSS Resource running on all nodes
　　Node Name Status
　　------------------------------------ ------------------------
　　rac02 passed
　　Result: CTSS resource check passed
　　Querying CTSS for time offset on all nodes...
　　Result: Query of CTSS for time offset passed
　　Check CTSS state started...
　　Check: CTSS state
　　Node Name State
　　------------------------------------ ------------------------
　　rac02 Active
　　CTSS is in Active state. Proceeding with check of clock time offsets on all nodes...
　　Reference Time Offset Limit: 1000.0 msecs
　　Check: Reference Time Offset
　　Node Name Time Offset Status
　　------------ ------------------------ ------------------------
　　rac02 0.0 passed
　　Time offset is within the specified limits on the following set of nodes:
　　"[rac02]"
　　Result: Check of clock time offsets passed
　　Oracle Cluster Time Synchronization Services check passed
　　Verification of Clock Synchronization across the cluster nodes was successful.
```
### 集群中所有正在运行的实例 — (SQL):
```
SELECT inst_id , instance_number inst_no , instance_name inst_name , parallel , status ,database_status db_status , active_state state , host_name host FROM gv instance ORDER BY inst_id;
```
### 启动和停止集群：
　　以下操作需用root用户执行。
- 在本地服务器上停止Oracle Clusterware 系统：
```
/u01/app/11.2.0/grid/bin/crsctl stop cluster
```
　***注：在运行“crsctl stop cluster”命令之后，如果 Oracle Clusterware 管理的资源中有任何一个还在运行，则整个命令失败。使用 -f 选项无条件地停止所有资源并***
### 停止 Oracle Clusterware 系统。
　　***另请注意，可通过指定 -all 选项在集群中所有服务器上停止 Oracle Clusterware系统。如下所示，在rac01和rac02上停止oracle clusterware系统：***
```
/u01/app/11.2.0/grid/bin/crsctl stop cluster –all
```
#### 在本地服务器上启动oralce clusterware系统：
```
u01/app/11.2.0/grid/bin/crsctl start cluster
```
***注：可通过指定 -all 选项在集群中所有服务器上启动 Oracle Clusterware 系统。***
```
/u01/app/11.2.0/grid/bin/crsctl start cluster –all
```
#### 还可以通过列出服务器（各服务器之间以空格分隔）在集群中一个或多个指定的
服务器上启动 Oracle Clusterware 系统：
```
/u01/app/11.2.0/grid/bin/crsctl start cluster -n rac01 rac02
```
#### 使用 SRVCTL 启动/停止所有实例:
```
srvctl stop database -d racdb
srvctl start database -d racdb
```