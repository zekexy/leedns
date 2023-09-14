## 编译

### Linux
```
git clone https://github.com/zekexy/leedns
cd ./leedns
make build
```

### Docker
```
git clone https://github.com/zekexy/leedns
cd ./leedns
make build-docker
```

## 配置文件
```yaml
## 监听下游
listener:
  ## udp
  - type: udp
    addr: 0.0.0.0:5353
  ## tcp
  - type: tcp
    addr: 0.0.0.0:5353
#  ## http
#  - type: http
#    addr: 0.0.0.0:8080
#    ### 设置 http 路径, 默认为 /dns-query
#    http-path: /dns-query
#  ## tcp-tls
#  - type: tls
#    addr: 0.0.0.0:8053
#    ### tls 以及 https 必须设置证书
#    certfile: /path/server.crt
#    keyfile: /path/server.key
#  ## https
#  - type: https
#    addr: 0.0.0.0:8443
#    certfile: /path/server.crt
#    keyfile: /path/server.key
#    http-path: /dns-query

## 上游服务器
upstream:
  ## udp: 未指定端口时默认为 53
  - url: udp://8.8.8.8:53
    weight: 10 # 权重, 仅在 strategy 设置为负载均衡时有效，且负载均衡模式下未指定 weight 有效值(>=0)时此上游服务器将被忽略
  ## tcp: 未指定端口时默认为 53
  - url: tcp://8.8.8.8:53
    weight: 10
  ## dns over tls: 未指定端口时默认为 853
  - url: tls://8.8.8.8:853
    weight: 10
  ## dns over https: 未指定端口时默认为 80(http) 或者 443(https)
  - url: https://cloudflare-dns.com/dns-query
    weight: 10

## 用于解析 upstream 中 Servers 的域名, 仅支持 IP
## 此项设置可以为空, 但要保证 upstream 中有至少一个可用的 Host 为 IP 的 Server
bootstrap:
  - udp://1.0.0.1
  - udp://8.8.8.8

# 是否缓存 dns 记录，默认为 true
cache: true

# 向上游查询的方式, 支持并发(concurrent, 默认)、随机(random)、fallback以及负载均衡(load-balanced)
strategy: concurrent

# 向 upstream 查询出错后重试的最大次数, 出错并重试超过此次数后一定时间内不会再向该 upstream 查询
# 如果所有 upstream 的都因达到最大重试次数而失效, 则会将所有的 upstream 的重试次数重置, 默认值为 5
max-retries: 5

# hosts 文件位置, 首先会查询此 hosts 文件, 未设置则不会查询, 即没有默认 hosts 文件
hosts: /etc/hosts
```

## 运行
### Linux
默认配置文件为 /etc/leedns/config.yaml, 或使用 -c(--config) 指定
```
./target/leedns --config ./config_example.yaml
```

### Docker
用默认配置文件启动
```
docker run -d --net=host leedns:latest
```
或者 (假设宿主机的 /etc/leedns 目录存在且包含一个名为 config.yaml 配置文件)
```
docker run -d --net=host -v /etc/leedns:/etc/leedns leedns:latest
```
