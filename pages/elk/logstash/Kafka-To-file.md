# Logstash实现Kafka数据到本地文件的存储
场景：<br>
  * 实现后台与各项目的表数据同步
  * 各项目均有一个kafka生产端，用于查询所属项目的表数据<br>
  * 后台通过logstash收集这些数据，并存入到后台的greenplum数据库<br>
## Kafka生产端
生产的数据格式(JSON)：
```
{"data":"6,65,20151219 09:44:37,20151219 10:08:09,1, ,1,", "tabname":"itm_alarm_snap"}
```
data: csv格式数据（以逗号分隔）<br>
tabname：data数据所属表名称

## Logstash配置
```
input {
  kafka {
    codec => "json"                             # 设置json解析
    group_id => "logstash1"                     # 该消费者所属组id
    auto_offset_reset => "earliest"             # 
    topics => ["test"]                          # kafka主题
    bootstrap_servers => "192.168.73.101:9092"  # Kafka集群实例，可以指定多个，格式：host1:port1,host2:port2
    decorate_events => true
  }
}
filter {
  json {
    source => "message"
  }
}
output {
  stdout {                                      # 标准输出到控制台
    codec => rubydebug
  }
  file {                                                         # 文件输出插件  
    path => "/opt/ldatas/data/%{tabname}_%{+YYYYMMdd-HH}.csv"    #    定义输出文件路径、文件名称，每小时重新创建一个文件
    codec => line { format => "%{data}"}                         #    格式化输出到文件的内容，获取data字段内容
    dir_mode => 0777                                             #    输出文件所在目录的权限设置
    file_mode => 0777                                            #    输出文件的权限设置
    filename_failure => "/opt/ldatas/failures/filefailures.txt"  #    文件路径和名称解析出现错误时，数据的保存路径
  }
}
```

## 数据导入脚本
脚本实现的功能：循环logstash导出的csv文件，判断文件是否正在被logstash使用；若没有被使用，执行postgres导入命令(copy)，若导入成功，将该csv文件移动到../success目录下，若导入失败，则移动到../failures目录下；若该文件被使用，则跳过该csv文件。
/opt/ldatas/bin/copy2postgres.sh（以向表itm_alarm_snap导入数据为例）:
```
#!/bin/bash

curdate=$(date "+%Y%m%d-%H:%M:%S")
DIR="$(cd "$(dirname "$0")" && pwd)"
DIR="$(cd "$(dirname "$DIR")" && pwd)"
logfile="$DIR/logs/exec_$curdate.log"
touch $logfile

files=$(ls $DIR/data/)
for f in $files
do
  tmp="$DIR/data/$f"
  echo "$tmp start" >> $logfile
  if [ -f "$tmp" ]; then
    # 文件存在
    isopen=$(lsof $tmp)
    echo "isopen=$isopen"
    if [ ! $isopen ]; then
      echo "start copying... from $tmp"  >> $logfile
      result=$(su postgres -c "/usr/pgsql-9.6/bin/psql -h 192.168.73.110 \"dbname=kong user=postgres password=123456a?\" -c \"copy itm_alarm_snap(res_id, rule_id, create_time, last_time, status, value, is_pushed, note) from '$tmp' with delimiter as ',' null as '';\" ")
      echo $result >> $logfile
      if [[ "$result" =~ "COPY" ]]; then
        echo "$tmp has been copied to database! this file moving to ../success/ directory"  >> $logfile
        mv $tmp $DIR/success/
      else
        echo "$tmp copied to database is failed! this file is moving to ../failures/ directory"  >> $logfile
        mv $tmp $DIR/failures/
      fi
    else
      # 该文件被logstash打开，表明正在传输数据
      echo "$tmp is opening by logstash" >> $logfile
    fi
  else 
    echo "$tmp is not a file" >> $logfile
  fi
  echo "$tmp end" >> $logfile
done
exit 0
```
## 创建定时任务
适应linux的crontab定时任务：
```
# crontab -e #添加定时任务
```
在打开的定时任务编辑器中，添加一行：
```
10 * * * *  sh /opt/ldatas/bin/copy2postgres.sh
```
每小时10分执行一次脚本，例如：8点10分钟执行了一次，下次再9点10分执行

