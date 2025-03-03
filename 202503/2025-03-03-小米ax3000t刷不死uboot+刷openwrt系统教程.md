# 小米AX3000T刷不死uboot+刷openwrt系统教程
- URL: https://blog.wwang.pw/post/ax3000t
- Added At: 2025-03-03 03:27:18
- [Link To Text](2025-03-03-小米ax3000t刷不死uboot+刷openwrt系统教程_raw.md)

## TL;DR
本文详细介绍了小米AX3000T路由器的硬件规格、刷机步骤及注意事项。内容包括开启SSH、刷入uboot、刷入OpenWrt固件的具体操作，以及完成后的后台地址和密码。文章还提供了购买已刷好固件的路由器链接，并声明了版权协议。适合有一定技术基础的用户参考。

## Summary
1. **硬件规格**：
   - **SoC**：MediaTek MT7981B 2x A53
   - **Flash**：ESMT F50L1G41LB 128MB
   - **RAM**：NT52B128M16JR-FL 256MB
   - **Ethernet**：4x 10/100/1000 Mbps
   - **Switch**：MediaTek MT7531AE
   - **WiFi**：MediaTek MT7976C
   - **Button**：Reset, Mesh
   - **Power**：DC 12V 1A

2. **刷机文件**：
   - **下载地址**：https://pan.wwang.pw/%F0%9F%94%A5OpenWrt_LEAN/%E5%B0%8F%E7%B1%B3AX3000T
   - **备用获取方式**：如地址失效，加群获取。

3. **开启SSH**：
   - **步骤**：
     1. 使用`curl`命令开启SSH，替换`stok=`后面的值为自己的。
     2. 使用WIN+R唤起命令提示符，依次输入命令，每条命令输入完成后出现`{"code":0}`表示成功。
     3. 获取root密码：访问https://wwang.pw/mi/，输入路由器SN获取密码。

4. **刷入uboot**：
   - **步骤**：
     1. 打开MobaXterm，输入路由器管理地址`192.168.31.1`，用户名`root`，密码为刚刚获取的密码。
     2. 上传`mt7981_ax3000t-fip-fixed-parts-multi-layout.bin`文件到`tmp`目录。
     3. 输入命令`mtd write /tmp/mt7981_ax3000t-fip-fixed-parts-multi-layout.bin FIP`。

5. **刷OPENWRT**：
   - **步骤**：
     1. 进入uboot WEB页面，修改电脑IP为`192.168.1.*`，网关为`192.168.1.1`。
     2. 拔掉电源，按住reset键，蓝灯常亮即可进入BOOT，浏览器输入`192.168.1.1`，进入BOOT页面。
     3. 刷入H大的闭源immortalwrt固件或LEAN固件。

6. **完成**：
   - **后台地址**：`192.168.5.1`
   - **密码**：`password`
   - **注意事项**：靠近电源接口的网口是WAN口，进不去后台的，插拔电源重启一下即可。

7. **购买路由器**：
   - **购买链接**：https://shop.wwang.pw/
   - **服务**：小店已上架此款路由器，刷好固件再发货，不愿意折腾的可以下单。

8. **版权声明**：
   - **协议**：本博客所有文章除特别声明外，均采用 CC BY-NC-SA 4.0 许可协议。
   - **来源**：文章作者: XiaoWan💦，文章链接: https://blog.wwang.pw/post/ax3000t.html
