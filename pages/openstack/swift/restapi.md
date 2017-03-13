##1、首先认证用户

keystone验证获取X-Subject-Token的头信息
```
curl -si -H "Content-Type: application/json" \
        -d '{  "auth": {  "identity": {"methods":["password"],"password":{"user":{"domain":{"name":"default"},"name":"admin","password":"admin"}}} \
                          ,"scope":{"project":{"domain":{"name":"default"}, "name":"admin"}}}}' \
http://10.0.8.170:5000/v3/auth/tokens
                          
```
结果：
```
HTTP/1.1 201 Created
Date: Mon, 13 Mar 2017 02:46:29 GMT
Server: Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5
X-Subject-Token: gAAAAABYxggFopwJEUZqkOvJhhvaLJ6mlmyrzrDqMAtU88tH0H8ydfgjKN59ZnQ3oULh41bQxylcWYkbSziEb4LiRCAm-4OUjN32xAPfI8pX8AznBDzV-Ch7v1H3e5i_6Ni88Dn1fNOPqXeI9mr2ckqj7qAzs87TQShKz3_kIaIWPb2IowyDIuE
Vary: X-Auth-Token
...
```
记录下头部的X-Subject-Token值

##2、swift rest api
###account信息
查询account详情(container列表等)：
```
curl -si -H 'X-Auth-Token: $token' -X GET http://10.0.8.170:8080/v1/AUTH_15db29268dce42bab6a6f8813c86bbf2
HTTP/1.1 200 OK
Content-Length: 11
X-Account-Object-Count: 4
X-Account-Storage-Policy-Policy-0-Bytes-Used: 37
X-Account-Storage-Policy-Policy-0-Container-Count: 1
X-Timestamp: 1487817421.14726
X-Account-Storage-Policy-Policy-0-Object-Count: 4
X-Account-Bytes-Used: 37
X-Account-Container-Count: 1
Content-Type: text/plain; charset=utf-8
Accept-Ranges: bytes
x-account-project-domain-id: 027f20c08b4744db836eb448e0a8af6a
X-Trans-Id: tx26624f05b56848de96e36-0058c60e55
Date: Mon, 13 Mar 2017 03:13:25 GMT
```
其他account api:  -X POST修改account元数据、-X HEAD显示account元数据

###Container相关
创建container
```
curl -si -H 'X-Auth-Token: $token' -X PUT http://10.0.8.170:8080/v1/AUTH_15db29268dce42bab6a6f8813c86bbf2/container1
HTTP/1.1 201 Created
Content-Length: 0
Content-Type: text/html; charset=UTF-8
X-Trans-Id: txe72c5ae7afe1462bb6134-0058c60f71
Date: Mon, 13 Mar 2017 03:18:09 GMT
```
查询container: -X GET
修改container: -X POST
删除container: -X DELETE

###Object对象相关
创建/修改一个对象(创建一个test.txt文本，内容是hello swift object):
```
curl -si -H 'X-Auth-Token: $token' -X PUT http://10.0.8.170:8080/v1/AUTH_15db29268dce42bab6a6f8813c86bbf2/container1/test.txt -d 'hello swift object'

HTTP/1.1 201 Created
Last-Modified: Mon, 13 Mar 2017 03:23:34 GMT
Content-Length: 0
Etag: f74fbc9984123b6c1c791848ad00c529
Content-Type: text/html; charset=UTF-8
X-Trans-Id: tx5c635a3918f34966bd343-0058c610b4
Date: Mon, 13 Mar 2017 03:23:33 GMT

```
查看/下载一个对象： -X GET
删除一个对象：  -X DELETE
复制一个对象：    -X COPY -H 'Destination: container1/test2' 将当前对象复制到container1容器，并命名为test2
