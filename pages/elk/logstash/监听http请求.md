# Logstash监听HTTP请求
将一个http请求POST过来的json格式字符串数据解析，并转储到postgresql数据库中<br>
## Logstash安装
下载安装包
```
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.0.0.tar.gz
```
解压到/opt目录下：
```
tar -xf logstash-6.0.0.tar.gz -C /opt/
```
安装jdbc-output插件：
```
cd /opt/logstash-6.0.0/bin
./logstash-plugin install logstash-output-jdbc
```
## Logstash配置文件
/opt/logstash/conf/http-log.conf:
```
input {
  http {
    host => "0.0.0.0"   #监听所有请求
    port => 9000        #监听服务端口
    ssl => false        #是否启用https
    threads => 2        #线程数
  }
}

filter{                  #过滤插件
  json{
   source => "message"   #过滤出message字段的内容，即只传递http请求过来的JSON数据，不需要logstash额外的key
  }
}

output {                 #输出
  stdout{                #标准输出插件，用作测试
    codec=>rubydebug{}
  }
  jdbc {                 #jdbc输出插件
    connection_string => "jdbc:postgresql://192.168.73.110:5432/kong?user=kong&password=123456a?"    #指定postgresql数据库的jdbc连接字符串
    statement => ["insert into OD_KONG_LOG(API_ID, API_NAME, CONSUMER_ID, CONSUMER_USERNAME, URI, QUERTYSTRING, REQUEST_SIZE, METHOD, CLIENT_ID, LATENCIES, RESPONSE_SIZE, STARTED_AT) values (?, ?, ?, ?, ?, ?, CAST(? as integer), ?, ?, ?, CAST(? as integer), to_timestamp(?))", "%{[api][id]}", "%{[api][name]}", "%{[consumer][id]}", "%{[consumer][username]}", "%{[request][uri]}", "%{[request][querystring]}", "%{[request][size]}", "%{[request][method]}", "client_ip", "latencies", "%{[response][size]}", "started_at"]
  }
}
```
input：<br>
  http: http输入插件，监听http请求
filter：<br>
  json：JSON过滤插件，将http请求数据解析成json格式，并可以重新处理成想要的数据
output：<br>
  jdbc：jdbc输出插件，将经过过滤插件过滤后的数据，输出到数据库表中
    connection_string：jdbc连接字符串
    statement：执行的数据更新SQL，拼接SQL中使用json数据的格式：%{[api][id]}
## 启动logstash
```
cd /opt/logstash/bin
/logstash -f ../conf/http-log.conf &
```
## 测试
```
curl -X POST http://127.0.0.1:9000 -d '{ "consumer": { "created_at": 1506494530000, "username": "webb", "id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c" }, "api": { "created_at": 1510620873000, "strip_uri": true, "id": "7a2ab4e9-325b-4922-9ad1-7c53c5e0ec1e", "name": "example-api2", "http_if_terminated": false, "https_only": false, "upstream_url": "http://webdav.imaicloud.com/", "uris": [ "/my-path" ], "preserve_host": false, "upstream_connect_timeout": 60000, "upstream_read_timeout": 60000, "upstream_send_timeout": 60000, "retries": 5 }, "request": { "querystring": {}, "size": "90", "uri": "/my-path", "request_uri": "http://kong:8000/my-path", "method": "GET", "headers": { "host": "kong:8000", "x-consumer-username": "webb", "user-agent": "curl/7.29.0", "accept": "*/*", "x-consumer-id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c", "apikey": "1" } }, "client_ip": "192.168.73.102", "authenticated_entity": { "consumer_id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c", "id": "c0ca1b8c-0d38-4b18-a77e-f69c35974c03" }, "latencies": { "request": 45, "kong": 0, "proxy": 45 }, "response": { "headers": { "content-type": "text/html; charset=UTF-8", "date": "Tue, 14 Nov 2017 09:28:19 GMT", "connection": "close", "via": "kong/0.11.0", "access-control-allow-headers": "DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,X-Auth-Token", "x-kong-proxy-latency": "0", "server": "nginx/1.10.3", "transfer-encoding": "chunked", "x-kong-upstream-latency": "45", "access-control-allow-origin": "*", "access-control-allow-methods": "GET, POST, OPTIONS,DELETE, PUT, COPY, MOVE,MKCOL" }, "status": 200, "size": "1991" }, "tries": [ { "balancer_latency": 0, "port": 80, "ip": "60.216.42.102" } ], "started_at": 1510651699463 }'
```
模拟http的post请求结束后，查看postgres数据表【OD_KONG_LOG】，是否成功插入
