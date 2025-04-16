Title: docker 中安装 openwrt 并作为旁路由 – 夏至

URL Source: https://mxenc.com/2025/04/04/docker_install_openwrt_and_use_as_bypass_router/

Markdown Content:
在 docker 运行 openwrt 的方式如下：  
网络接口最好使用网线接口，我测试的下来使用 wifi 的接口，macvlan 连不上网，ipvlan 一段时间后宿主机会获取不到 ip 地址，导致宿主机掉线。

如果时使用网线接口，macvlan 可能时更好的方法，这种方法，容器具有独立的 mac 地址和 ip 地址。

```
docker stop openwrt
docker rm openwrt
docker network rm opvlan-net

docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=enp3s0 \
  opvlan-net

docker run \
  --name openwrt \
  --network opvlan-net \
  --privileged \
  --sysctl net.ipv6.conf.all.disable_ipv6=0 \
  -d xinebf/openwrt:24.10.0-x86-64 /sbin/init

docker exec openwrt ip addr show eth0 | grep 'inet '
# You can access the router's backend using the following command.
docker exec -it openwrt sh
```

如果使用 macvlan 宿主机和容器要想通信需要创建 macvlan 子接口。此时宿主机和容器就能通过 容器 ip 和 hostvlan 中的 ip 进行互相通信了。

```
sudo ip link add hostvlan-net link enp3s0 type macvlan mode bridge
sudo ip addr add 192.168.1.15/24 dev hostvlan-net
sudo ip link set hostvlan-net up

# Clear hostvlan settings
sudo ip link set hostvlan-net down
sudo ip link delete hostvlan-net
```

如果想要使用 ipvlan 命令如下，ipvlan 的方式宿主机和容器可以直接通信。

```
docker network create -d ipvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=wlp1s0 \
  -o ipvlan_mode=l2 \
  opvlan-net
```

在 docker 中使用 openwrt 作为旁路由根据网络情况可以分为如下四种情况。

1.  旁路由网关不启用 IPv6，并且禁用 DHCP，使用主路由提供 DHCP 服务。
2.  旁路由网关不启用 IPv6，并且作为 DHCP 服务器，此时主路由要禁用 DHCP。
3.  旁路由网关启动 IPv6，并且作为 DHCP 服务器，此时主路由需要禁用 DHCPv4，DHCPv6 只需要开启 RA 服务。

下面按照上面四种情况分别进行解释说明，下面所有的命令都假设是在通过上面命令启动的全新的 openwrt 容器内执行。

```
# Step One: Configure the LAN Port's Network
uci set network.lan.device='eth0'
uci set network.lan.ipaddr='192.168.1.6'
uci set network.lan.gateway='192.168.1.1'
uci add_list network.lan.dns='192.168.1.1'

uci commit network
service network restart
```

```
# Step Two: Disable DHCP Service
uci del dhcp.lan.ra
uci del dhcp.lan.dhcpv6
uci set dhcp.lan.ignore='1'

uci commit dhcp
service network restart
```

```
# Step Three: Create a New lan6 Interface
# Modify Firewall Configuration
uci del firewall.cfg02dc81.network
uci add_list firewall.cfg02dc81.network='lan'
uci add_list firewall.cfg02dc81.network='lan6'

# Modify Network Configuration
uci set network.lan6=interface
uci set network.lan6.proto='dhcpv6'
uci set network.lan6.device='@lan'
uci set network.globals.packet_steering='1'
uci set network.lan6.reqaddress='try'
uci set network.lan6.reqprefix='auto'
uci set network.lan6.norelease='1'

# Submit and Apply
uci commit firewall
uci commit network
service network restart
service firewall reload
```

```
# Configure devices with the tag 'direct' to use the main router as their gateway and DNS server.
uci add_list dhcp.lan.dhcp_option='tag:direct,3,192.168.1.1'
uci add_list dhcp.lan.dhcp_option='tag:direct,6,192.168.1.1'
# Configure devices whose tag is not 'proxy' to use the main router as their gateway and DNS server.
# uci add_list dhcp.lan.dhcp_option='tag:!proxy,3,192.168.1.1'
# uci add_list dhcp.lan.dhcp_option='tag:!proxy,6,192.168.1.1'
uci commit network
service network restart
```

1.  旁路由网关不启用 IPv6，并且禁用 DHCP，使用主路由提供 DHCP 服务。

需要执行上面 step one 和 step two 中命令。

2.  旁路由网关不启用 IPv6，并且作为 DHCP 服务器，此时主路由要禁用 DHCP。

只需要执行上面 step one 中的命令。

3.  旁路由网关启动 IPv6，并且作为 DHCP 服务器，此时主路由需要禁用 DHCPv4，DHCPv6 只需要开启 RA 服务。

需要执行上面 step one 和 step three 中的命令。
