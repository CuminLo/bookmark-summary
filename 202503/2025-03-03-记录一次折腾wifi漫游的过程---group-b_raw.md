Title: 记录一次折腾WIFI漫游的过程 - Group B

URL Source: https://www.drxcloud.club/765.html

Markdown Content:
跳至内容
Group B
ZHI WEI 的个人博客
主页
文章
关于
记录一次折腾WIFI漫游的过程
2024年6月2日

自己撸的小工具升级以后，我的X86 OpenWRT终于做下线的准备了，替换它的是小米AC2100。这款路由器是我2019年购买的，一直给X86当作AP使用，机缘巧合之下我又获得一款号称WIFI6的路由器，又将AC2100替换下来，之后便一直放在储物间中。

时过境迁，路由器已经变得花里胡哨起来，什么MESH自组网，全屋覆盖之类的技术宣传的铺天盖地，价格也是蹭蹭的往上涨。技术都是一样的名字，可是却互相不兼容。终于我在咸鱼上闲逛的时候，大数据给我推送了小米AC2100，50包邮到家，于是便有了替换X86，实现WIFI自动漫游这篇文章。

前置知识
技术摘要
802.11系列协议
WIFI信道
信号值dBm
AC2100的参数
刷入OpenWRT固件
配置OpenWRT
容错配置
桥接端口
更新wpad包
安装dawn服务
配置接入点AP
测试验证
漫游802.11r
DAWN配置
前置知识
技术摘要

在开始折腾之前，需要了解相关技术文档中提到的技术名称和其实现的功能，否则在调试的过程中往往会手足无措，或者弄巧成拙。

802.11系列协议
802.11k	向客户端提供接口报告网络中AP的信息，帮助客户端选择最佳AP
802.11v	AP主动通知客户端设备有关网络的信息，包括漫游建议等信息
802.11r	减少客户端在无线网络不同接入点之间切换漫游时的延迟
802.11s	MESH网络，定义了MESH网络的框架和操作，包括节点间的通信和路由
802.11ac/ax	使用ac协议一般称为WIFI5，ax则为WIFI6

ax/ac都支持160Mhz的带宽，但是ac是使用两个80MHz组成的一个160Mhz（ac wave2），这种方式只能最多只能链接一个客户端，且需要客户端网卡支持该协议。

https://www.zhihu.com/tardis/bd/ans/3356511467?source_id=1001

WIFI信道

途中蓝色部分为中国地区可用的WIFI信道，WIFI信道实际就是无线电波的频段，灰色部分因为有其他比如飞机，特种设备之类的使用，所以不开放公众使用。有的时候即使你指定了某个WIFI信道，但实际上无线工作的信道和你指定信道也有可能并不一样，这是因为DFS（Dynamic Frequency Selection），DFS会自动探测这些非民用的无线电信号，并且自动避开。在选择可用的WIFI信道的时候，需要避开或者选择同频段信号较少的信道，这样有助于增强WIFI信号的质量。

信号值dBm

客户端接收到的信号（RSSI）强度，用单位dBm表示，这个数值通常是一个负值。比如-40dBm和-45dBm，因为-40 > -45，所以前者的信号更好。在OpenWRT中，AP属性中有一个字段叫做噪音(noise level)表示当前AP收到干扰程度，这个数值越小说明当前信道的干扰越小，信号质量越好。

单位	强度	描述（WIFI5场景）
-30dBm	强到可怕	把设备直接放在天线旁边
-50dBm	很强	信号非常不错，玩游戏看4K电影是足够的。
-65dBm	优秀	此信号强度对于网络电话和流媒体视频的使用是足够的。
-70dBm	一般	对于网页冲浪和收发邮件的使用是足够的。
-80dBm	较差	实现基本连接，但数据包传输不稳定。
-90dBm	非常差	你将会经常看到类似转圈圈的标志。

最为对比，我将我的IPAD AIR放在AC2100边上，测得的数据为：信号 -32dBm，噪声 -89dBm，信噪比 -57dBm。

AC2100的参数
刷入OpenWRT固件

