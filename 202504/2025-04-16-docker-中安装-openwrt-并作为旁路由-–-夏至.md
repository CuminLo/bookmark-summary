# docker 中安装 openwrt 并作为旁路由 – 夏至
- URL: https://mxenc.com/2025/04/04/docker_install_openwrt_and_use_as_bypass_router/
- Added At: 2025-04-16 03:56:07
- [Link To Text](2025-04-16-docker-中安装-openwrt-并作为旁路由-–-夏至_raw.md)

## TL;DR
本文总结了在Docker中使用macvlan和ipvlan模式配置网络的实践方法，重点分析了有线/无线接口的差异，并提供了旁路由场景下的详细配置步骤，包括IPv6设置、DHCP选项和防火墙规则等核心操作命令。

## Summary
1. **网络接口选择**：
   - **有线接口优势**：使用网线接口（如enp3s0）时，macvlan模式能稳定运行，容器可拥有独立MAC和IP地址。
   - **无线接口问题**：
     - macvlan模式下无法联网
     - ipvlan模式会导致宿主机IP丢失而断线

2. **macvlan模式配置**：
   - **创建网络**：
     ```bash
     docker network create -d macvlan \
       --subnet=192.168.1.0/24 \
       --gateway=192.168.1.1 \
       -o parent=enp3s0 \
       opvlan-net
     ```
   - **运行容器**：
     ```bash
     docker run \
       --name openwrt \
       --network opvlan-net \
       --privileged \
       -d xinebf/openwrt:24.10.0-x86-64 /sbin/init
     ```
   - **宿主机通信**：
     - 需创建macvlan子接口：
       ```bash
       sudo ip link add hostvlan-net link enp3s0 type macvlan mode bridge
       sudo ip addr add 192.168.1.15/24 dev hostvlan-net
       ```

3. **ipvlan模式配置**：
   - **命令示例**（适用于无线接口wlp1s0）：
     ```bash
     docker network create -d ipvlan \
       --subnet=192.168.1.0/24 \
       --gateway=192.168.1.1 \
       -o parent=wlp1s0 \
       -o ipvlan_mode=l2 \
       opvlan-net
     ```
   - **通信特性**：宿主机与容器可直接通信，无需额外子接口

4. **旁路由配置场景**：
   - **场景1**（禁用IPv6+主路由DHCP）：
     - 执行Step1（LAN口配置）和Step2（关闭DHCP）
     - 关键命令：
       ```bash
       uci set dhcp.lan.ignore='1'
       ```
   - **场景2**（禁用IPv6+旁路由DHCP）：
     - 仅需执行Step1（LAN口配置）
     - 主路由需手动关闭DHCP服务
   - **场景3**（启用IPv6+旁路由DHCP）：
     - 执行Step1和Step3（创建lan6接口）
     - 主路由需：
       - 禁用DHCPv4
       - 仅开启DHCPv6的RA服务

5. **核心配置命令**：
   - **网络基础配置**（Step1）：
     ```bash
     uci set network.lan.ipaddr='192.168.1.6'
     uci add_list network.lan.dns='192.168.1.1'
     ```
   - **IPv6配置**（Step3）：
     ```bash
     uci set network.lan6.proto='dhcpv6'
     uci set network.lan6.reqprefix='auto'
     ```
   - **DHCP选项**：
     ```bash
     uci add_list dhcp.lan.dhcp_option='tag:direct,3,192.168.1.1'
     ```

6. **防火墙配置**：
   - 需同步更新防火墙规则：
     ```bash
     uci add_list firewall.cfg02dc81.network='lan6'
     service firewall reload
     ```

7. **调试技巧**：
   - 查看容器IP：
     ```bash
     docker exec openwrt ip addr show eth0 | grep 'inet '
     ```
   - 进入容器Shell：
     ```bash
     docker exec -it openwrt sh
     ```
