# nginx实现web防火墙

## 一、ngx_lua_waf
组件名称 |网址
--- | ---
LuaJIT | http://luajit.org/download/LuaJIT-2.0.5.tar.gz
ngx_devel_kit | https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz
lua-nginx-module | https://github.com/openresty/lua-nginx-module/archive/v0.10.11.tar.gz
ngx_lua_waf | https://github.com/loveshell/ngx_lua_waf/archive/v0.7.2.tar.gz


### 安装LuaJIT
解压LuaJIT-2.0.5.tar.gz，进入解压目录，编译和安装：
```
# tar -xf LuaJIT-2.0.5.tar.gz
# cd LuaJIT-2.0.5
# make && make install
```
### Nginx添加Lua模块
分别解压v0.3.0.tar.gz(ngx_devel_kit模块)和v0.10.11.tar.gz(lua-nginx-module模块):
```
# tar -xf v0.3.0.tar.gz -C /opt/ngx_devel_kit
# tar -xf v0.10.11.tar.gz /opt/lua-nginx-module
```
设置LuaJIT变量：
```
# export LUAJIT_LIB=/usr/local/lib 
# export LUAJIT_INC=/usr/local/include/luajit-2.0
```
重新编译nginx：
```
# cd /opt/nginx-1.12.1
# ./configure --with-http_stub_status_module --with-http_ssl_module --with-http_sub_module --with-http_realip_module --add-module=/opt/ngx_devel_kit --add-module=/opt/lua-nginx-module --with-ld-opt="-Wl,-rpath,$LUAJIT_LIB"
# make -j4 && make install
```
nginx重新编译成功后，lua-nginx-module和ngx_devel_kit模块就安装成功了，下面开始利用开源waf项目配置nginx的web防火墙。
### 配置Waf
解压v0.7.2.tar.gz：
```
# mkdir /usr/local/nginx/conf/waf
# tar -xf v0.7.2.tar.gz -C /usr/local/nginx/conf/waf
```
修改ngx_lua_waf的配置(/usr/local/nginx/conf/waf/config.lua)：
```
RulePath = "/usr/local/nginx/conf/waf/wafconf/"   # 规则目录
attacklog = "on"                                  # 是否开启日志
logdir = "/usr/local/nginx/logs/hack/"            # 日志目录
UrlDeny="on"                                      # 是否开启url拦截
Redirect="on"                                     # 是否拦截后重定向
CookieMatch="on"                                  # 是否拦截cookie攻击
postMatch="on"                                    # 是否拦截post攻击
whiteModule="on"                                  # 是否开启url白名单
black_fileExt={"php","jsp"}                       # 不允许上传文件类型
ipWhitelist={"127.0.0.1"}                         # ip白名单，逗号分隔
ipBlocklist={"1.0.0.1"}                           # ip黑名单，逗号分隔
CCDeny="on"                                       # 是否开启拦截CC攻击
CCrate="100/60"                                   # 设置CC攻击频率：60s内同一个IP只能请求同一个地址100次
# html为拦截后的警告html内容
html=[[
<html xmlns="http://www.w3.org/1999/xhtml"><head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>网站防火墙</title>
<style>
p {
        line-height:20px;
}
ul{ list-style-type:none;}
li{ list-style-type:none;}
</style>
</head>

<body style=" padding:0; margin:0; font:14px/1.5 Microsoft Yahei, 宋体,sans-serif; color:#555;">

 <div style="margin: 0 auto; width:1000px; padding-top:70px; overflow:hidden;">
  
  
  <div style="width:600px; float:left;">
    <div style=" height:40px; line-height:40px; color:#fff; font-size:16px; overflow:hidden; background:#6bb3f6; padding-left:20px;">网站防火墙 </div>
    <div style="border:1px dashed #cdcece; border-top:none; font-size:14px; background:#fff; color:#555; line-height:24px; height:220px; padding:20px 20px 0 20px; overflow-y:auto;background:#f3f7f9;">
      <p style=" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;"><span style=" font-weight:600; color:#fc4f03;">您的请求带有不合法参数，已被网站管理员设置拦截！</span></p>
<p style=" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;">可能原因：您提交的内容包含危险的攻击请求</p>
<p style=" margin-top:12px; margin-bottom:12px; margin-left:0px; margin-right:0px; -qt-block-indent:1; text-indent:0px;">如何解决：</p>
<ul style="margin-top: 0px; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; -qt-list-indent: 1;"><li style=" margin-top:12px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;">1）检查提交内容；</li>
<li style=" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;">2）如网站托管，请联系空间提供商；</li>
<li style=" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;">3）普通网站访客，请联系网站管理员；</li></ul>
    </div>
  </div>
</div>
</body></html>
]]
```
修改nginx配置文件nginx.conf，增加ngx_lua_waf相关配置：
```
......
http {
    ......    
    lua_package_path /usr/local/nginx/conf/waf/?.lua;   # ngx_lua_waf解压目录，匹配所有lua文件
    lua_shared_dict limit 10m;     # 开启拦截cc攻击时设置
    init_by_lua_file /usr/local/nginx/conf/waf/init.lua;  # 指定初始化lua文件
    access_by_lua_file /usr/local/nginx/conf/waf/waf.lua;  # 指定访问过滤lua文件
    ......
    server {
        listen       80;
        server_name  localhost;
        access_log  logs/localhost.access.log;
        
        location / {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Referer $invalid_referer;
        } 
    }
    ......    

}

```
配置完成后，重启nginx
```
# cd /usr/local/nginx/sbin/
# ./nginx -t
# ./nginx -s reload
```
浏览器输入：http://192.168.73.104/test?id=<script>alert('123');</script>

![image](http://fogray.github.io/fogray/images/nginx/ngx_lua_waf_error.png)
带有js脚本参数的请求被拦截，查看http status会为403

## 二、HTTP请求method防篡改
原理：nginx设置可被允许的method类型，禁止类型返回405状态码。
```
......
        location / {
            ......
            add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT';
            if ($request_method ~ 'BOGUS'){
              return 405;
            }
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
......
```
上述配置表示：设置允许的method类型为GET、POST、PUT三种请求，禁止method类型为BOGUS的http请求。
## 三、跨站点请求伪造
使用nginx的valid_referers命令，匹配指定referers，若不匹配，返回错误状态：
```
......
        location / {
            ......
            valid_referers none blocked *.example.com c7304.example.com;
            if ($invalid_referer) {
                return 403;
            }
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
......
```
上述配置表示：获取请求header的Referer值，若为空、或者可以匹配设置的*.example.com或c7304.example.com，请求正常进行，否则返回403状态。
另外，也可以不允许Referer为空，配置如下：
```
        location / {
            ......
            valid_referers *.example.com c7304.example.com;
            if ($invalid_referer) {
                return 403;
            }
            ......
        }
```
