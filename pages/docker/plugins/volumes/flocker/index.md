---
layout: default
title: flocker插件安装
---

#一、安装flocker client
##1、安装编译、python、虚拟环境
```
yum install -y gcc libffi-devel openssl-devel python python-devel python-virtualenv
```
##2、安装flocker-cli
```
virtualenv --python=/usr/bin/python2.7 flocker-client
source flocker-client/bin/activate
pip install --upgrade pip
pip install https://clusterhq-archive.s3.amazonaws.com/python/Flocker-1.15.0-py2-none-any.whl
```
当需要使用flocker-cli命令时，首先要进入flocker client环境：
```
source flocker-client/bin/activate
flocker-ca --version
```
#二、安装flocker node services
##安装clusterhq-flocker-node包
```
yum list installed clusterhq-release || yum install -y https://clusterhq-archive.s3.amazonaws.com/centos/clusterhq-release$(rpm -E %dist).noarch.rpm
yum install -y clusterhq-flocker-node
```
##安装clusterhq-flocker-docker-plugin包
```
yum install -y clusterhq-flocker-docker-plugin
```
#三、配置集群证书
##生成集群证书
```
mkdir /etc/flocker
flocker-ca initialize <mycluster>
```
##生成控制节点证书
```
flocker-ca create-control-certificate <hostname>
```
复制生成的证书到控制节点flocker目录(/etc/flocker)
```
control-flkn1.crt
control-flkn1.key
cluster.crt (as created by the flocker-ca initialize step)
```
重命名控制节点证书
```
mv control-flkn1.crt control-service.crt
mv control-flkn1.key control-service.key
chmod 0700 /etc/flocker
chmod 0600 /etc/flocker/control-service.key
```
##生成节点认证证书
```
flocker-ca create-node-certificate
8eab4b8d-c0a2-4ce2-80aa-0709277a9a7a.crt
8eab4b8d-c0a2-4ce2-80aa-0709277a9a7a.key
```
复制生成的两个文件到flocker集群的代理节点的flocker目录(/etc/flocker)，并重命名为node.crt、node.key
```
scp <yourUUID>.crt root@<hostname>:/etc/flocker/
scp <yourUUID>.key root@<hostname>:/etc/flocker/
scp cluster.crt root@<hostname>:/etc/flocker/
ssh root@<hostname>
cd /etc/flocker/
mv <yourUUID>.crt node.crt
mv <yourUUID>.key node.key
chmod 0700 /etc/flocker
chmod 0600 /etc/flocker/node.key
```
#四、生成API客户端证书
```
flocker-ca create-api-certificate <client_name>
```
将生成的<client_name>.crt、<client_name>.key，以及之前生成的cluster.crt，提供给API客户端使用。客户端使用这三个证书调用flocker的rest api

#五、为docker的flocker插件生成一个API证书
```
flocker-ca create-api-certificate plugin
```
将生成的plugin.crt、plugin.key复制到每个flocker节点的目录中(/etc/flocker)
