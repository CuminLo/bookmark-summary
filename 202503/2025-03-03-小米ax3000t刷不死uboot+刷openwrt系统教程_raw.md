Title: 小米AX3000T刷不死uboot+刷openwrt系统教程

URL Source: https://blog.wwang.pw/post/ax3000t

Published Time: 2023-12-12T02:21:16.000Z

Markdown Content:
文章
121
	
标签
108
	
分类
9
 主页
 文章
 分类
 归档
 标签
 仓库
 固件仓库
 固件仓库[分流站]
 资源库
 工具
 小米SSH密码计算
 GitHub文件加速
 缩短网址
 图床
 商店
 更多
 友链
 音乐
 关于
 单页
 倒计时
 俄罗斯方块
 时钟
 初音未来
 小圆点
 液体
 网址收藏
 时光搜索
 图片加水印
 动态白云
 图片自适应UI
 在线画板
 在线音乐
时光's Blog
 搜索 主页 文章 仓库 工具 商店 更多 单页
小米AX3000T刷不死uboot+刷openwrt系统教程
发表于2023-12-12|更新于2024-09-23|路由器
字数总计:784|阅读时长:3分钟|阅读量:
硬件规格
SoC: MediaTek MT7981B 2x A53
Flash: ESMT F50L1G41LB 128MB
RAM: NT52B128M16JR-FL 256MB
Ethernet: 4x 10/100/1000 Mbps
Switch: MediaTek MT7531AE
WiFi: MediaTek MT7976C
Button: Reset, Mesh
Power: DC 12V 1A

https://github.com/openwrt/openwrt/pull/12770

刷机文件

下载地址：

https://pan.wwang.pw/%F0%9F%94%A5OpenWrt_LEAN/%E5%B0%8F%E7%B1%B3AX3000T

如地址失效，加群获取即可。

开启SSH

代码中**stok=**后面的xxx换成你自己的

1
2
3
4
5
6
7

	
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=xxx/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20set%20ssh_en%3D1%0A"

curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=xxx/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20commit%0A"

curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=xxx/api/misystem/arn_switch -d "open=1&model=1&level=%0Ased%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%22debug%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%0A"

curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=xxx/api/misystem/arn_switch -d "open=1&model=1&level=%0A%2Fetc%2Finit.d%2Fdropbear%20start%0A"


WIN+R唤起命令提示符，依次输入以上命令，开启SSH，每条命令输入完成后出现{"code":0}才可以

开启SSH后需要获取root密码，去这里：https://wwang.pw/mi/ ，输入路由器SN获取密码

刷入uboot

打开MobaXterm，输入路由器管理地址192.168.31.1，用户名root，密码就是刚刚获取到的，输入密码不会有显示，输入完成后直接回车即可。

把mt7981_ax3000t-fip-fixed-parts-multi-layout.bin文件上传到tmp目录下，输入

1

	
mtd write /tmp/mt7981_ax3000t-fip-fixed-parts-multi-layout.bin FIP


刷OPENWRT
进入uboot WEB页面
修改电脑IP为192.168.1.*，网关为192.168.1.1

拔掉电源，按住reset键，蓝灯常亮即可进入BOOT，浏览器输入192.168.1.1，进入BOOT页面

刷入H大的闭源immortalwrt固件

此固件是基于hanwckf大佬的immortalwrt-mt798x项目，特点如下

基于immortalwrt 21.02分支，内核版本5.4，目前仅支持mt7981/mt7986
使用mtk-openwrt-feeds提供的有线驱动、hnat驱动、内核补丁及配置工具，支持所有加速特性
mt7981/mt7986均支持两个ppe，每个ppe最大支持32k连接数，默认开启1个ppe
使用mtwifi原厂无线驱动（目前默认使用7.6.6.1无线驱动，并提供7.6.7.0版本的二进制包），默认开启802.11k，支持warp在内的所有加速特性
使用luci-app-mtk作为无线配置界面，支持查看连接的客户端相关信息、支持ApCli中继扫描，不支持原生luci。目前原生luci无线界面已在开发中。

参考下图刷入immortalwrt-mediatek-mt7981-xiaomi_mi-router-ax3000t-squashfs-factory.bin文件

刷入LEAN固件

此固件是基于LEAN大佬的lede源码编译。

参考下图刷入openwrt-mediatek-filogic-xiaomi_mi-router-ax3000t-squashfs-sysupgrade.bin文件即可

完成

后台地址 192.168.5.1 密码 password

进不去后台的，插拔电源重启一下即可。

注意：靠近电源接口的网口是WAN口

购买路由器

小店已上架此款路由器，刷好固件再发货，不愿意折腾的可以下单。

购买链接：https://shop.wwang.pw/

文章作者: XiaoWan💦
文章链接: https://blog.wwang.pw/post/ax3000t.html
版权声明: 本博客所有文章除特别声明外，均采用 CC BY-NC-SA 4.0 许可协议。转载请注明来自 时光's Blog！
openwrt不死uboot小米ax3000t
上一篇
OpenWrt带你畅享AirPlay 2无损音乐
下一篇
红米ax6000刷不死uboot+刷openwrt+刷回原厂系统教程
相关推荐
 2024-12-01
京东云百里AX6000刷机教程
 2024-08-20
安博通ABT-ASR3000刷机指南
 2024-06-21
京东云无线宝亚瑟AX1800Pro低版本免拆刷机
 2023-12-01
红米ax6000刷不死uboot+刷openwrt+刷回原厂系统教程
 2024-09-20
诺基亚贝尔AX3000(EA0326GMP)刷机教程
 2024-08-04
OpenWrt下访问光猫教程
 评论
XiaoWan💦
文章
121
	
标签
108
	
分类
9
关注我
公告
如果你在使用中遇到问题，可以加QQ群或TG群进行反馈
QQ2群👍 QQ3群👍 TG群👍
那年今日
目录
1. 硬件规格
2. 刷机文件
3. 开启SSH
4. 刷入uboot
5. 刷OPENWRT
6. 完成
7. 购买路由器
最新文章
京东云百里AX6000刷机教程
2024-12-01
诺基亚贝尔AX3000(EA0326GMP)刷机教程
2024-09-20
安博通ABT-ASR3000刷机指南
2024-08-20
OpenWrt下访问光猫教程
2024-08-04
用 dd 命令快速更换 OpenWrt 固件
2024-07-30
©2025  XiaoWan💦
正在加载今日诗词....
 固件仓库 •  小商店 •  音乐库 •  TG群
繁
