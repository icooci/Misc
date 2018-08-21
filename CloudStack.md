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
```
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

> ir -p /export/secondary
