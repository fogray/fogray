# Hue + OpenLDAP集成
## Hue配置
配置文件：/opt/hue/desktop/conf/pseudo-distributed.ini
```
[desktop]
  [[auth]]
    backend=desktop.auth.backend.LdapBackend #使用ldap认证后端
  [[ldap]]
    ldap_url=ldap://u1401.ambari.apache.org:389  #ldap service url
    bind_dn="cn=admin,dc=ambari,dc=apache,dc=org"  #ldap admin dn
    bind_password=vagrant  #ldap admin dn密码
    ldap_username_pattern="uid=<username>,ou=people,dc=ambari,dc=apache,dc=org"  #ldap 用户模板，<username>是在登录页面输入的用户名
    create_users_on_login = true   #登录hue的同时，在hue创建ldap的用户
    sync_groups_on_login=true  #登录hue的同时，同步用户组
    ignore_username_case=true   #忽略用户名大小写
    search_bind_authentication=true
    [[[users]]]
      user_filter="objectclass=person"
      user_name_attr=uid
    [[[groups]]]
      group_filter="objectclass=group"
      group_name_attr=member
      group_member_attr=distinguishedName
    
```
保存并退出，重新启动hue
```
$ cd /opt/hue
$ ./build/env/bin/hue runserver 192.168.14.103:8000
```
使用ldap用户登录hue
