# Nginx 常用功能(转载)
> Nginx应该是现在最火的web和反向代理服务器。

## 一、静态代理

> Nginx擅长处理静态文件，是非常好的图片、文件服务器。把所有的静态资源的放到nginx上，可以使应用动静分离，性能更好。

## 二、负载均衡

> Nginx通过反向代理可以实现服务的负载均衡，避免了服务器单节点故障，把请求按照一定的策略转发到不同的服务器上，达到负载的效果。常用的负载均衡策略有
```
#access_log logs/host access log main;
upstream serviceCluster{
 server 10.12.66.186:9431;
 server 10.12.66.187:9431;
}
upstream service2Cluster{
 server 10.12.66.186:9431 weight=1;
 server 10.12.66.187:9431 weight=2;
 server 10.12.66.188:9431 weight=2
}
upstream service3CLuster{
 server 10.12.66.186:9433;
 server 10.12.66.187:9433;
}
```

* 1、轮询

将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。

* 2、加权轮询

不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此它们的抗压能力也不相同。给配置高、负载低的机器配置更高的权重，让其处理更多的请；而配置低、负载高的机器，给其分配较低的权重，降低其系统负载，加权轮询能很好地处理这一问题，并将请求顺序且按照权重分配到后端。

* 3、ip_hash（源地址哈希法）

根据获取客户端的IP地址，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是客户端要访问服务器的序号。采用源地址哈希法进行负载均衡，同一IP地址的客户端，当后端服务器列表不变时，它每次都会映射到同一台后端服务器进行访问。

* 4、随机

通过系统的随机算法，根据后端服务器的列表大小值来随机选取其中的一台服务器进行访问。

* 5、least_conn（最小连接数法）

由于后端服务器的配置不尽相同，对于请求的处理有快有慢，最小连接数法根据后端服务器当前的连接情况，动态地选取其中当前积压连接数最少的一台服务器来处理当前的请求，尽可能地提高后端服务的利用效率，将负责合理地分流到每一台服务器。

## 三、限流

> Nginx的限流模块，是基于漏桶算法实现的，在高并发的场景下非常实用。
```
limit_req_zone $binary_remote_addr zone=my limit:10m rate=100r/s;

location /service {
    limit_req zone = mylimit burst=20 nodelay:
    proxy_pass http://servicecluster;
}

location /service2 {
    limit_req zone = mylimit burst=20 nodelay;
    proxy_pass http://service2ctusten
}

location / {
    root html;
    index index.html index.html;
}

```


* 1、配置参数

1）limit_req_zone定义在http块中，$binary_remote_addr 表示保存客户端IP地址的二进制形式。

2）Zone定义IP状态及URL访问频率的共享内存区域。zone=keyword标识区域的名字，以及冒号后面跟区域大小。16000个IP地址的状态信息约1MB，所以示例中区域可以存储160000个IP地址。

3）Rate定义最大请求速率。示例中速率不能超过每秒100个请求。

* 2、设置限流

burst排队大小，nodelay不限制单个请求间的时间。

## 四、缓存

* 1、浏览器缓存，静态资源缓存用expire。
```
location ~ .*\.(?:jpg|jpeg|gif|png|cur|gz|svg|svgz|mp4|ogg|ogv|webm)$ {
    expires 7d;
}

location ~ .*\.(?:js|css)${
    expires 7d;
}
```


* 2、代理层缓存
```

proxy_cache_path /data/cache/nginx levels=1:2 keys_zone=cache:512m inactive = 1d max_size=8g;

location / {
    location ~ \.(htm|html)?$ {
        proxy_cache cache;
        proxycache_key $uri$is_argssargs;   //以此变量值做hash,作为KEY
        add_header X-Cache $upstream_cache_status;
        proxy-cache_valid 200 10m;
        proxy_cache_valid any 1m;
        proxy_pass http://real_server;
        proxy_redirect off;
    }

    location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)$ {
        root /data/webapps/edc;
        expires 3d:
        add_header Static Nginx-Proxy:
    }
}
```

## 五、黑白名单

* 1、不限流白名单
```
geo $limit {
    122.16.11.0/24 0;
}

map $limit $limit_key {
    1 $binary_remote_addr;
    0 "";
}

limit_req_zone $limit_key zone=myLimit:10m rate=1r/s;

location / {
    limit_req zone=mylimit burst=1 nodelay;
    proxy_pass http://service3cluster;
}

```


* 2、黑名单
```

location / {
    deny 10.52.119.21;
    deny 122.12.1.0/24
    allow 10.1.1.0/16;
    allow 1001:0db8::/32;
    deny all;
}

```

