Title: 我的家庭网络设计思路，开启debian的旁路由之路（一）

URL Source: https://evine.win/p/%E6%88%91%E7%9A%84%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF%E5%BC%80%E5%90%AFdebian%E7%9A%84%E6%97%81%E8%B7%AF%E7%94%B1%E4%B9%8B%E8%B7%AF%E4%B8%80/

Published Time: 2023-10-19T00:00:00+08:00

Markdown Content:
[![Image 1: Featured image of post 我的家庭网络设计思路，开启debian的旁路由之路（一）](https://evine.win/p/%E6%88%91%E7%9A%84%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF%E5%BC%80%E5%90%AFdebian%E7%9A%84%E6%97%81%E8%B7%AF%E7%94%B1%E4%B9%8B%E8%B7%AF%E4%B8%80/%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C_hu_bffe148f502a1d7b.jpg)](https://evine.win/p/%E6%88%91%E7%9A%84%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF%E5%BC%80%E5%90%AFdebian%E7%9A%84%E6%97%81%E8%B7%AF%E7%94%B1%E4%B9%8B%E8%B7%AF%E4%B8%80/)

[软路由](https://evine.win/categories/gateway/)

### 本篇主要讲解子网划分及网络拓扑设计，以及为了容纳旁路由的存在，而需要在爱快上设置的一些内容。

前言
--

之前我用的是爱快作为主路由，OpenWRT作为旁路由，旁路由上会跑一些只有它能跑而主路由上不能跑的服务。尽管OpenWRT还是我自己定制化编译的，抛弃了许多我不使用的功能，但可能因为要用的服务仍然有些多，并且它们之间都会操作netfilter，导致网络经常出问题。所以，用OpenWRT作为旁路由的时候，我从来不敢在爱快上直接指定全部设备的网关和DNS为OpenWRT（网关和DNS仍然是爱快自己），只敢在我自己的设备上手动指定网关和DNS为OpenWRT。但这长久用起来不是个办法，所以干脆下定决心，抛弃OpenWRT，完全启用Debian来作为旁路由。

网络设计思路
------

为什么不直接采用Debian作为主路由呢，毕竟爱快作为专业的路由系统，还是有其便利性的，查看相关数据也比较容易，如果直接采用Debian作为主路由的话，将会有大量的配置工作，并且想要便捷的查看各项数据也不是很方便。

**并且，我还需要旁路由的存在不能影响端口转发，必须由主路由爱快直接将端口转发到NAS及其他服务，不能先转发到旁路由再经旁路由再转发一次。**

那么，基于爱快作为主路由，Debian作为旁路由，我将以下面的拓扑图开展。

![Image 2: 家庭网络](https://evine.win/p/%E6%88%91%E7%9A%84%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF%E5%BC%80%E5%90%AFdebian%E7%9A%84%E6%97%81%E8%B7%AF%E7%94%B1%E4%B9%8B%E8%B7%AF%E4%B8%80/%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C.jpg)

上图中仅涉及了爱快中`lan1`的设置，如果需要使用IPTV，还可以参考[我的这篇文章](https://evine.win/p/ikuai-how-use-iptv-in-shanghai)来设置一下`lan2`。

关于IPv6
------

IPv6是未来的趋势，我的ISP也已经支持IPv6了，并且不管是DDNS还是PT，有IPv6的双栈总归是好的。但又同时，我所使用的科学上网服务并不支持IPv6，所以，我需要在`10.0.0.0/24`这个科学上网的网段保留国内网站的IPv6解析，屏蔽掉国外网站IPv6的解析。这就得靠我在旁路由`10.0.0.2`上自建DNS服务了，与此同时，如果不关闭爱快IPv6的DNS服务，由于我并没有设置旁路由作为IPv6网关，仍然使用主路由作为IPv6网关，这样的话，终端就不仅仅从我的自建DNS服务来获取解析了，也会从IPv6 DNS服务器来获取，所以还需要关闭爱快IPv6的DNS。自建DNS服务一样可以解析IPv6地址，以IPv6访问国内网站不是问题。

说明
--

### 网段的划分

*   `10.0.0.0/24`作为科学上网的子网，该子网下全部设备默认直接可以科学上网。注意其中旁路由的IP是`10.0.0.2/24`
    
*   `10.0.1.0/24`作为无需科学上网的子网，该子网下全部设备默认不可以科学上网。
    

### 爱快中的设置

1.  `lan1`设置`10.0.0.1`为主IP，子网掩码为`/24`，同时设置`10.0.1.1`为扩展IP，子网掩码同样为`/24`，详见下图。你也可以设置更多的扩展IP来分别管理不同的网段及设备（比如再单独设置一个子网为访客网络）。

![Image 3: 爱快lan1设置](https://evine.win/p/%E6%88%91%E7%9A%84%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF%E5%BC%80%E5%90%AFdebian%E7%9A%84%E6%97%81%E8%B7%AF%E7%94%B1%E4%B9%8B%E8%B7%AF%E4%B8%80/%E7%88%B1%E5%BF%ABlan1%E8%AE%BE%E7%BD%AE.png)

2.  爱快的DHCP服务器中分别设置两个子网的掩码、网关和DNS，其中`10.0.0.3-10.0.0.254`的网关和DNS都是旁路由的IP`10.0.0.2`(请在完成全部旁路由配置以后再在爱快中设置成这样，没配置好旁路由前就设置成这样会上不了网)；`10.0.1.2-10.0.1.254`这一段的网关是爱快的扩展IP`10.0.1.1`，DNS可以设置为当地响应最快的DNS服务器，如果爱快开启了DNS加速，也可以将爱快的扩展IP`10.0.1.1`设置为这一网段的DNS服务器。

![Image 4: 爱快DHCP设置](https://evine.win/p/%E6%88%91%E7%9A%84%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF%E5%BC%80%E5%90%AFdebian%E7%9A%84%E6%97%81%E8%B7%AF%E7%94%B1%E4%B9%8B%E8%B7%AF%E4%B8%80/%E7%88%B1%E5%BF%ABDHCP%E8%AE%BE%E7%BD%AE.png)

3.  为全部设备设置DHCP静态分配。如果你需要为后续首次接入网络的设备分配到`10.0.1.2-10.0.1.254`中，而不是直接分配到`10.0.0.3-10.0.0.254`网段中，你可以在DHCP静态分配中，把那些还没有设备占用的IP用假MAC先占用着，MAC批量生成工具[在此](https://uutool.cn/mac/)，批量生成后粘贴在从爱快DHCP静态分配导出的csv文件中，再导入回去就好。或者也可以交换一下`10.0.0.0/24`作为无需科学上网的子网，`10.0.1.0/24`作为需要科学上网的子网。
    
4.  关闭IPv6的DNS。后续我将在旁路由上自建DNS服务，所以必须关闭爱快IPv6的DNS通告。
    

![Image 5: 爱快IPv6设置](https://evine.win/p/%E6%88%91%E7%9A%84%E5%AE%B6%E5%BA%AD%E7%BD%91%E7%BB%9C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF%E5%BC%80%E5%90%AFdebian%E7%9A%84%E6%97%81%E8%B7%AF%E7%94%B1%E4%B9%8B%E8%B7%AF%E4%B8%80/%E7%88%B1%E5%BF%ABIPv6%E8%AE%BE%E7%BD%AE.png)

5.  全部子网内的设备不再单独设置防火墙，全部在主路由爱快中设置，具体的教程可以见这两篇文章：[IPv4](https://evine.win/p/ikuai-set-ipv4-acl)、[IPv6](https://evine.win/p/ikuai-set-ipv6-acl-2)。
    
6.  网关为旁路由的设备在爱快的终端监控中，全部显示为旁路由的信息，所以我们还需要设置爱快的“高级应用-\>跨三层应用”，具体如何设置因为还涉及到旁路由中的设置，我将在接下来的 [本系列第（八）篇](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e5%85%ab) 中详细讲解。
    

### 交换机

如果主路由有足够的网口，交换机可以省略，全部设备都直接接入主路由。有交换机时，即使交换机本身的管理IP处于`10.0.0.3/24`，但接入的处于`10.0.1.0/24`网段的设备一样是可以正常上网的，毕竟它是交换机。

### AP

如果主路由或者旁路由有无线功能，AP也可以省略，直接以主路由或者旁路由作为AP即可。

### 旁路由

旁路由是不是虚拟机都没有关系。我的主路由是物理机，旁路由是虚拟机。其实我的很多设备都是通过虚拟机或者Docker的MacVLAN接入网络中，它们既有`10.0.0.0/24`的，也有`10.0.1.0/24`的，都按照我上面设置的DHCP服务正常工作着。

总结
--

通过分离为两个子网，将需要科学上网的设备和不需要科学上网的设置分开，同时在前者所在子网下自建DNS服务器，达到国内支持IPv4/IPv6双栈，国外仅通过IPv4科学上网的目的。而在不需要科学上网的子网下，设备都是直接采用公共DNS服务器。

不同的服务可以放在不同的子网下分别进行管理，比如我的许多Docker容器就通过创建在两个子网下的MacVLAN网络分别走不同的路线，一块网卡也可以创建多个MacVLAN网络以进行分流，详见我的 [这篇文章](https://evine.win/p/docker-create-macvlan-with-multi-subnet)，比如 [我的qBittorrent容器](https://evine.win/p/docker-install-qbittorrent) 就放在`10.0.1.0/24`下，完全不用担心流量走了代理（我甚至在Clash中开启了代理PT网站，但这只影响我用PC访问PT网站，对 [我的qBittorrent容器](https://evine.win/p/docker-install-qbittorrent) 毫无影响）。至于虚拟机也是一样，可以将其放入不同的子网来走不同的路线。

接下来，我们将继续设置Debian为旁路由，真正的开启Debian的旁路由之路，这将在我的 [软路由系列文章](https://evine.win/categories/gateway/) 中逐渐地一篇一篇发出来。这个系列虽然比较长，看起来比较复杂，但你可以根据你的需要进行选择性的使用，并且，配置好后是一劳永逸的，不用担心刷机，不用担心哪天服务故障，这套配置比OpenWRT稳定多了，即使未来升级系统也是分分钟的事。

系列
--

*   [一、子网划分及网络拓扑设计、爱快上的设置](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e4%b8%80)
    
*   [二、安装Debian、初始网络配置、必要软件安装](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e4%ba%8c)
    
*   [三、AdGuardHome、mosdns、clash在DNS的关系，以及AdGuardHome、mosdns的安装和配置](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e4%b8%89)
    
*   [四、clash透明代理(Tproxy)的安装和配置、相关的IP规则和nftables规则](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e5%9b%9b)
    
*   [五、UnblockNeteaseMusic的安装和配置、相关的的nftables规则、客户端如何使用](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e4%ba%94)
    
*   [六、subconverter的安装和配置](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e5%85%ad)
    
*   [七、配置nginx为文件服务器、自动更新clash订阅、自动生成需要的文件](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e4%b8%83)
    
*   [八、配置旁路由的SNMPD服务、配置主路由的跨三层应用](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e5%85%ab)
    
*   [九、监测陌生设备](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e4%b9%9d)
    
*   [十、快速简单的安装和更新软件包](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e5%8d%81)
    
*   [十一、DNS、网络故障等的总结，以及一些注意事项](https://evine.win/p/%e6%88%91%e7%9a%84%e5%ae%b6%e5%ba%ad%e7%bd%91%e7%bb%9c%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af%e5%bc%80%e5%90%afdebian%e7%9a%84%e6%97%81%e8%b7%af%e7%94%b1%e4%b9%8b%e8%b7%af%e5%8d%81%e4%b8%80)
