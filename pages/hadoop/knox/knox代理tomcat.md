# Ambari+Knox代理tomcat服务
## Knox配置
### Advanced topology配置
添加tomcat服务
```
<service>
    <role>TOMCAT</role>
    <url>http://c7303.ambari.apache.org:8080</url>
</service>
```
### 创建knox service
登录Knox服务器，/usr/hdp/current/knox-server/data/services/目录下，创建tomcat目录
```
$ cd /usr/hdp/current/knox-server/data/services/
$ mkdir tomcat
$ chown knox:knox tomcat
$ cd tomcat/
$ mkdir 7.0 && chown knox:knox 7.0
$ cd 7.0/
$ vi service.xml     #创建service.xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<service role="TOMCAT" name="tomcat" version="7.0">
   <routes>
     <route path="/tomcatui/">
     </route>

     <route path="/tomcatui/**">
     </route>
     <route path="/tomcatui/**?**">
     </route>

   </routes>
</service>
$ vi rewrite.xml    #创建rewrite.xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<rules>
<!-- Inbound  rewrite rules   -->
<rule dir="IN" name="TOMCAT/root/inbound" pattern="*://*:*/**/tomcatui/">
   <rewrite template="{$serviceUrl[TOMCAT]}/"/>
</rule>

<rule dir="IN" name="TOMCAT/path/inbound" pattern="*://*:*/**/tomcatui/{**}">
    <rewrite template="{$serviceUrl[TOMCAT]}/{**}"/>
</rule>

<rule dir="IN" name="TOMCAT/full/inbound" pattern="*://*:*/**/tomcatui/{**}?{**}">
        <rewrite template="{$serviceUrl[TOMCAT]}/{**}?{**}"/>
</rule>
<rules>

$ chown knox:knox *
```
创建tomcat服务的service.xml和rewrite.xml配置文件后，清空/usr/hdp/current/knox-server/data/deployments/目录下的内容，重启knox服务
### 访问
重启完knox后，通过knox代理访问tomcat服务:https://c7301.ambari.apache.org:8443/gateway/default/tomcatui/
### 问题
重启knox后，若通过knox代理访问tomcat服务失败，
检查/usr/hdp/current/knox-server/data/deployments/default.topo.***/%2F/WEB-INF/目录下的rewrite.xml配置文件，
确认自定义的tomcat服务的rewrite.xml是否写入到该rewrite.xml中，若没有写入，手动copy到该rewrite.xml中对应的位置，重启knox