首先将原厂固件降级至漏洞版本，然后通过URL注入开启SSH，将OpenWRT固件传入/tmp目录下，使用mtd命令写入。网络上有大量教程可以参考，这里给出OpenWRT官方发布的方法。https://openwrt.org/toh/xiaomi/mi_router_ac2100

漏洞固件刷写成功以后，登录并修改管理员密码，注销后重新登录获取当前的stok值，注意不要关闭当前网页。在新窗口中请求下面的URL。

#开启SSH
http://192.168.31.1/cgi-bin/luci/;stok=【替换】/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B


#修改admin密码
http://192.168.31.1/cgi-bin/luci/;stok=【替换】/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20echo%20-e%20'admin%5Cnadmin'%20%7C%20passwd%20root%3B

开启SSH以后，使用root账户进行登录，密码为admin。这里会有一个问题，就是高版本的SSH连接低版本的SSH客户端会提示鉴权失败，需要修改使用下面的命令进行SSH链接。我使用SCP命令将固件发送到路由器上，所以也要修改sshd_config配置。

#高版本连接低版本ssh 使用ssh-ras加密方式
ssh -oHostKeyAlgorithms=+ssh-rsa root@192.168.31.1
#高版本sshd_conf 添加加密方式
HostKeyAlgorithms +ssh-rsa,ssh-dss
#写入固件
mtd write /tmp/openwrt-23.05.3-ramips-mt7621-xiaomi_mi-router-ac2100-squashfs-kernel1.bin kernel1
mtd -r write /tmp/openwrt-23.05.3-ramips-mt7621-xiaomi_mi-router-ac2100-squashfs-rootfs0.bin rootfs0

下面是固件资源，点击可下载

miwifi_r2100_firmware_4b519_2.0.722.bin

openwrt-23.05.3-ramips-mt7621-xiaomi_mi-router-ac2100-squashfs-kernel1.bin

openwrt-23.05.3-ramips-mt7621-xiaomi_mi-router-ac2100-squashfs-rootfs0.bin

配置OpenWRT

在咸鱼上淘了2个AC2100回来以后，我一共拥有三个，分别命名为R1、R2、R3。其中R1是我原有的，R2、R3是咸鱼回来的。R1作为出口路由，配置有一个LAN子网192.168.0.0，提供DHCP服务。设置R2、R3仅提供无线接入点，不提供路由和NAT服务，在我的网络中，我将其看成一个linux终端，而并非路由器，有点类似带网口的AP面板。

容错配置

将网线插入R2的1号网口（OpenWRT中命名为LAN3），登录OpenWRT在【网络】-【接口】-【设备】中修改br-lan桥接的端口，只保留1号网口（LAN3）。这样做可以避免配置错误无法访问路由。然后修改R2的【接口】中修改默认子网LAN地址，避免冲突（因为我的R1子网段和OpenWRT默认的1段并不冲突，所以可以跳过）。

桥接端口

在【设备】中创建一个网桥命名为SWITCH，将WAN、LAN1、LAN2桥接在一起，其他设置保持默认即可。回到【接口】中将WAN6关闭，修改WAN桥接至SWITCH。这里可以创建一个新的防火墙区域（有颜色好看），也可以跳过。添加防火墙策略，允许22和80的分组通过。

这里只是R2本身关闭了IPV6，不影响R1子网中的IPV6DHCP，任何连接到R2的无线客户端，相当于直接连接到了R1的子网，由R1提供DNS和DHCP4/6以及网关服务。

更新wpad包

现在可以将R1的子网接入R2的WAN接口，使R2可以访问互联网。从R2的WAN区域登录R2（即R1的子网中），更新wpad包以支持802.11k/v/r，固件中已经存在的wpad包为精简版本，只支持802.11r，需要删除并且重新安装完整版本，我这里选择了wpad-wolfssl包。参考：https://openwrt.org/docs/guide-user/network/wifi/mesh/80211s

