Title: 家庭网络小升级 Part II / 双 AP 组网实战：我的 IoT 设备怎么又双叒叕炸了

URL Source: https://mary.kevinmx.top/default/80211kvr.html

Markdown Content:
这篇文章创建于 2023-10-25 日，距今已有 494 天，请注意甄别内容是否已经过时！

### 0\. 前情提要 ~ simple introduction.

如 [前文](https://mary.kevinmx.top/default/immortalwrt-iptv.html) 所说，家里目前有两台路由器：

![Image 1: APs](https://mary.kevinmx.top/usr/uploads/2023/10/1803567887.png)

通过 802.11 k/v/r 无线漫游（以下简称 802.11kvr / kvr），可以极大的改善信号质量和网速，即使 K2P 在 5G 下只是能勉强摸到 500Mbps+ 的速度。有总比没有强。

### 1\. 什么是 802.11 kvr ~ What is it?

引用一下 Wikipedia 的描述：

*   802.11k: Provides information to discover the best available access point.
*   802.11v: Allows client devices to exchange information about the network topology, including information about the RF environment, making each client network-aware, facilitating overall improvement of the wireless network.
*   802.11r: Permit continuous connectivity aboard wireless devices in motion, with fast and secure client transitions from one Basic Service Set (abbreviated BSS, and also known as a base station or more colloquially, an access point) to another performed in a nearly seamless manner.

说人话就是：

*   802.11k 能让设备（Client） 自行搜索附近最佳接入点（AP）。
*   802.11v 允许设备之间交换信息，了解整个网络的状态，协助客户端（Client）漫游。
*   802.11r 在客户端（Client）切换接入点（AP）的时候更快进行认证，实现近乎无缝衔接的漫游。

kvr 只是自动切换，跟 Mesh 实际上是两码事，Mesh 是 802.11s，在此不做介绍。只是有些 AP 在开启 Mesh 的时候会一并开启 k/v/r 而已。一般来说，支持 802.11kv 就可以实现无线漫游了。如果能开 802.11r，那么漫游体验会更好，降低切换时的延迟。当然前提是设备得支持。如标题所说，大家都不希望自己的设备在 AP 开启 kvr 之后连不上网，对吧。

### 2\. 如何配置 802.11kvr ~ How to do it?

互联网上关于 OpenWrt 配置 802.11kvr 的教程已经不少了，我也参考了相当一部分。在主线 OpenWrt 上操作是相对简单的，你只需要：

*   刷入支持无线漫游的固件
*   如果是 OpenWrt / ImmortalWrt，确保已安装支持 802.11kvr 的 wpad & hostapd
*   路由器无线驱动必须支持 802.11kvr
*   AP 的防火墙、DHCP 等已配置好
*   必要的情况下，适当降低 AP 发射功率
*   路由器厂商/品牌不重要，但必须用相同类型的固件，比如 MT7981 你用 OpenWrt/ImmortalWrt 这种主线的就都用主线开源驱动的，用 MTK SDK 就用 MTK SDK 的，不要混用，会爆炸的

我的两台 AP 分别是 MT7981 的 CMCC RAX3000m 和 MT7621/7615 的斐讯 K2P，均已刷入 ImmortalWrt 23.05-rc4 固件。（没刷 23.05 release 是因为还没发，ImmortalWrt 维护者表示变更不大，为了节省空间，暂时没必要）

两台设备的开源驱动均支持 kvr，并且固件预装了支持 802.11kvr 的 hostapd 和 wpad，无须手动操作，仅需要手动开启即可。

首先，登录路由器后台，进入 `网络 - 无线`，对 2.4GHz 和 5GHz 频段的 WiFi 进行编辑。这里以 5GHz 为例。后面会对一些配置进行解释。

2.4GHz 和 5GHz 保持一样的配置即可，如下：

无线安全：

*   加密：WPA-PSK/WPA2-PSK Mixed Mode（或者 WPA2-PSK，**绝对不要选 WPA3-SAE 或 WPA2-PSK/WPA3-SAE Mixed Mode**）
*   802.11w 管理帧保护：默认即可，禁用/可选

WLAN 漫游：

*   802.11r 快速切换：勾选
*   NAS ID：留空
*   移动域：任意四位十六进制数字，默认是按照你 SSID 算 sha256sum 取了四位，**所有 AP 上此数值必须一致**
*   重关联截止时间：20000
*   FT 协议：默认（FT over the Air）
*   本地生成 PMK：**取消勾选**
*   802.11k RRM：勾选
*   邻居报告：勾选
*   信标报告：勾选
*   时间公告：启用
*   时间区域：CST-8（默认有什么就选什么）
*   WMN 睡眠模式：勾选
*   WMN 睡眠模式修复：**不要勾选**
*   BSS 过渡：勾选
*   代理 ARP：**不要勾选**

其他配置保持默认即可。

![Image 2: wlansec](https://mary.kevinmx.top/usr/uploads/2023/10/3801157341.png)  
![Image 3: roaming](https://mary.kevinmx.top/usr/uploads/2023/10/1780094929.png)

配置至此告一段落。可以拿着手机开个 WiFiMan / WiFi 魔盒 / WiFi 评测大师 / TP-Link 的 WiFi 百宝箱 什么的看看漫游能不能用了。如果不太工作，比如信号已经非常弱了还是不切 AP，可以降低发射功率看看。如果 WiFi 都断了也死都不切 AP，那么可能需要修改配置。或者换个 AP。

### 3\. 为什么你是这么配的？跟别人的不太一样啊？~ Why the f\*\*\*?

如标题所说……**我的 IoT 设备怎么又\*\*的炸了……**

我在网上看到的其他教程，有建议开启 WPA3 的，有全勾选的，有说关闭 802.11r 的。众说纷纭，自己实测下来，现在这套配置最为稳定，2.4GHz 和 5GHz 都能连上，老手机和 IoT 设备也都正常。

已知会导致奇奇怪怪的兼容性问题的选项：

*   中文 SSID：这个应该不用我说了。最好别用，有些老设备就是连不上中文 SSID。
*   WPA3：可能导致部分（天玑 / MTK Dimensity / IoT）设备无法连接至 AP，2.4/5GHz 都一样。  
    *   有人说这玩意和 802.11r 冲突，然而我这边实测是即使关闭 11r 也连不上。
*   WMN 睡眠模式修复：会导致部分 2.4GHz IoT 设备无法连接至 AP。
*   代理 ARP：会导致（另一）部分 2.4GHz IoT 设备无法连接至 AP。

简单测试一下：

![Image 4: done](https://mary.kevinmx.top/usr/uploads/2023/10/2502117919.png)

路由器后台会出现类似这样的日志：

```
daemon.notice hostapd: phy1-ap0: AP-STA-CONNECTED ........... auth_alg=**ft**
```

看到出现 `ft` 字样，说明漫游已经生效了。

完事，收工。

### 4\. 致谢 / 参考 ~ Credits.

1.  [ImmortalWrt](https://immortalwrt.org/)
2.  [如何正确配置 OpenWrt WPA3 + 802.11kvr](https://vicfree.com/2022/11/openwrt-wpa3-802.11kvr-ap-setup/)
3.  [IEEE 802.11k-2008 - Wikipedia](https://en.wikipedia.org/wiki/IEEE_802.11k-2008)
4.  [IEEE 802.11v-2011](https://en.wikipedia.org/wiki/IEEE_802.11v-2011)
5.  [IEEE 802.11r-2008](https://en.wikipedia.org/wiki/IEEE_802.11r-2008)
