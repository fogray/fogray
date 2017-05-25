## 环境准备
ambari启用kerberos需要有一个独立的KDC server
```
$ vagrant up u1404
```
## 安装KDC
SSH登录U1404
```
# sed -i "s/us.archive.ubuntu.com/mirrors.163.com/" /etc/apt/sources.list
# sed -i "s/security.ubuntu.com/mirrors.163.com/" /etc/apt/sources.list
# apt-get update       (如果直接使用ubuntu官方源更新，会提示有hashsum错误)

# apt-get install krb5-kdc krb5-admin-server
# 
```
安装过程中出现提示窗口让输入Default Kerberos version 5 realm，保留默认值AMBARI.APACHE.ORG。然后出现两次让输入hostname，都输入的"u1404.ambari.apache.org"。最后提示说这个向导没有自动建立一个kerberos realm，如果想建立就执行命令"krb5_newrealm"。相关说明在/usr/share/doc/krb5-kdc/README.KDC中。
```
# krb5_newrealm
master key name 'K/M@AMBARI.APACHE.ORG'
Enter KDC database master key:    (输入两次密码，密码是vagrant)
```
启动KDC server和KDC admin server：
```
# service krb5-kdc restart                     （如果不执行krb5_newrealm就无法启动这个服务）
# service krb5-admin-server restart            （如果不执行krb5_newrealm就无法启动这个服务
```
## 创建Kerberos Admin
通过创建admin主体来建立KDC admin：
```
# kadmin.local -q "addprinc root/admin"
Enter password for principal "root/admin@AMBARI.APACHE.ORG":    (输入两次密码，密码是vagrant)
Principal "admin/admin@AMBARI.APACHE.ORG" created.
```
将刚创建的admin主体添加到KDC ACL中：
```
# echo "*/admin@AMBARI.APACHE.ORG *" >> /etc/krb5kdc/kadm5.acl
# service krb5-admin-server restart
```
查看主体清单的方法：在u1404节点（安装KDC的节点）上执行：
```
$ kadmin.local
kadmin.local: list_principals  
HTTP/u1402.ambari.apache.org@AMBARI.APACHE.ORG
HTTP/u1403.ambari.apache.org@AMBARI.APACHE.ORG
K/M@AMBARI.APACHE.ORG
admin/admin@AMBARI.APACHE.ORG
(略)
```
## 在ambari中启动kerberos安装向导
由于已经在u1404上部署了KDC，所有在向导中选择“Existing MIT KDC”（已经存在的MIT KDC）。
KDC hosts中输入"u1404.ambari.apache.org"，在realm中输入"AMBARI.APACHE.ORG"。点击test KDC connection按钮，应显示连接成功。
Kadmin host中输入"u1404.ambari.apache.org"，在Admin principal中输入"root/admin@AMBARI.APACHE.ORG"，输入密码，点击next按钮。
页面切换到了“Install and Test Kerberos Client”，并开始安装kerberos client。<br>
安装完成kerberos client后执行测试，若测试出错，说kadmin找不到。猜测是u1401节点未安装kerberos客户端导致，所以在u1401节点上执行：
```
apt install krb5-user  (如果提示包依赖错误，就用手机上网执行apt-get udpate)
```