wpad-mesh-openssl（精简版以节省空间）
wpad-openssl (完整大版本)
wpad-mesh-wolfssl（精简版以节省空间）
wpad-wolfssl (完整大版本)
wpad-mesh-mbedtls（精简版以节省空间）
wpad-mbedtls（完整大版本）
该wpad-basic-*版本仅支持802.11r和802.11w。
该wpad-mesh-*版本仅支持802.11r/w和802.11s。
这wpad-*是完整版本，wpad支持802.11k/v/r/w和802.11s。
完整版意味着wpad没有修剪任何内容来减小其尺寸。
opkg update
opkg remove wpad-mini
opkg remove wpad-basic
opkg remove wpad-basic-wolfssl
opkg remove wpad-basic-openssl
opkg remove wpad-basic-mbedtls
opkg install wpad-wolfssl
安装dawn服务

dawn服务是一个“Decentralized Wifi Controller”，可以理解为AC控制器，管理网络中的无线AP，提高漫游以及WIFI性能，特别是当你有很多个AP接入点的时候。其中有很多高级功能，如果需要对其进行更加精细的配置可以参考：https://github.com/berlin-open-wireless-lab/DAWN/blob/master/CONFIGURE.md。这里是dawn的安装方法：https://openwrt.org/docs/guide-user/network/wifi/dawn

opkg update
opkg install dawn luci-app-dawn
#修改配置 
nano /etc/config/dawn
#config network
#	option broadcast_ip '192.168.0.255' #广播地址用来发现ap
#	option broadcast_port '1025'  #端口
#	option network_option '0' #配置0使用udp广播 默认2使用tcp
/etc/init.d/dawn restart #重启服务
reboot

广播地址类似于群发消息，目标是子网中的所有地址，因为R2的WAN口已经桥接在SWITCH上了，所以R2本身也是R1子网中的成员。所以当R1/R3发送广播的时候，R2就会收到dawn数据包。

dawn重启成功后，将R2重启。重启完成重新登录web在顶部就会出现dawn菜单，注意：不要忘记打在防火墙上打开UDP1025端口！参考上文桥接端口小节。

到这里相对于对R2完成了初始化，然后按照上面的方法重复一遍配置R3，然后就可以开始配置无线接入点了。

配置接入点AP

进入【网络】-【无线】中配置radio1（5G），默认已经有一个SSID叫做OpenWRT的配置，修改国家代码（可以筛掉无法使用的信道），选择一个占用比较少的信道，可以使用OpenWRT的信道分析功能查看。设置带宽为80Mhz，发送功率为20dBm（略微降低该值，可以鼓励客户端更主动切换AP）。将网络绑定为SWITCH，在无线安全中选择加密方式为WPA2-PSK，算法自动。MAC过滤和高级设置默认不做修改。

重点在于WLAN漫游菜单，根据dawn的使用说明，需要在无线配置文件中写入bss_transition ‘1’、ieee80211k ‘1’字段。其中ieee80211k就是802.11k协议，bss_transition就是802.11v协议。因为ieee80211v字段已经被移除了，详细配置说明可以参考https://openwrt.org/docs/guide-user/network/wifi/basic，其中就包括了kvr协议的详细配置。

回到luci配置中来，打开802.11r快速切换，NAS ID不填写，移动域不填写，默认为SSID的首字母的HEX表示方法，而不是f457,（启动后可以在/var/run/hostapd-phy*.conf文件中查看https://forum.openwrt.org/t/any-way-to-monitor-if-802-11r-is-working/73504/38）。如果你的SSID全部一致则没有必要修改此参数。重关联截止时间修改为思科推荐的20000，FT协议选择FT over the air，勾选本地生成PMK、802.11k RRM（ieee80211k）、BSS 过渡（bss_transition）选项，其他保持默认。

按照以上步骤配置R3，设置R3的防火墙、dawn服务、以及无线接入点配置，与R2保持一致。开启R2、R3的无线接入点，在LUCI的dawn菜单中查看AP接入点详情。这里dawn已经发现了R1子网中存在的两个无线热点。

测试验证
漫游802.11r

在R1上开启iperf3服务端，ipad使用iperf3客户端进行网速测试，同时在R1上对ipad进行ping大包测试，size设置为20000。我在书房和卧室之间来回走动进行测试，中间需要穿过客厅。为了能准确定位问题，暂时没有对R1配置无线接入点。

测试结果还是令人满意的，我一共来回走动了3次，其中最后一次，在R2和R3信号都相对比较弱的客厅做了停留，所以导致了丢包，但是iperf很快便又有了速度，应该是dawn配置还需要调优，加上我的R1还没有开启WIFI。

