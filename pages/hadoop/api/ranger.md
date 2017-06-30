# Ranger REST API
### 列出所有用户
```
curl -s -u admin:admin -H "Accept: application/json" -H "Content-Type: application/json" \
  -X GET http://c7003.ambari.apache.org:6080/service/xusers/users 
```
### 列出指定id的用户
```
curl -s -u admin:admin -H "Accept: application/json" -H "Content-Type: application/json" \
  -X GET http://c7003.ambari.apache.org:6080/service/xusers/users/{id}
```
### 修改某个用户
```
curl -s -u admin:admin -H "Accept: application/json" -H "Content-Type: application/json" \
  -X POST http://c7003.ambari.apache.org:6080/service/xusers/users/{id} -d \
  '{"id":24,"createDate":"2017-06-29T08:51:54Z","updateDate":"2017-06-29T08:51:54Z","owner":"rangerusersync","updatedBy":"rangerusersync" \
   ,"name":"john","firstName":"john","lastName":"john","password":"123456a?","description":"john - add from Unix box","groupIdList":[] \
   ,"groupNameList":[],"status":1,"isVisible":1,"userSource":1,"userRoleList":["ROLE_USER"]}'
```
