# 构建一个基于 Debian 的 DIY N100 NAS，第二部分：软件 | PCat 的秘密之地 --- Building a N100 Debian-based DIY NAS, Part 2: Software | PCat's Secret Place
- URL: https://tongkl.com/building-a-nas-part-2/
- Added At: 2025-01-09 08:27:50
- [Link To Text](2025-01-09-构建一个基于-debian-的-diy-n100-nas，第二部分：软件-pcat-的秘密之地-----building-a-n100-debian-based-diy-nas,-part-2-software-pcat's-secret-place_raw.md)

## TL;DR
本文是DIY NAS系列的第二部分，详细介绍了如何将硬件配置为功能齐全的NAS系统。使用Debian 12作为操作系统，配置了ZFS存储池、网络共享、定期快照和磁盘检查等功能。文章还涵盖了硬盘休眠、SSH安全设置、Docker和NGINX安装等内容，并进行了性能测试。未来将探讨备份策略以进一步完善系统。

## Summary
1. **系列文章背景**：本文是DIY NAS系列的第二部分，重点介绍软件配置，将硬件转变为功能齐全的NAS。

2. **操作系统安装**：
   - 使用Debian 12作为基础操作系统。
   - 假设操作系统已安装完毕，网络接口命名为`enp2s0`。

3. **安装必要工具**：
   - 修改`sources.list`以启用ZFS安装。
   - 更新系统并安装常用工具，如`curl`、`openssh-server`、`vim`等。
   - 提示：如果不需要亚洲字幕，可以跳过安装`fonts-noto-cjk`。

4. **配置硬盘休眠**：
   - 使用`hdparm`配置硬盘在空闲时休眠，以节省电力。
   - 设置硬盘在20分钟无活动后休眠。
   - 修改`/etc/hdparm.conf`文件并重启系统以应用更改。

5. **网络配置**：
   - 设置主机名为`nas`。
   - 配置静态IP地址、IPv6地址和VLAN接口。
   - 修改`/etc/network/interfaces`文件并重启网络服务。
   - 配置DNS服务器为路由器地址。

6. **禁用SSH密码认证**：
   - 修改`/etc/ssh/sshd_config`文件，禁用密码认证，仅允许密钥认证。
   - 重启SSH服务以应用更改。

7. **安装NGINX和Docker**：
   - 安装最新版本的NGINX和Docker。
   - 为Docker启用IPv6支持，以改善P2P连接。

8. **安装ZFS并创建存储池**：
   - 安装ZFS相关软件包。
   - 创建镜像存储池，并设置压缩、属性等参数。
   - 创建多个数据集以分别管理不同类型的数据。

9. **配置网络共享**：
   - 选择NFS作为网络共享协议，安装NFS服务器并配置共享文件夹。
   - 修改`/etc/exports`文件并重启NFS服务。

10. **安装其他服务**：
    - 安装并迁移多个服务到NAS，如Immich、Jellyfin、Audiobookshelf、Frigate和qBittorrent。

11. **配置定期快照**：
    - 使用`zfs-auto-snapshot`工具配置定期快照。
    - 修改`/etc/cron.d/zfs-auto-snapshot`文件以设置快照保留策略。
    - 为不需要快照的数据集禁用自动快照。

12. **配置定期磁盘检查和SMART测试**：
    - 使用ZFS的`scrub`功能检测和修复文件系统级别的错误。
    - 使用`smartd`工具定期进行SMART测试，并配置Telegram通知。
    - 修改`/etc/zfs/zed.d/zed.rc`和`/etc/smartd.conf`文件以启用通知。

13. **性能测试**：
    - 使用`dd`命令测试NAS的读写性能。
    - 创建测试数据集并运行读写测试脚本。
    - 测试结果显示写入速度为171 MB/s，读取速度为193 MB/s。

14. **总结**：
    - 本文详细介绍了如何将DIY NAS硬件配置为功能齐全的NAS系统。
    - 使用Debian作为操作系统，配置了ZFS存储池、网络共享、定期快照和磁盘检查等功能。
    - 未来将介绍备份策略，以进一步完善NAS系统。
