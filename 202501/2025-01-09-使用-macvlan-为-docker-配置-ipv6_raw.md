Title: 使用 macvlan 为 Docker 配置 IPv6

URL Source: https://blog.xm.mk/posts/73f9/

Published Time: 2023-05-03T20:08:10+08:00

Markdown Content:
Cyrus's Blog湫谷
文章
标签
分类
友链
主页
文章
使用 macvlan 为 Docker 配置 IPv6
使用 macvlan 为 Docker 配置 IPv6
 Cyrus 收录于  网络  Docker  群晖
2023-05-03 约 3200 字 预计阅读 7 分钟 - 次阅读 
前言
Docker 的 IPv6 支持

Docker 容器默认不支持 IPv6，但使用 qBittorrent 等工具时，很多地区在不提供公网 IPV4 的情况下可能会有公网 IPv6 地址，有 IPv6 地址会让连接性好很多，以及可以通过公网访问容器服务。

为 Docker 配置 IPv6 有三种常见的思路，一种是通过 HOST 模式，直接使用主机网络，但是可能引来外部扫描和攻击，存在十分严重的安全性问题。IPv6 号称可以为每一粒沙子都分配一个 IP 地址，因此完全可以为每个容器都分配一个独立的 IPv6 地址，重新找回 IPv6 超大地址空间的安全优势。

第二种是启用官方的 IPv6 支持，在官方文档中有说明，但是启用后仅能获取到私网（link-local）IPv6 地址，想要获取公网（global）IPv6 地址，还要指定--fixed-cidr-v6，在目前国内运行商随机分配前缀的情况下，配置较为麻烦。

第三种也就是本文使用的方法是通过 macvlan，为 docker 创建一个支持 IPv6 的网络，自动从上级路由获取公网 IPv6 地址。

关于 IPv6

通常情况下，客户端会同时存在多个 IPv6 地址，一般包括：

Loopback 地址：::1，等价于 IPv4 中的 127.0.0.1。
链路本地地址（Link Local Address）：fe80::/10网段，只在区域中是合法地址，类似于 IPv4 中的 169.254.0.0/16。
唯一区域地址（Unique Local Addresses）：私有地址空间，只能用于本地通信，在 IPv6 Internet 上无法被路由。和 IPv4 的专用网络地址10.0.0.0/8、172.16.0.0/12和192.168.0.0/16类似，在RFC 4193中定义。固定前缀是fc00::/7，分为fc00::/8和fd00::/8两部分，fc00::/8目前作为保留地址没有定义，fd00::/8是现在使用的 ULA 地址，其中 40bit 随机生成以避免冲突，可以使用ULA 生成工具 来生成。
全局地址（Global Address Type）：通常意义上的公网地址，以2XXX:开头。但2002:开头的是特殊的 6to4 过渡地址，不是原生的纯 IPv6 网路地址。
本地站点地址（Site Local Address）：fec0::/10网段，私网地址，目前在 RFC 规范中已被废弃。

此外，考虑到安全等因素，客户端会同时存在多个公网 IP 地址，一个是 SLAAC 产生的地址，基于EUI-64标准和网卡生成，不作为首选通信地址，即下方的IPv6 地址；另一个是 SLAAC 产生的临时地址，用于网络通信，暴露于公网，有效期较短，用于保护隐私，即下方的临时 IPv6 地址。

1
2
3
4

	
连接特定的 DNS 后缀 . . . . . . . : lan
IPv6 地址 . . . . . . . . . . . . : 2409:8a55:260:1a60:a183:ee9e:98c3:df85
临时 IPv6 地址. . . . . . . . . . : 2409:8a55:260:1a60:fdf8:8167:38cd:b57c
本地链接 IPv6 地址. . . . . . . . : fe80::a183:ee9e:98c3:df85%14
Docker 配置
创建网络

在使用 macvlan 为 Docker 开启 IPv6 支持时，与 IPv4 的 macvlan 配置过程类似，需要创建一个配置了 v6 子网的 IPv6 网络。可通过以下命令配置：

