# 自建DNS



企业内部用户量到达一个规模时，优化网络流量成了一个问题。其中一个办法便是自建 DNS 服务器，减小 DNS 请求在边界网关的转发量，空闲出更多的带宽。



目前使用最广的 DNS 服务器软件是 Bind(Berkeley Internet Name Domain)，支持多种平台。默认使用 TCP 53/UDP 53 端口进行服务。

客户端查询服务时使用的是UDP的53端口，TCP的端口在主辅DNS之间同步和传输数据使用。

有一种特殊情况，客户端发出DNS查询请求之后，接受到的应答总长度超过512字节，之后使用TCP重发查询请求。





## 用法



主仓库地址：https://github.com/fengzhao/docker-dnsmasq.git

国内镜像地址： https://e.coding.net/fengzhao/wiki/docker-dnsmasq.git



> 声明
>
> 该方案仅用于自建内网DNS解析做参考。如希望提供公网递归解析服务需符合相关政策法规。
>
> 可参阅：https://cloud.tencent.com/document/product/213/35533









```shell
# 拉取镜像
docker pull ghcr.io/fengzhao/dnsmasq:latest

docker pull fengzhao-docker.pkg.coding.net/wiki/docker/dnsmasq:latest

mkdir -p /data/dns/

vi /data/dns/dnsmasq.conf


# 启动容器
docker run \
    --name dnsmasq \
    -d \
    -p 53:53/udp \
    -p 5380:8080 \
    -v /data/dns/dnsmasq.conf:/etc/dnsmasq.conf \
    --log-opt "max-size=100m" \
    -e "HTTP_USER=admin" \
    -e "HTTP_PASS=123456" \
    --restart always \
    ghcr.io/fengzhao/dnsmasq:latest 
#   fengzhao-docker.pkg.coding.net/wiki/docker/dnsmasq:latest
    
    


# 管理界面  http://host_ip:5380  admin/123456 


```











