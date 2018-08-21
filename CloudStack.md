## Management部署

配置管理节点Hosts

> vi /etc/hosts
```
127.0.0.1       localhost
127.0.0.1       cloudstack
127.0.0.1       icooci.com
127.0.0.1       cloudstack.icooci.com
```

安装NTP服务器
> apt install openntpd


关闭apparmor
> systemctl disable apparmor


安装数据库
> apt install mysql-server


配置cnf
> vi /etc/mysql/conf.d/cloudstack.cnf
```bash
[mysqld]
server-id=master-01
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log-bin=mysql-bin
binlog-format = 'ROW'
```


重启服务
> service mysql restart


安装NFS
> apt install nfs-kernel-server


创建NFS目录
> mkdir -p /export/primary
> mkdir -p /export/secondary


配置NFS
> vi /etc/exports

```bash
/export  *(rw,async,no_root_squash,no_subtree_check)
```

> mkdir /etc/sysconfig

> vi /etc/sysconfig/nfs
```
LOCKD_TCPPORT=32803
LOCKD_UDPPORT=32769
MOUNTD_PORT=892
RQUOTAD_PORT=875
STATD_PORT=662
STATD_OUTGOING_PORT=2020
```

> vi /etc/idmapd.conf
```
Domain = icooci.com
```

测试NFS挂载
```bash
touch  /export/primary/t1
touch  /export/secondary/t2

mount -t nfs 10.7.1.98:/export/primary /mnt/
ls -al /mnt
umount /mnt
```

配置CloudStack源
> vi /etc/apt/sources.list.d/cloudstack.list
```
deb http://cloudstack.apt-get.eu/ubuntu xenial 4.11
```

添加APT-KEY
> wget -O - http://cloudstack.apt-get.eu/release.asc|apt-key add -

更新APT源
> apt update

安装Cloudstack Management
> apt install cloudstack-management


配置sudoers
> chmod 640 /etc/sudoers

> vi /etc/sudoers
```
Defaults:cloud !requiretty
```


初始化数据库
> cloudstack-setup-databases cloud:asd@localhost --deploy-as=root:asd


启动CSM
> cloudstack-setup-management


部署KVM系统VM模板
```
/usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
-m /export/secondary \
-u http://cloudstack.apt-get.eu/systemvm/4.11/systemvmtemplate-4.11.1-kvm.qcow2.bz2 \
-h kvm -F
```

部署Xen系统VM模板
```
/usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
-m /export/secondary \
-u http://cloudstack.apt-get.eu/systemvm/4.11/systemvmtemplate-4.11.1-xen.vhd.bz2 \
-h xenserver -F
```

部署vhd工具
> cd /usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver
> wget http://download.cloud.com.s3.amazonaws.com/tools/vhd-util
> chmod 755 vhd-util


验证操作
---

通过8080端口访问
http://10.7.1.98:8080

> Username: admin
> Password: password

**Secondary Storage流量放行**

GUI配置
> {secstorage.allowed.internal.sites} 10.7.1.0/24


数据库直接修改
> mysql -u cloud -pasd
```
UPDATE cloud.configuration SET value='10.7.1.0/24' WHERE name='secstorage.allowed.internal.sites';
```