1
2
3
4
5
6
7
8

	
docker network create -d macvlan \
--subnet=192.168.1.0/24 \
--gateway=192.168.1.1 \
--ipv6 \
--subnet=fd0d:7eb5:2afd::/64 \
--gateway=fd0d:7eb5:2afd::1 \
-o parent=ovs_eth0 \
bridge-host

其中，

第一组--subnet和--gateway是 IPv4 子网段和网关，根据路由器配置进行设置；
第二组--subnet和--gateway是为 IPv6 配置的 ULA 网段，根据上面的内容，这部分是私有网段，与宿主机的 IP 设置和路由器设置无关，不会用于公网通信，因此可以自行设置。容器会自动从上级路由获取公网 IPv6 地址。
-o parent指定了出口网卡，群晖系统一般是ovs_eth0，需要通过ifconfig或ip a命令查看实际的出口网卡，还可能是en0、eth0等等，务必修改为正确的网卡。
最后的bridge-host是网络名，可以自行命名。
为容器应用网络
Docker Cli

使用命令行创建容器时，通过--network指定网络，有需要还可以通过--ip配置固定 IPv4 地址，另外 macvlan 模式下无需配置端口映射。

例如：

1

	
docker run -itd --name test --ip=192.168.1.201 --network bridge-host busybox
使用 docker compose

使用 compose 时，通过networks段指定网络配置，例如以qBittorrent为例：

 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26

	
version: '3'
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - WEBUI_PORT=8080
    volumes:
      - /path/to/appdata/config:/config
      - /path/to/downloads:/downloads
    restart: unless-stopped
    networks: # 指定网络
      bridge-host:
        ipv4_address: 192.168.1.202
    dns:  #可选，指定 dns 服务器
      - 223.5.5.5
      - 119.29.29.29
      - "2400:3200::1"
      - "2402:4e00::"

networks:
    bridge-host: # 引用之前创建的外部网络
      external: true

至此，docker 应该可以顺利使用 IPv6 网络了，可以进入容器内部通过ifconfig或ip a命令查看网络配置是否正常。

解决容器和宿主机通信问题

在 macvlan 模式下，出于安全考虑，默认禁止宿主机与容器的通信，也就是宿主机无法访问以及反代容器。但是局域网中的其他设备可以访问到容器。此时的网络拓扑如下图所示，可以将接入macvlan的容器看作一个直接接入局域网的设备（蓝色线），只是无法与宿主机（以及宿主机中使用brigde模式的设备）直接通信（红色线）。

但 macvlan 之间可以通信，因此通过新建一个虚拟接口，然后修改路由，使得主机<->容器变为主机<->虚拟接口<->容器，就可以打通容器与宿主机之间的通信。

配置过程

使用 root 用户执行以下命令，创建一个名为macvlan-host的网卡，连接到ovs_eth0物理网卡上（需要根据实际情况修改网卡名字）

1

	
ip link add macvlan-host link ovs_eth0 type macvlan mode bridge

设置该虚拟接口的 IP，需要在 macvlan 网段内

1

	
ip addr add 192.168.1.254 dev macvlan-host

启动接口

1

	
ip link set macvlan-host up

修改路由，其中192.168.1.201、192.168.1.202是容器 ip，如果有多个容器，需要分别设置

1
2

	
ip route add 192.168.1.201 dev macvlan-host
ip route add 192.168.1.202 dev macvlan-host

至此，即可实现宿主机到容器的访问。

修改完成后的网络拓扑类似下图。通过手动配置静态路由，将192.168.1.201、192.168.1.202的路由指向新建的 macvlan-host 虚拟接口，由于该接口视作一个局域网设备可以访问到容器，因此从主机->容器变得可达。（粉色线）

反过来，若是 macvlan 下的容器需要访问主机，直接访问主机 IP 也是不通的（红色线），但此时访问主机可以直接使用虚拟接口的IP来替代（绿色线），即在macvlan容器下使用 192.168.1.254:port 即可正常访问到主机服务。

持久化配置

由于该网卡和路由设置在宿主机重启后会丢失，因此需要创建开机启动脚本，以实现自动配置。

使用 Systemd

