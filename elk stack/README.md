https://elkguide.elasticsearch.cn/

# elasticsearch
https://www.elastic.co/downloads/elasticsearch
https://www.cnblogs.com/ajianbeyourself/p/5529575.html
http://blog.51cto.com/zero01/2079879
https://zhaoyanblog.com/archives/732.html
1. 下载tar文件
2. 安装jdk，配置环境变量JAVA_HOME
3. bin/elasticsearch(-d 后台启动)
```
[2018-08-29T00:50:07,172][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:140) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:127) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) ~[elasticsearch-cli-6.4.0.jar:6.4.0]
	at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-6.4.0.jar:6.4.0]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:93) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:86) ~[elasticsearch-6.4.0.jar:6.4.0]
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:104) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:171) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:326) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:136) ~[elasticsearch-6.4.0.jar:6.4.0]
```
* 请新建用户，并赋予权限
  *  useradd elastic
  *  passwd elastic
  *  chmod 777 -R elasticsearch
  *  su elastic
---
```
max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
```
* 请增加配置项(username为你创建的用户)
  *  vim /etc/security/limits.conf
  *  username hard nofile 65536
  *  username soft nofile 65536
---
```
max number of threads [2048] for user [elastic] is too low, increase to at least [4096]
```
* 请修改文件
  *  vim /etc/security/limits.d/*nproc.conf
  *  soft    nproc     1024 -》soft    nproc     2048
---
```
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
* 请增加配置项
  *  vim /etc/sysctl.conf
  *  vm.max_map_count=262144
  *  sh sysctl -p
---
```
如果出现配置外网ip还无法访问问题
```
* 关闭防火墙
  *  systemctl stop firewalld.service #停止firewall 
  *  systemctl disable firewalld.service #禁止firewall开机启动 
  *  firewall-cmd –state 
  *  查看默认防火墙状态（关闭后显示notrunning，开启后显示running）

* 默认配置
```
cluster.name: master-node  # 集群中的名称
node.name: master  # 该节点名称
node.master: true  # 意思是该节点为主节点
node.data: false  # 表示这不是数据节点
network.host: 0.0.0.0  # 监听全部ip，在实际环境中应设置为一个安全的ip
http.port: 9200  # es服务的端口号
discovery.zen.ping.unicast.hosts:["192.168.77.128","192.168.77.130","192.168.77.134"] # 配置自动发现
```

# kibana
https://www.elastic.co/downloads/kibana
1. 下载tar文件
2. ./bin/kibana
* 默认配置文件
```
server.port: 5601
server.host: "192.168.11.67"
elasticsearch.url: "http://192.167.11.67:9200"
```


# logstash
https://www.elastic.co/downloads/logstash
1. 下载tar文件
2. ./bin/logstash
* 默认配置文件
```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://192.168.11.67:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

# filebeat
https://www.elastic.co/downloads/beats/filebeat
1. 下载tar文件
2. ./filebeat
* 默认配置文件
```
filebeat.inputs:
- type: log
  paths:
    - /mnt/log/*.log
output.logstash:
  hosts: ["192.168.11.67:5044"]
```

# packetbeat
https://www.elastic.co/downloads/beats/packetbeat 
1. 下载tar文件
2. ./packetbeat
* 默认配置文件
```
packetbeat.interfaces.device: any
packetbeat.flows:
  timeout: 30s
  period: 10s
packetbeat.protocols:
- type: icmp
  enabled: true
- type: amqp
  ports: [5672]
- type: cassandra
  ports: [9042]
- type: dns
  ports: [53]
  include_authorities: true
  include_additionals: true
- type: http
  ports: [80, 8080, 8000, 5000, 8002]
- type: memcache
  ports: [11211]
- type: mysql
  ports: [3306]
- type: pgsql
  ports: [5432]
- type: redis
  ports: [6379]
- type: thrift
  ports: [9090]
- type: mongodb
  ports: [27017]
- type: nfs
  ports: [2049]
- type: tls
  ports: [443]
output.logstash:
  hosts: ["192.168.11.67:5044"]
```

# topbeat
https://www.elastic.co/downloads/beats/topbeat
1. 下载tar文件
2. ./topbeat -e -c topbeat.yml
* 默认配置文件
```
input:
  period: 60
  procs: [".*"]
  stats:
    system: true
    process: true
    filesystem: true
output:
  logstash:
    hosts: ["192.168.11.67:5044"]
```