通过编辑无线配置文件来开启syslog，也可以观察FT的工作情况。在LUCI的【状态】-【系统日志】中查看，也可以使用logread命令在shell中查看。

nano /etc/config/wireless
#config wifi-device 'radio1'
#       option type 'mac80211'
#      option path '1e140000.pcie/pci0000:00/0000:00:00.0/0000:01:00.0'
#      option channel '157'
#      option band '5g'
#      option htmode 'VHT80'
#      option country 'CN'
#      option cell_density '0'
#      option txpower '19'
      option log_level '1'  #启用日志
#日志中出现下列字段 表示FT工作正常
#IEEE 802.11: authentication OK (FT)
#WPA: FT authentication already completed - do not start 4-way handshake
Sun Jun  2 15:46:37 2024 daemon.debug hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed IEEE 802.11: binding station to interface 'phy1-ap0'
Sun Jun  2 15:46:37 2024 daemon.debug hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed IEEE 802.11: authentication OK (FT)
Sun Jun  2 15:46:37 2024 daemon.debug hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed MLME: MLME-AUTHENTICATE.indication(aa:54:fc:4b:fc:ed, FT)
Sun Jun  2 15:46:37 2024 daemon.debug hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed IEEE 802.11: association OK (aid 1)
Sun Jun  2 15:46:37 2024 daemon.info hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed IEEE 802.11: associated (aid 1)
Sun Jun  2 15:46:37 2024 daemon.notice hostapd: phy1-ap0: AP-STA-CONNECTED aa:54:fc:4b:fc:ed auth_alg=ft
Sun Jun  2 15:46:37 2024 daemon.debug hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed MLME: MLME-REASSOCIATE.indication(aa:54:fc:4b:fc:ed)
Sun Jun  2 15:46:37 2024 daemon.debug hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed IEEE 802.11: binding station to interface 'phy1-ap0'
Sun Jun  2 15:46:37 2024 daemon.debug hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed WPA: event 6 notification
Sun Jun  2 15:46:37 2024 daemon.debug hostapd: phy1-ap0: STA aa:54:fc:4b:fc:ed WPA: FT authentication already completed - do not start 4-way handshake
DAWN配置

在测试中，就目前的效果就已经可以满足我的日常使用了，所以不再对dawn进行精细调整，如果需要调整可以在https://github.com/berlin-open-wireless-lab/DAWN/blob/master/CONFIGURE.md找到详细的配置方法，简单询问了GPT他给了我这些建议：

channel_utilization：启用信道利用率检测，以帮助选择最佳的接入点。

rssi_val：设置RSSI阈值，低于此值的客户端将被重新分配到更好的接入点。

chan_util_check 和 client_check：启用信道利用率和客户端检查。

确保所有AP的SSID和安全设置（例如加密类型和密码）一致，这样可以保证客户端可以在不同的AP之间无缝漫游。根据实际环境调整rssi_val（RSSI阈值），确保客户端可以及时切换到信号更强的AP。设置DAWN的日志级别和监控机制，以便及时发现和解决问题。可以将log_level设置为更高的值（如 2 或 3）以获得更多调试信息。

发表回复

您的电子邮箱地址不会被公开。 必填项已用*标注

在浏览器中保存我的名字、电子邮件和网站地址，以便下次发表评论时使用。

搜索
搜索
近期文章
记录一次折腾WIFI漫游的过程
记一次闭门造车的迭代开发
PVE离线备份虚拟机
ChatGPT无法访问的解决办法
部署kubernetes集群并添加Dashboard插件
文章分类
C++
chatGPT
kubernetes
OpenWRT
v2dd
WordPress
国产化
学习研究
安装使用-openwrt
安装使用-wordpress
未分类
系统架构师
网络安全基础
虚拟化
近期评论
drx发表在部署kubernetes集群并添加Dashboard插件
猪猫发表在部署kubernetes集群并添加Dashboard插件
匿名发表在ESXI注入驱动网卡驱动，直通AMD显卡输出
上一篇
在OpenWRT上使用docker实现NextCloud
苏ICP备2022029188号-1
