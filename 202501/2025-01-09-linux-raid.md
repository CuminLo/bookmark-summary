# Linux RAID
- URL: https://webmin.com/docs/modules/linux-raid/
- Added At: 2025-01-09 08:30:00
- [Link To Text](2025-01-09-linux-raid_raw.md)

## TL;DR
RAID技术通过将多个磁盘组合成虚拟设备，提升数据读写速度和安全性。常见的RAID级别包括RAID 0、RAID 1、RAID 4和RAID 5，各有优缺点。Linux系统中使用`mdadm`命令管理软件RAID，支持创建、监控和修复RAID阵列。硬件RAID性能更高，但成本较高且灵活性较低。

## Summary
1. **RAID简介**：
   - RAID（Redundant Array of Inexpensive Disks）是一种将多个磁盘分区组合成一个大型虚拟设备的技术，具有以下优势：
     - 创建单一文件系统，容量为所有硬盘的总和。
     - 读写速度通常比单个磁盘更快，因为数据分布在多个驱动器上。
     - 通过冗余配置，即使一个硬盘故障，数据仍可恢复。

2. **RAID级别**：
   - **线性模式（Concatenated or Linear）**：
     - 将分区按顺序组合，数据依次写入磁盘，不提升访问速度。
   - **RAID 0（Striped）**：
     - 数据均匀分布在所有磁盘上，读写速度显著提升，但不支持不同大小的磁盘。
   - **RAID 1（Mirrored）**：
     - 所有分区存储相同数据，数据安全性高，但磁盘利用率低，写入速度受限于最慢的磁盘。
   - **RAID 4（Parity）**：
     - 提供单磁盘故障保护，读取速度提升，但写入速度受限于专用奇偶校验磁盘。
   - **RAID 5（Redundant）**：
     - 提供单磁盘故障保护，读写速度提升，但需要牺牲部分磁盘空间存储冗余信息。

3. **Linux RAID模块**：
   - 该模块用于创建、格式化和删除Linux系统中的RAID阵列。
   - 如果系统缺少必要的命令或内核未编译RAID支持，会显示错误信息。
   - 创建新RAID设备的步骤：
     - 在本地磁盘分区模块中为每个磁盘创建分区，分区类型设置为Linux raid。
     - 可能需要重启系统以检测新分区。
     - 选择RAID级别并选择分区创建RAID设备。
     - 强制初始化RAID以确保分区无文件系统。
     - 创建文件系统并挂载。

4. **`mdadm`命令**：
   - `mdadm`是Linux中管理软件RAID阵列的主要工具。
   - 主要功能：
     - 创建各种级别的RAID阵列。
     - 监控阵列的健康状态。
     - 管理阵列，如添加或移除磁盘。
     - 修复阵列，如替换故障磁盘并重建阵列。
   - 常用命令：
     - 创建RAID阵列：`mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1`
     - 监控RAID状态：`cat /proc/mdstat`
     - 添加备用磁盘：`mdadm /dev/md0 --add /dev/sdc1`
     - 移除磁盘：`mdadm /dev/md0 --fail /dev/sda1 --remove /dev/sda1`
     - 停止并删除RAID阵列：`mdadm --stop /dev/md0` 和 `mdadm --remove /dev/md0`

5. **硬件与软件RAID对比**：
   - 软件RAID灵活且无需专用硬件，但会增加CPU负担。
   - 硬件RAID使用专用控制器，性能更高但成本较高且灵活性较低。

6. **相关模块**：
   - [Logical Volume Management](https://webmin.com/docs/modules/logical-volume-management)
   - [Partitions on Local Disks](https://webmin.com/docs/modules/partitions-on-local-disks)
