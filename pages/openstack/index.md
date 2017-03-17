#Openstack安装
##环境准备

设置SELINUX=disabled
```
vi /etc/sysconfig/selinux 
SELINUX=disabled
```
设置hostname
```
hostnamectl set-hostname node1
systemctl restart network
```
##安装OpenStack包
启用OpenStack库
```
yum install centos-release-openstack-mitaka
yum upgrade
reboot
```
安装openstack客户端
```
yum install python-openstackclient
yum install openstack-selinux
```
##安装mysql
```
wget http://repo.mysql.com/mysql-community-release-el7-7.noarch.rpm
rpm -ivh mysql-community-release-el7-7.noarch.rpm
yum install -y mysql-server
systemctl enable mysql
systemctl start mysql
```
##安装消息队列组件rabbitmq
```
yum install rabbitmq-server
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
```
添加openstack用户
```
rabbitmqctl add_user openstack <password>
```
给openstack用户配置写和读权限
```
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
##安装memcached
```
yum install memcached python-memcached
systemctl enable memcached.service
systemctl start memcached.service
```
##安装认证组件keystone
先决条件，创建keystone数据库，并创建keystone用户，配置权限
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY '<password>';
```
生成一个随机数作为管理员令牌(后面用到)
```
openssl rand -hex 10
```
0913caf0fe6a2ec3f075
