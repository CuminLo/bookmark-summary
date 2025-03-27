Title: 京东云 AX1800 Pro 路由器(亚瑟)刷入 u-boot 和 OpenWrt

URL Source: https://phyng.com/2024/05/05/ax1800-openwrt.html

Markdown Content:
设备说明
----

*   2024-05 购于京东，512M 内存/128G 存储版本
*   合格证显示为 2022-03 检验合格，说明应该是 2022 年 3 月生产的库存
*   初始化之后管理界面底部显示为 1.2.2.r2080 - 1.3.0.r2106 版本，属于比较老的版本，可以免拆机开启 SSH
*   此教程只针对 128G 版本且 ROM 为 1.2.2.r2080 - 1.3.0.r2106 的路由器测试验证通过，其他版本可能需要自行测试
*   刷机有风险，操作需谨慎，刷机后可能导致路由器无法启动，需要自行承担风险

开启 SSH
------

拿到手之后先不要插 WAN 口网线，避免系统自动升级。开机按照默认配置进入管理界面之后，在路由设置/系统/升级检测路径关闭自动升级，确认底部版本显示为 1.2.2.r2080 - 1.3.0.r2106。然后使用 Chrome 浏览器打开路由器管理后台 http://192.168.68.1/ 按 F12 打开控制台，参考截图输入下面的代码，通过调用接口的方式开启 SSH。