Debian 等 Linux 可以用 Systemd 管理 macvlan，该方法可以实现与 docker 服务关联，docker 启动时自动设置 macvlan，启动失败或者退出时还原修改。

群晖也可以（推荐）使用该方法，但与 Linux 有一点不同，且看后文。

具体步骤如下：

将以下脚本放到一个文件夹，例如 /usr/local/bin/setup_docker_macvlan.sh，注意修改其中的变量。并赋予可执行权限：sudo chmod +x /usr/local/bin/setup_docker_macvlan.sh

 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40

	
#!/bin/bash

# macvlan 接口名称[可修改]
MACVLAN_INTERFACE="macvlan-host"
# 桥接网口[需修改]
PARENT_INTERFACE="eth0"
# macvlan 接口 IP [需修改]
IP_ADDRESS="192.168.1.254/24"
# 所有路由列表 [需修改]
ROUTES=("192.168.1.201" "192.168.1.202")

start_macvlan() {
    ip link add $MACVLAN_INTERFACE link $PARENT_INTERFACE type macvlan mode bridge
    ip addr add $IP_ADDRESS dev $MACVLAN_INTERFACE
    ip link set $MACVLAN_INTERFACE up
    for route in "${ROUTES[@]}"; do
        ip route add $route dev $MACVLAN_INTERFACE
    done
}

stop_macvlan() {
    for route in "${ROUTES[@]}"; do
        ip route del $route dev $MACVLAN_INTERFACE || true
    done
    ip link set $MACVLAN_INTERFACE down || true
    ip addr del $IP_ADDRESS dev $MACVLAN_INTERFACE || true
    ip link del $MACVLAN_INTERFACE || true
}

case "$1" in
    start)
        start_macvlan
        ;;
    stop)
        stop_macvlan
        ;;
    *)
        echo "Usage: $0 {start|stop}"
        exit 1
esac

sudo vim /etc/systemd/system/setup_docker_macvlan.service 创建 Systemd 服务单元文件，内容如下：

 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15

	
[Unit]
Description=Setup macvlan shim for docker to allow to route to host
BindsTo=docker.service
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/setup_docker_macvlan.sh start
ExecStop=/usr/local/bin/setup_docker_macvlan.sh stop
RemainAfterExit=yes
Restart=no

[Install]
WantedBy=multi-user.target

如果是群晖用户，由于群晖的 docker systemed 不是 docker.service，因此需要修改 BindsTo、Requires 和 After，根据 DSM 版本不同，服务名称可能也不同，可以通过以下方式查看：

1

	
systemctl list-units --type=service | grep docker

目前已知可能的名称有：

pkgctl-Docker.service
pkg-ContainerManager-dockerd.service
pkg-Docker-dockerd.service

当然也可以使用 network.target，当网络服务设置完成后启动，不强制依赖 docker，是一种更宽松的设置方式。

启用服务单元

1
2

	
sudo systemctl daemon-reload
sudo systemctl enable setup_docker_macvlan.service
群晖 DSM

更简单的配置方式：群晖 DSM7 下可以在 /usr/local/etc/rc.d 中写入启动脚本，例如 vim /usr/local/etc/rc.d/macvlan.sh，然后通过 chmod +x /usr/local/etc/rc.d/macvlan.sh 赋予可执行权限；也可以通过控制面板-任务计划来设置开机脚本。

脚本内容参考：

1
2
3
4
5
6

	
#!/bin/sh
ip link add macvlan-host link ovs_eth0 type macvlan mode bridge
ip addr add 192.168.1.254 dev macvlan-host
ip link set macvlan-host up
ip route add 192.168.1.201 dev macvlan-host
ip route add 192.168.1.202 dev macvlan-host
参考资料
IPv6 - ArchWiki
2. Network part, also known as prefix
IPv6 with Docker
Docker 使用 macvlan 网络容器与宿主机的通信过程 | 一小步
更新于 2023-05-03 
CC BY-NC 4.0
技术笔记
返回 | 主页
毕业了……
PVE 安装 Windows Step By Step
由 Hugo 强力驱动 | 主题 - FixIt
 2018 - 2024CC BY-NC 4.0
网站运行中……
