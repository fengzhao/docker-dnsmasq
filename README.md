# 自建DNS



企业内部用户量到达一个规模时，优化网络流量成了一个问题。其中一个办法便是自建 DNS 服务器，减小 DNS 请求在边界网关的转发量，空闲出更多的带宽。


目前使用最广的 DNS 服务器软件是 Bind(Berkeley Internet Name Domain)，支持多种平台。默认使用 TCP 53/UDP 53 端口进行服务。

客户端查询服务时使用的是 UDP 的 53 端口，TCP 的端口在主辅 DNS 之间同步和传输数据使用。

有一种特殊情况，客户端发出DNS查询请求之后，接受到的应答总长度超过 512 字节，之后使用 TCP 重发查询请求。



# Dnsmasq详解

Dnsmasq为小型网络提供网络基础设施：DNS，DHCP，路由器通告和网络引导。它被设计为轻量级且占用空间小，适用于资源受限的路由器和防火墙。

它还被广泛用于智能手机和便携式热点的共享，并支持虚拟化框架中的虚拟网络。


Dnsmasq原理：


- 本机APP访问主机的/etc/resolv.conf获取DNSServer，该文件指向的DNSServer为Dnsmasq。
- 本地局域网中的主机可以直接访问Dnsmasq，即在这些主机中/etc/resolv.conf指向了Dnsmasq。
- Dnsmasq需要通过上游DNS来进行域名解析，上游DNS可以配置在/etc/resolv.dnsmasq.conf中，该文件需要在Dnsmasq的配置文件/etc/dnsmasq.conf中指定。


## 安装

dnsmasq在各个发行版自带软件仓库基本都会有，使用对应包管理安装命令安装即可：

```bash
yum install -y dnsmasq     
apt-get install -y dnsmasq    
pacman -Sy dnsmasq/yay -y dnsmasq
```

## 配置

配置文件在/etc/dnsmasq.conf，/etc/dnsmasq.d/目录中存放的是一些自定义配置

```ini
# 主配置文件：/etc/dnsmasq.conf
# 配置侦听地址与端口，dns服务侦听端口，默认就是53，可不配
port=53
# dns服务侦听接口，如果需要侦听多个接口，可重复配置多条
interface={{ 本机网络接口 }}
# dns服务侦听地址，如果需要侦听多个地址，可重复配置多条
listen-address={{ 本机ip地址 }}



##############################################################################
#################################DNS选项######################################
# 不加载本地的 /etc/hosts 文件
#no-hosts

# 添加读取额外的 hosts 文件路径，可以多次指定。如果指定为目录，则读取目录中的所有文件。
#addn-hosts=/etc/hosts
# 读取目录中的所有文件，文件更新将自动读取
#hostsdir=<path>


# 缓存时间设置，一般不需要设置
# 本地 hosts 文件的缓存时间，通常不要求缓存本地，这样更改hosts文件后就即时生效。
#local-ttl=3600


##############################################################################
# 记录dns查询日志
#log-queries
# 设置日志记录器，‘-‘ 为 stderr，也可以是文件路径。默认为：DAEMON，调试时使用 LOCAL0。
#log-facility=<facility>
#log-facility=/var/log/dnsmasq/dnsmasq.log
# 异步log，缓解阻塞，提高性能。默认为5，最大100。
#log-async[=<lines>]
#log-async=50
 
##############################################################################
# 指定用户和组
#user=nobody
#group=nobody
 
##############################################################################
```




###  上游DNS

```ini
# 上游dns配置文件：/etc/dnsmasq.d/public_upstream.conf

# 上游dns服务器地址配置文件路径
resolv-file=/etc/resolv.dnsmasq.conf
# 使上游dns服务器查询按照配置文件中的顺序查询而不是同时查询
strict-order
```


```ini
# 上游dns服务地址配置文件：/etc/resolv.dnsmasq.conf

nameserver 8.8.8.8
nameserver 114.114.114.114
```


## 服务管理

启动dnsmasq服务并配置开机自启动

```shell
systemctl start dnsmasq
systemctl enable dnsmasq
```


## 国内外分流

使用dnsmasq-china-list作为大陆域名白名单，定义国内域名使用的上游DNS，不匹配的则走dnsmasq定义的上游DNS，完美利用解析优先级机制。


- accelerated-domains.china.conf           国内域加速 
- google.china.conf                        Google域加速
- apple.china.conf                         Apple域加速      
- bogus-nxdomain.china.conf                反劫持


```bash
cd /opt
git clone https://github.com/felixonmars/dnsmasq-china-list


# 你可以选择直接将上面几个文件从dnsmasq-china-list目录中拷贝到dnsmasq.d目录下，但考虑到这个项目的文件是定时更新维护的。
# 因此超链接的方式更方便，后续只需定时执行git pull更新项目文件即可，无需重新拷贝。
ln -sf /opt/dnsmasq-china-list/accelerated-domains.china.conf  /etc/dnsmasq.d/accelerated-domains.china.conf 
ln -sf /opt/dnsmasq-china-list/google.china.conf /etc/dnsmasq.d/google.china.conf
ln -sf /opt/dnsmasq-china-list/apple.china.conf /etc/dnsmasq.d/apple.china.conf
ln -sf /opt/dnsmasq-china-list/bogus-nxdomain.china.conf /etc/dnsmasq.d/bogus-nxdomain.china.conf 
```

上一步可见国内域名默认都是指定114的DNS作为上游，你可以选择替换为运营商分配给你的LDNS，即本地出口DNS，LDNS可以通过[此网站](https://ping.huatuo.qq.com/)查询。

假定LDNS为113.87.49.47，那么替换命令可以这么写


```bash
sed -i 's|114.114.114.114|113.87.49.47|g' accelerated-domains.china.conf
```




# 用法



主仓库地址：https://github.com/fengzhao/docker-dnsmasq.git


> 声明
>
> 该方案仅用于自建内网DNS解析做参考。如希望提供公网递归解析服务需符合相关政策法规。
>
> 可参阅：https://cloud.tencent.com/document/product/213/35533





```shell
# 拉取镜像
# docker pull ghcr.io/fengzhao/dnsmasq:latest
# docker pull fengzhao-docker.pkg.coding.net/wiki/docker/dnsmasq:latest

mkdir -p /data/dns/

git clone https://github.com/fengzhao/docker-dnsmasq.git

docker build -t ghcr.io/fengzhao/dnsmasq:latest .


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
    
    

# 管理界面  http://host_ip:5380  admin/123456 


```