如果是 2023 年 5 月份左右出厂的 r2242 版本可以参考 [https://www.bilibili.com/read/cv34442695/](https://www.bilibili.com/read/cv34442695/) 这个教程开启 SSH。

```
// 作者：cdlif https://www.bilibili.com/read/cv15885352/ 出处：bilibili

$.ajax({
url:'http://'+$.cookie('HostAddrIP')+'/jdcapi',
async:false,
data:JSON.stringify({
jsonrpc:'2.0',
id:1,
method:'call',
params:[
$.cookie('sessionid'),
'service',
'set',
{
name:'dropbear',
instances:{instance1:{command:['/usr/sbin/dropbear']}},
},
],
}),
dataType:'json',
type:'POST',
})
```

![Image 1: picture 1](https://oss.phyng.com/imgproxy/unsigned/rs:fit:1500:1500/plain/public/md/2024-05-05/e75b1551.png@webp)

开通之后可以用通过终端连接 SSH，密码就是路由器的管理密码。macOS/Linux 以及 Windows 11 用系统自带的终端工具即可。SSH 密码和路由器管理密码一致。

```
ssh-oHostKeyAlgorithms=+ssh-rsa-lroot192.168.68.1
```

下面是登录成功之后的界面

```
➜  ~ ssh -oHostKeyAlgorithms=+ssh-rsa -l root 192.168.68.1
The authenticity of host '192.168.68.1 (192.168.68.1)' can't be established.
RSA key fingerprint is SHA256:*.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.68.1' (RSA) to the list of known hosts.
root@192.168.68.1's password:


BusyBox v1.30.1 () built-in shell (ash)

 --------------------------------------------------------------------------
   Welcome to JDBox Router
 --------------------------------------------------------------------------

   $$$$$\ $$$$$$$\         $$$$$$\  $$\       $$$$$$\  $$\   $$\ $$$$$$$\
   \__$$ |$$  __$$\       $$  __$$\ $$ |     $$  __$$\ $$ |  $$ |$$  __$$\
      $$ |$$ |  $$ |      $$ /  \__|$$ |     $$ /  $$ |$$ |  $$ |$$ |  $$ |
      $$ |$$ |  $$ |      $$ |      $$ |     $$ |  $$ |$$ |  $$ |$$ |  $$ |
$$\   $$ |$$ |  $$ |      $$ |      $$ |     $$ |  $$ |$$ |  $$ |$$ |  $$ |
$$ |  $$ |$$ |  $$ |      $$ |  $$\ $$ |     $$ |  $$ |$$ |  $$ |$$ |  $$ |
\$$$$$$  |$$$$$$$  |      \$$$$$$  |$$$$$$$$\ $$$$$$  |\$$$$$$  |$$$$$$$  |
 \______/ \_______/        \______/ \________|\______/  \______/ \_______/

 --------------------------------------------------------------------------
   For those about to rock... (1.2.2.r2080, 14433b06a+r49254)
 --------------------------------------------------------------------------
```

登录 SSH 之后就可以执行下面的操作刷入 u-boot 和 OpenWrt 了。

刷入 u-boot
---------

根据项目 [https://github.com/0x5826/uboot-ipq60xx-build](https://github.com/0x5826/uboot-ipq60xx-build) 编译得到 u-boot.mbn 文件，也可以用下面我编译好的文件，下面是需要执行的命令和说明，后面有执行记录。

```
# 我是上传到了阿里云的 oss 上，通过 curl 下载，也可以通过其他方式比如 WinSCP 上传到路由器里面
# 如果是通过 curl 下载，此时路由器需要插上 WAN 口连上互联网，可以给 WAN 口接上网线
curl-ou-boot.mbnhttp://oss-hk4.oss-cn-hongkong.aliyuncs.com/tmp/u-boot.mbn
# 确认文件存在且大小正确，我编译的文件大小是 644624
ls-l/root/u-boot.mbn
```

```
# 刷入 u-boot，此步骤需要谨慎，刷错可能导致路由器无法启动，需要确保文件存在且完整
# 如果文件不完整这一步可能会导致路由器无法启动，所以一定要确认文件大小和完整性
ddif=/root/u-boot.mbnof=/dev/mmcblk0p13
ddif=/root/u-boot.mbnof=/dev/mmcblk0p14
```

下面是执行记录

```
root@JDBoxV2:~#curl-ou-boot.mbnhttp://oss-hk4.oss-cn-hongkong.aliyuncs.com/tmp/u-boot.mbn
%Total%Received%XferdAverageSpeedTimeTimeTimeCurrent
DloadUploadTotalSpentLeftSpeed
100629k100629k00245k00:00:020:00:02--:--:--260k

root@JDBoxV2:~#ls-l/root/u-boot.mbn
-rw-r--r--1rootroot644624Dec911:49/root/u-boot.mbn

root@JDBoxV2:~#ddif=/root/u-boot.mbnof=/dev/mmcblk0p13
1259+1recordsin
1259+1recordsout
644624bytes(629.5KB)copied,0.051704seconds,11.9MB/s

root@JDBoxV2:~#ddif=/root/u-boot.mbnof=/dev/mmcblk0p14
1259+1recordsin
1259+1recordsout
644624bytes(629.5KB)copied,0.050960seconds,12.1MB/s

root@JDBoxV2:~#
```

刷入 u-boot 成功之后可以按照下面的步骤进入 u-boot 界面，然后刷入 OpenWrt 固件。

*   直接拔掉路由器的电源和全部网线
*   在不插电的情况下先用牙签或者类似东西按着背后的 reset 按钮，保持按住不动然后插上电源，保持 reset 按钮一直按着，可以观察到路由器启动从红灯闪烁变成持续蓝灯，等待出现稳定不变的蓝灯之后再保持 30s 左右松开 reset。如果是正常系统启动路由器的话没有网线会持续红灯，而刷入 u-boot 之后即使没有网线也会变成蓝灯
*   插上网线，可以只连接路由器的 LAN 口（3 个黑色网口中的任意一个）和电脑，浏览器打开 192.168.1.1 进入 u-boot 界面，如果打不开可能需要手动设置电脑的 IP 地址，以 Windows 为例，可以打开电脑的控制面板 —\> 网络和共享中心 —\> 以太网 —\> 属性 —\> lnternet 协议版本 4 (TCP/IPv4)，设置 IP 为 192.168.1.10 网关为 192.168.1.1 掩码为 255.255.255.0，更多细节参考[此文档](https://docs.gl-inet.cn/router/4/features/uboot/#5-ip)，macOS 参考下面的图设置。如果浏览器存在之前的 Cookies 导致无法打开 192.168.1.1，可以使用 Chrome 的隐身模式新开一个窗口

![Image 2: picture 1](https://oss.phyng.com/imgproxy/unsigned/rs:fit:1500:1500/plain/public/md/2024-05-12/3d5c02dd.png@webp)

打开 192.168.1.1 看到下面的界面说明 u-boot 刷入成功

![Image 3: picture 2](https://oss.phyng.com/imgproxy/unsigned/rs:fit:1500:1500/plain/public/md/2024-05-05/28d22137.png@webp)

然后上传固件刷入即可，u-boot 上传固件之后会一直显示进度条，不用理会，等待几分钟之后路由器指示灯稳定之后可以访问对应管理界面说明固件刷入成功。推荐使用网线连接，因为有些固件默认关闭了无线网络。

![Image 4: picture 2](https://oss.phyng.com/imgproxy/unsigned/rs:fit:1500:1500/plain/public/md/2024-05-12/555fafe6.png@webp)

刷入 OpenWrt 固件
-------------

*   一般来说在 u-boot 界面要刷入 `squashfs-factory` 格式的固件，可以使用[这个 iStoreOS 固件](https://static.phyng.com/blog/jdc-ax1800/istoreos-squashfs-factory.bin)，文件大小为 62980100，sha256sum: d59908cebc826b6cd0f6edc2c98572d574411f516c199c6eba1e5aa17819f20f，对应管理界面为 192.168.1.1 用户 root 密码 password
*   刷入成功之后，进入 OpenWrt 升级界面可以刷入 其他 OpenWrt 的 `squashfs-sysupgrade` 格式的固件
*   部分固件可以通过内部的升级界面直接升级或者刷入其他固件，也可能不兼容，这时候可以通过 u-boot 界面刷入 `squashfs-factory` 格式的固件，进入 u-boot 的方式和上面文档一致
*   下面是一些已知的固件，下面这些固件都是使用默认分区，只能支持大小 60M 的固件，不需要改成大分区就能使用
    *   iStoreOS [https://fw.koolcenter.com/Lean/JDC\_AX1800\_Pro/](https://fw.koolcenter.com/Lean/JDC_AX1800_Pro/) （仅 istoreos-ipq60xx-23.09.09 为默认分区版/内核版本 4.4 比较低）
    *   [openwrt.ai](http://openwrt.ai/) [https://openwrt.ai/](https://openwrt.ai/) （打开之后选择设备 JD Cloud JDC AX1800 Pro）
    *   koksaver [https://github.com/koksaver/OpenWrt360\_6.1/](https://github.com/koksaver/OpenWrt360_6.1/) (内核版本 6.1/6.6)
    *   恩山论坛 [https://www.right.com.cn/forum/thread-8366456-1-1.html](https://www.right.com.cn/forum/thread-8366456-1-1.html)

尝试了一圈固件之后最后参考 [koksaver/OpenWrt360\_6.1](https://github.com/koksaver/OpenWrt360_6.1/) 项目编译了一个版本 [phyng/OpenWrt](https://github.com/phyng/OpenWrt) ，6.6 内核、支持安装 OpenClash、自带 WireGuard、支持 NSS 加速、无线速度稳定，下面是 `iperf3` 的无线测试结果。

```
➜  iperf3 -c 192.168.1.215
Connecting to host 192.168.1.215, port 5201
[  5] local 192.168.1.25 port 56718 connected to 192.168.1.215 port 5201
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  94.0 MBytes   786 Mbits/sec
[  5]   1.00-2.00   sec  98.8 MBytes   827 Mbits/sec
[  5]   2.00-3.01   sec  96.0 MBytes   805 Mbits/sec
[  5]   3.01-4.01   sec  98.5 MBytes   826 Mbits/sec
[  5]   4.01-5.00   sec  93.8 MBytes   788 Mbits/sec
[  5]   5.00-6.01   sec   100 MBytes   837 Mbits/sec
[  5]   6.01-7.01   sec  97.2 MBytes   816 Mbits/sec
[  5]   7.01-8.01   sec   100 MBytes   840 Mbits/sec
[  5]   8.01-9.01   sec  95.6 MBytes   802 Mbits/sec
[  5]   9.01-10.01  sec  98.5 MBytes   826 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-10.01  sec   972 MBytes   815 Mbits/sec                  sender
[  5]   0.00-10.02  sec   972 MBytes   814 Mbits/sec                  receiver

iperf Done.
```

扩展 overlay 分区
-------------

默认 /overlay 分区非常小，在不改分区的情况下可以通过挂载大分区的方式扩展 overlay 分区

```
[root@OpenWrt:10:13PM~]# df -h
FilesystemSizeUsedAvailableUse%Mountedon
/dev/root33.3M33.3M0100%/rom
tmpfs212.0M124.0K211.9M0%/tmp
/dev/loop023.4M463.0K21.1M2%/overlay
overlayfs:/overlay23.4M463.0K21.1M2%/
tmpfs512.0K0512.0K0%/dev
/dev/mmcblk0p27111.7G28.0K110.6G0%/mnt/mmcblk0p27
/dev/mmcblk0p2218.2M790.0K16.0M5%/mnt/mmcblk0p22
/dev/mmcblk0p2479.2M18.0K76.6M0%/mnt/mmcblk0p24
/dev/mmcblk0p25280.5M2.0K273.5M0%/mnt/mmcblk0p25
```

参考教程 [https://post.smzdm.com/p/a7p2opw5/](https://post.smzdm.com/p/a7p2opw5/) 可以将 /dev/mmcblk0p27 这个大分区挂载为 /overlay

其中 `mkfs.ext4 -F /dev/mmcblk0p27` 需要执行几分钟，需要耐心等待结束再执行下一步

```
# 格式化 mmcblk0p27 大分区
umount/dev/mmcblk0p27
mkfs.ext4-F/dev/mmcblk0p27
# 复制当前 overlay 文件到 mmcblk0p27
mkdir-p/mnt/mmcblk0p27
mount/dev/mmcblk0p27/mnt/mmcblk0p27
cp-r/overlay/*/mnt/mmcblk0p27
ls/mnt/mmcblk0p27
# 修改分区
blockdetect>/etc/config/fstab
sed-is#/mnt/mmcblk0p27#/overlay#/etc/config/fstab
sed-i'12s/1/0/g'/etc/config/fstab
# 重启路由器
reboot
```

执行完之后重启路由器。可以在软件包管理看到剩余可用的空间变大了。

![Image 5: picture 0](https://oss.phyng.com/imgproxy/unsigned/rs:fit:1500:1500/plain/public/md/2024-05-12/f6a6b92f.png@webp)

部分固件如果没有成功，可以参考下图在系统/挂载点管理那里删除之前的 /overlay 分区然后勾选挂载 mmcblk0p27 为 /overlay，然后重启路由器。

![Image 6: picture 0](https://oss.phyng.com/imgproxy/unsigned/rs:fit:1500:1500/plain/public/md/2024-06-08/6eb0db40.png@webp)

参考文档
----

*   [https://post.smzdm.com/p/al8ne66p/](https://post.smzdm.com/p/al8ne66p/)
*   [https://www.bilibili.com/read/cv15885352/](https://www.bilibili.com/read/cv15885352/)
*   [https://www.zhihu.com/tardis/zm/art/671227407](https://www.zhihu.com/tardis/zm/art/671227407)
*   [https://www.right.com.cn/forum/thread-8366456-1-1.html](https://www.right.com.cn/forum/thread-8366456-1-1.html)
*   [https://docs.gl-inet.cn/router/4/features/uboot/#5-ip](https://docs.gl-inet.cn/router/4/features/uboot/#5-ip)
