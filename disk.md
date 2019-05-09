# glusterfs
### 配置主机名
```
[root@ssy-db3 ~]# cat /etc/hosts
10.66.178.32    ssy-db1
10.66.179.171   ssy-db3
10.66.179.190   ssy-service
```
### 安装包
```
[root@ssy-db3 ~]# yum install centos-release-gluster
[root@ssy-db3 ~]# yum install -y glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma
```
### 启动服务
```
[root@ssy-db3 ~]# systemctl start glusterd
[root@ssy-db3 ~]# systemctl enable glusterd  #开机启动
```
### 在ssy-db1上配置节点加入集群
```
[root@ssy-db1 packages]# gluster peer probe ssy-db1
peer probe: success. Probe on localhost not needed
[root@ssy-db1 packages]# gluster peer probe ssy-db3
peer probe: success.
```
### 创建数据目录
```
[root@ssy-db1 ~]# mkdir /home/easemob/tmp
[root@ssy-db3 ~]# mkdir /home/easemob/tmp
```
### 创建glusterfs磁盘
```
[root@ssy-db1 ~]# gluster volume create easemob replica 2 ssy-db1:/home/easemob/tmp ssy-db3:/home/easemob/tmp force
volume create: easemob: success: please start the volume to access data
```

### 启动glusterfs volume
```
[root@ssy-db1 ~]# gluster volume start easemob
volume start: easemob: success
[root@ssy-db1 ~]# gluster volume info
 
Volume Name: easemob
Type: Replicate
Volume ID: a0e9f623-fa3c-4230-aed4-700d9fe1f967
Status: Started  #由created变为started
Snapshot Count: 0
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: ssy-db1:/home/easemob/tmp
Brick2: ssy-db3:/home/easemob/tmp
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off
```
### 挂载
```
[root@ssy-db1 ~]# mount -t glusterfs  ssy-db1:easemob /root/glus/
[root@ssy-db1 ~]# echo 'test' > /root/glus/test   #测试
[root@ssy-db1 ~]# ll /home/easemob/tmp/
total 12
-rw-r--r-- 2 root root 5 May  8 17:46 test
[root@ssy-db3 ~]# ll /home/easemob/tmp/ #ssy-db3上也有
total 8
-rw-r--r-- 2 root root 5 May  8 17:46 test
```
### 卸载
```
[root@ssy-service1 easemob]# df -h
Filesystem       Size  Used Avail Use% Mounted on
/dev/vda1         40G   18G   20G  47% /
ssy-db1:easemob   40G   26G   13G  67% /home/easemob/test
[root@ssy-service1 easemob]# umount ssy-db1:easemob
[root@ssy-service1 easemob]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G   18G   20G  47% /
[root@ssy-service1 easemob]# 
```
### mount失败问题处理
```
[root@ssy-service1 ~]# mount -t glusterfs  ssy-db1:easemob /home/easemob/test/
Mount failed. Please check the log file for more details.
需在gluster服务端添加访问策略（添加all后无效，原因未知）
[root@ssy-db1 ~]# gluster volume set easemob auth.allow 10.66.179.190
volume set: success
```

# df -h卡住
### 使用strace名令追踪卡住的地方
```
[root@ssy-service1 ~]# strace df -h
execve("/usr/bin/df", ["df", "-h"], [/* 23 vars */]) = 0
...
...
stat("/", {st_mode=S_IFDIR|0555, st_size=4096, ...}) = 0
stat("/proc/sys/fs/binfmt_misc", 
```
### 然后重启卡住的服务
```
[root@ssy-service1 ~]# systemctl restart proc-sys-fs-binfmt_misc.automount
[root@ssy-service1 ~]# 
[root@ssy-service1 ~]# 
[root@ssy-service1 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G   32G  5.6G  86% /
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           3.9G  408M  3.5G  11% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
tmpfs           783M     0  783M   0% /run/user/0
tmpfs           783M     0  783M   0% /run/user/1000
```
