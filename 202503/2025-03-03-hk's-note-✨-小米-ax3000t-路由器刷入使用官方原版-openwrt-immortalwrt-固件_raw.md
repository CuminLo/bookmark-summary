Title: HK's note ✨ 小米 AX3000T 路由器刷入使用官方原版 OpenWrt / ImmortalWrt 固件

URL Source: https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt

Markdown Content:
xiaomi-ax3000t-router-install-openwrt-immortalwrt

文章介绍了如何将小米 AX3000T 路由器刷入官方原版 OpenWrt 或 ImmortalWrt 固件，包括降级系统、获取 stok 值、开启 SSH、备份原始分区、刷入 initramfs system 以及最终升级固件的全过程。

### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#fd43da4c7c624a12befe89563a02ec9b "更新：")更新：

hanwckf 版本的 ImmortalWrt 和 U-Boot 已经支持 AN8855 版本的 AX3000T ，感谢 H 大的适配。

固件及 U-Boot ：

简要说明：

⚠️

自8月左右新版本的 AX3000T 设备（固件版本为 1.0.84 或更高版本），已换用新的硬件（AN8855 交换机），暂时与 OpenWrt 官方原版镜像不兼容，需等待适配。

若在此类设备上通过小米路由器修复工具降级系统版本，后续路由器的有线网口将表现为交换机工作模式，不能通过一般 Wan 口接入宽带网络。

~除非只使用原厂固件，否则建议及时退换货。~

* * *

**已有支持 1.0.84 版本的 OpenWrt 固件了，按照本文方法，但将固件替换为如下引用帖子中的即可（展开查看**

> 添加 Winbond W25N01KV 支持 添加 Foresee spinand 支持 添加 switch-an8855 驱动程序（由 @linaro 提供）

下载:：

原帖：

恩山论坛：

### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#9c58ae609ba54698b8ee07a8084a54d9 "前言")前言

小米路由器从刚开始推出时的争议吐槽不断到现在一路走过来隐隐成了性价比之选。在此前曾使用过小米路由器mini、红米AC2100、红米AX6(S)等热门型号机器，整体体验还是很不错的，

~尤其是在刷了 OpenWrt 之后~（hhh 😂） 。

小米 AX3000T 路由作为一款入门级的 Wi-Fi 6 路由，使用了发哥（联发科）的方案，正是继红米 AX6S 之后的又一款热门路由。但硬件虽好，软件并不能满足我的需求，下面是将其整成 OpenWrt 固件的记录。

展开 小米 AX3000T 基础配置信息

> CPU : 联发科 FiLogic 820系列的MT7981（12nm 双核A53架构，12nm工艺制程，1.3GHz主频）

> ROM : 128MB

> RAM : 256MB

> 网络 : 4个自适应千兆口，天线数量4根（其中一根为 5GHz 和 2.4GHz 共用）支持 Wi-Fi 6 160MHz频宽

### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#a9e56cacdde8484f9687f320229393d5 "过程记录")过程记录

整个刷入过程主要分为以下几步：

1.  降级系统至旧版本（版本号 1.0.47，该版本存在漏洞可开启 SSH）

2.  获取 stok 值，执行命令请求开启 SSH 访问权限

3.  登录 SSH ，**备份**原始分区

4.  刷入 OpenWrt initramfs system

5.  进入 OpenWrt initramfs system 升级 OpenWrt 正式固件

💡

开启 SSH 也可以看看这个针对小米路由器开源项目

#### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#0cc24c322eae4261975171309b5d1eb6 "安装降级固件")安装降级固件

⬇️ 下载降级固件此处需要用到小米路由器修复工具，仅支持 Windows。下载地址：

![Image 1: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2F139a11f6-5b88-4ebf-b3ba-f847270dd477%2Fimage.png?table=block&id=9fc4cea7-9aa9-4bf4-8ddd-fb015bc2e04a&cache=v2)

下载并运行此修复工具，选择上面👆下载的降级固件（版本 1.0.47），然后选中要使用的网卡，此处建议将其他网卡都先关闭以避免干扰，只保留物理网口对应的以太网网卡即可。

