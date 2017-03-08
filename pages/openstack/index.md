#Openstack安装
##环境准备
1、设置SELINUX=disabled
```
vi /etc/sysconfig/selinux 
SELINUX=disabled
```
2、设置hostname
```
hostnamectl set-hostname node1
systemctl restart network
```
3、安装工具包
```
yum install -y ntp wget net-tools
yum install -y yum-plugin-priorities
yum install -y http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm
yum upgrade
reboot
```
##安装openstack组件
```
yum install -y mysql mysql-server MySQL-python
yum install -y rabbitmq-server
yum install -y centos-release-openstack-liberty
yum makecache

yum install -y openstack-utils openstack-keystone python-keystoneclient
yum install -y openstack-nova openstack-glance
yum install -y memcached mod-wsgi openstack-dashboard
```
