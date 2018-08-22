## KVM计算节点

配置Hosts

> vi /etc/hosts
```
127.0.0.1       localhost
10.7.1.98       cloudstack
10.7.1.98       icooci.com
10.7.1.98       cloudstack.icooci.com
```

关闭Apparomr
> systemctl disable apparmor

配置CloudStack源
> vi /etc/apt/sources.list.d/cloudstack.list
```
deb http://cloudstack.apt-get.eu/ubuntu xenial 4.11
```

添加APT-KEY  
`wget -O - http://cloudstack.apt-get.eu/release.asc|apt-key add -`

更新APT源
> apt update

安装NTP
> apt install openntpd

安装NFS
> apt install nfs-client


安装Cloudstack Agent
> apt install cloudstack-agent

配置libvirt
> vi /etc/libvirt/libvirtd.conf
```bash
listen_tls = 0
listen_tcp = 1
tcp_port = "16509"
mdns_adv = 0
unix_sock_group = "libvirtd"
unix_sock_ro_perms = "0777"
unix_sock_rw_perms = "0770"
auth_unix_ro = "none"
auth_unix_rw = "none"
auth_tcp = "none"
```

> vi /etc/init/libvirt-bin.conf
```
env libvirtd_opts="-d -l"
```

配置网络

`PS: 必须先装cloudstack-agent，否则无bridge-utils`

> vi /etc/network/interfaces
```bash
auto ens3
iface ens3 inet manual

auto cloudbr0
iface cloudbr0 inet static
    bridge_ports ens3
    bridge_fd 5
    bridge_stp off
    bridge_maxwait 1
    address 10.7.1.98
    netmask 255.255.255.0
    gateway 10.7.1.1
    dns-nameservers 8.8.8.8
```

验证操作
---

部署 XenServer 6.5.0
> XenServer-6.5.0-xenserver.org-install-cd.iso

GUI操作
> GUI操作 -> 创建pool

配置bridge
> xe-switch-network-backend bridge