![Image 2: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2F69168900-461a-492f-bca0-7d3fcbf4925c%2F1000008428.jpg?table=block&id=d059f1ee-0a96-47cf-87b2-e572dfe4a5d2&cache=v2)

按修复工具的提示逐步操作，到如下界面时：断开路由器电源，用网线将路由器网口和电脑网口连接。然后用牙签或卡针顶住路由器上的 reset 按键，保持不松开。此时再插上电源，约莫8秒左右直到路由器指示灯开始快速闪烁，即可松开 reset 键。修复工具会接收到路由器的请求开始恢复固件。

![Image 3: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2Fdae3ce2a-4b09-454d-b754-a4b630c0dbd2%2F1000008430.jpg?table=block&id=39a2bcd5-cb73-453b-8294-be515d20327c&cache=v2)

稍等一会儿恢复完成后，即可重新给路由器上电，进入系统后查看已经降级成功，如下图所示。

![Image 4: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2F82833140-d400-4f7b-9683-c020618c5d30%2Fimage.png?table=block&id=b1287186-7f88-4caf-b8a9-aa5fb9166d1a&cache=v2)

#### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#cebe78e76f8e4ae18bec92387995da60 "获取 stok 值")获取 stok 值

登录路由器 Web 管理后台（默认为 [http://192.168.31.1](http://192.168.31.1/) ），获取 `stok` 值——下图中红框所标识的字符串。

![Image 5: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2F0a1d23ab-5d3e-4226-862e-89e0d9d9e935%2F0cb84116-2eb2-4c7e-b128-732b826c112f.png?table=block&id=7c0f96b8-6cf6-4daa-adfd-6a1cb33dd2e9&cache=v2)

~查询路由器初始密码~（后续命令会设置 root 密码，此处无需进行了）

#### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#30b156a782734dd8b32db9875763260b "启用 SSH 权限")启用 SSH 权限

打开命令行工具（Terminal）依次执行如下 `curl` 请求，其中需要将上文中获取到的 `stok` 值替换。（若是路由器 IP 和我一样有变化的，也需要将其中的 192.168.31.1 进行替换）

*   开启 SSH 配置

```
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=此处替换为上面记录的stok值/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20set%20ssh_en%3D1%0A"
```

*   提交配置

```
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=此处替换为上面记录的stok值/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20commit%0A"
```

*   设置 dropbear SSH 为 debug

```
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=此处替换为上面记录的stok值/api/misystem/arn_switch -d "open=1&model=1&level=%0Ased%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%22debug%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%0A"
```

*   启动 dropbear SSH

```
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=此处替换为上面记录的stok值/api/misystem/arn_switch -d "open=1&model=1&level=%0A%2Fetc%2Finit.d%2Fdropbear%20start%0A"
```

*   设置 root 密码为空

```
curl -X POST "http://192.168.31.1/cgi-bin/luci/;stok=此处替换为上面记录的stok值/api/misystem/arn_switch" -d "open=1&model=1&level=%0Apasswd%20-d%20root%0A"
```

以上命令执行成功，则会响应` {“code”: 0}` ，如为其他则请检查 `stok` 或 IP 地址是否正确替换了

![Image 6: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2F7c1bad32-baac-4994-b4bf-25ba3a620599%2Fimage.png?table=block&id=b6eab70c-b621-434d-99eb-bf5d05ed305e&cache=v2)

👇 整合的 shell 脚本

```
#!/bin/bash

if [ "$1" = "" ]; then
  echo "Usage: $0 [stok]"
  echo "e.g. $0 84f8bce46164e9e0296ad38086cd8472"
  echo "登录Web后台后，从浏览器地址栏复制 stok 字符串"
  exit 1
fi

curl -X POST "http://192.168.31.1/cgi-bin/luci/;stok=${1}/api/misystem/arn_switch" -d "open=1&model=1&level=%0Anvram%20set%20ssh_en%3D1%0A"
sleep 1
curl -X POST "http://192.168.31.1/cgi-bin/luci/;stok=${1}/api/misystem/arn_switch" -d "open=1&model=1&level=%0Anvram%20commit%0A"
sleep 1
curl -X POST "http://192.168.31.1/cgi-bin/luci/;stok=${1}/api/misystem/arn_switch" -d "open=1&model=1&level=%0Ased%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%22debug%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%0A"
sleep 1
curl -X POST "http://192.168.31.1/cgi-bin/luci/;stok=${1}/api/misystem/arn_switch" -d "open=1&model=1&level=%0A%2Fetc%2Finit.d%2Fdropbear%20start%0A"
sleep 1
curl -X POST "http://192.168.31.1/cgi-bin/luci/;stok=${1}/api/misystem/arn_switch" -d "open=1&model=1&level=%0Apasswd%20-d%20root%0A"
```

#### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#222e5bc9ea6c4af59270de7b6f224183 "SSH 登录")SSH 登录

在上一步，我们已经成功开启了 SSH，下面使用 SSH 登录路由器，开始操作。

```
ssh root@192.168.31.1
```

👀

我在第一次连接时报错：

![Image 7: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2F8ab9566a-7742-4eb4-8501-c29501b00b10%2Fimage.png?table=block&id=47205cf1-8051-4a95-bbbf-228bfdbcd014&cache=v2)

这是加密算法不匹配导致，需要配置启用 ssh-rsa 。修改 `~/.ssh/config` ,添加内容如下：

```
  Host ax3000t
  HostName 192.168.31.1
  Port 22
  User root
  HostKeyAlgorithms +ssh-rsa
```

然后使用命令 `ssh ax3000t `连接即可。 或者不配置上述文件使用命令 `ssh -oHostKeyAlgorithms=+ssh-rsa root@192.168.31.1`

此时终于看到熟悉的 “Are you OK” 了

![Image 8: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2Fde9a6f16-8aeb-430c-8363-e4e07010f1f6%2Fimage.png?table=block&id=9240e12c-5f0b-47bf-aaf5-3377a674f863&cache=v2)

#### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#b24587ffb83d46269218635794e1976f "备份")备份

备份原厂分区信息

```
mkdir /tmp/bak
nanddump -f /tmp/bak/BL2.bin /dev/mtd1
nanddump -f /tmp/bak/Nvram.bin /dev/mtd2
nanddump -f /tmp/bak/Bdata.bin /dev/mtd3
nanddump -f /tmp/bak/Factory.bin /dev/mtd4
nanddump -f /tmp/bak/FIP.bin /dev/mtd5
nanddump -f /tmp/bak/ubi.bin /dev/mtd8
nanddump -f /tmp/bak/KF.bin /dev/mtd12
```

导出之后将其传输至其他的存储设备上。在电脑端执行

`scp -O -r ax3000t:/tmp/bak/ .`

💡

其中 `-O` 参数为设置 scp 传输模式为传统方式，不然会报错。报错原因是 openssh 9.0 及之后版本，scp 命令默认传输使用的是 SFTP 协议，而路由器是不支持的。

![Image 9: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2F9d09ea8d-224d-4495-9fd3-7f5709a77f22%2Fimage.png?table=block&id=20cef5a3-6a9e-404d-b684-06fe954aa82d&cache=v2)

#### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#09bd302c11f741cb908939fdfda2e04e "OpenWrt / ImmortalWrt 固件下载")OpenWrt / ImmortalWrt 固件下载

ImmortalWrt 是一个针对国人用户开发的基于 OpenWrt 的分支固件。

根据实际需要选择相应的固件即可，后续描述以 ImmortalWrt 为例，如需刷入 OpenWrt ，替换掉相应的文件即可。

OpenWrt:

固件：

*   [https://downloads.openwrt.org/releases/23.05.4/targets/mediatek/filogic/openwrt-23.05.4-mediatek-filogic-xiaomi\_mi-router-ax3000t-initramfs-factory.ubi](https://downloads.openwrt.org/releases/23.05.4/targets/mediatek/filogic/openwrt-23.05.4-mediatek-filogic-xiaomi_mi-router-ax3000t-initramfs-factory.ubi)

*   [https://downloads.openwrt.org/releases/23.05.4/targets/mediatek/filogic/openwrt-23.05.4-mediatek-filogic-xiaomi\_mi-router-ax3000t-squashfs-sysupgrade.bin](https://downloads.openwrt.org/releases/23.05.4/targets/mediatek/filogic/openwrt-23.05.4-mediatek-filogic-xiaomi_mi-router-ax3000t-squashfs-sysupgrade.bin)

ImmortalWrt:

固件：

*   [https://downloads.immortalwrt.org/releases/23.05.3/targets/mediatek/filogic/immortalwrt-23.05.3-mediatek-filogic-xiaomi\_mi-router-ax3000t-initramfs-factory.ubi](https://downloads.immortalwrt.org/releases/23.05.3/targets/mediatek/filogic/immortalwrt-23.05.3-mediatek-filogic-xiaomi_mi-router-ax3000t-initramfs-factory.ubi)

*   [https://downloads.immortalwrt.org/releases/23.05.3/targets/mediatek/filogic/immortalwrt-23.05.3-mediatek-filogic-xiaomi\_mi-router-ax3000t-squashfs-sysupgrade.bin](https://downloads.immortalwrt.org/releases/23.05.3/targets/mediatek/filogic/immortalwrt-23.05.3-mediatek-filogic-xiaomi_mi-router-ax3000t-squashfs-sysupgrade.bin)

#### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#bc83ac49b72141249be03e21c78acaea "刷入 initramfs system")刷入 initramfs system

将文件要刷入的文件上传到路由器 /tmp 目录

```
scp -O immortalwrt-23.05.3-mediatek-filogic-xiaomi_mi-router-ax3000t-initramfs-factory.ubi ax3000t:/tmp/
```

查看硬件信息：`cat /proc/cmdline`，视情况执行下面的命令

![Image 10: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2F42104ee0-879c-46de-8c28-302da9842d74%2Fimage.png?table=block&id=0a9b99e9-f9c1-4a4f-9695-d5e3a2733d2f&cache=v2)

**firmware = 0 时（此处为我的情况）：**

```
ubiformat /dev/mtd9 -y -f /tmp/immortalwrt-23.05.3-mediatek-filogic-xiaomi_mi-router-ax3000t-initramfs-factory.ubi
nvram set boot_wait=on
nvram set uart_en=1
nvram set flag_boot_rootfs=1
nvram set flag_last_success=1
nvram set flag_boot_success=1
nvram set flag_try_sys1_failed=0
nvram set flag_try_sys2_failed=0
nvram commit
reboot
```

**firmware = 1 时：**

```
ubiformat /dev/mtd8 -y -f /tmp/immortalwrt-23.05.3-mediatek-filogic-xiaomi_mi-router-ax3000t-initramfs-factory.ubi
nvram set boot_wait=on
nvram set boot_wait=on
nvram set uart_en=1
nvram set flag_boot_rootfs=0
nvram set flag_last_success=0
nvram set flag_boot_success=1
nvram set flag_try_sys1_failed=0
nvram set flag_try_sys2_failed=0
nvram commit
reboot
```

执行完后路由器会重启，进入到 initramfs system 。

#### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#bbedfba0688e4aeb811ccbf440bad1f5 "升级固件")升级固件

重启后 IP 发生变化，我这里路由器 IP 变为 192.168.1.1 ，使用此地址重新 SSH 登录。执行系统命令：

上传文件

```
scp -O immortalwrt-23.05.3-mediatek-filogic-xiaomi_mi-router-ax3000t-squash
fs-sysupgrade.bin ax3000t:/tmp/
```

执行命令

```
sysupgrade -n -v /tmp/immortalwrt-23.05.3-mediatek-filogic-xiaomi_mi-router-ax3000t-squash
fs-sysupgrade.bin
```

刷入后路由器会自动重启进入正式系统。

默认网关地址 [http://192.168.1.1](http://192.168.1.1/) ， 账号 root ，密码为空，进入后按照引导开始设置即可。（后续 SSH 登录也会用到这个密码）。

![Image 11: notion image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F38eb7257-66ea-481f-b02f-6d5b3b01bb66%2Fe6486127-0e4d-4954-865b-647ffa4f252d%2Fimage.png?table=block&id=db54b625-bf7c-4c15-9fa0-8560e2cadecb&cache=v2)

### [](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt#7ced048447934bcabee63f5c7d6c97d7 "参考")参考
