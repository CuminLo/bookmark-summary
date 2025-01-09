# How To Configure Storage On Linux
- URL: https://embeddedprojects101.com/the-beginners-guide-to-linux-storage-configuration/
- Added At: 2025-01-09 08:25:26
- [Link To Text](2025-01-09-how-to-configure-storage-on-linux_raw.md)

## TL;DR
本文为Linux存储配置的初学者指南，介绍了块存储设备命名、磁盘分区、文件系统创建与挂载以及逻辑卷管理器（LVM）等关键概念和命令。通过使用`lsblk`、`fdisk`、`mkfs`、`mount`等命令，用户可以管理存储设备、创建分区和文件系统，并通过LVM实现灵活的存储管理。推荐阅读相关书籍以深入学习。

## Summary
1. **文章简介**：本文为Linux存储配置的初学者指南，介绍了块存储设备命名、磁盘分区、文件系统创建与挂载以及逻辑卷管理器（LVM）等关键概念和命令。

2. **存储设备命名**：
   - Linux将存储设备视为文件，存储在`/dev`目录下。
   - 设备文件名取决于总线类型和插入顺序，例如SCSI/SATA设备命名为`sda`、`sdb`等，NVMe设备命名为`nvme0n1`等。
   - 使用`ls /dev`命令可以查看所有块设备。

3. **磁盘分区**：
   - **分区定义**：分区是将块设备划分为逻辑块设备的操作，使用`lsblk`命令可以查看所有磁盘和分区。
   - **分区方案**：主要有MBR（主引导记录）和GPT（GUID分区表）两种，GPT支持更大的磁盘容量。
   - **创建分区**：使用`fdisk`（MBR）或`gdisk`（GPT）命令创建分区表，并通过交互式命令设置分区大小和类型。

4. **文件系统**：
   - **文件系统定义**：文件系统是建立在分区之上的逻辑抽象层，常见的Linux文件系统有ext4和xfs。
   - **创建文件系统**：使用`mkfs`命令创建文件系统，例如`mkfs -t xfs /dev/sdb1`。
   - **挂载文件系统**：使用`mount`命令将文件系统挂载到指定目录，例如`mount /dev/sdb1 /data`。
   - **持久化挂载**：通过编辑`/etc/fstab`文件，确保挂载在系统重启后仍然有效。

5. **逻辑卷管理器（LVM）**：
   - **LVM定义**：LVM是一种存储技术，提供逻辑卷、精简配置、快照和软件RAID等高级功能。
   - **LVM结构**：分为物理卷（PV）、卷组（VG）和逻辑卷（LV）三层。
   - **创建PV**：使用`pvcreate`命令将块设备或分区标记为物理卷。
   - **创建VG**：使用`vgcreate`命令创建卷组，例如`vgcreate data_vg /dev/sdb2 /dev/nvme0n1 /dev/nvme0n2`。
   - **创建LV**：使用`lvcreate`命令创建逻辑卷，例如`lvcreate --size 15G --name data_lv data_vg`。
   - **文件系统与挂载**：在LV上创建文件系统并挂载，例如`mkfs -t xfs /dev/data_vg/data_lv`和`mount /dev/data_vg/data_lv /data2`。

6. **总结**：
   - Linux将块存储设备视为文件，存储在`/dev`目录下。
   - 使用分区、文件系统和挂载操作来管理存储空间。
   - LVM提供灵活的存储管理功能，支持动态调整逻辑卷大小。
   - 关键命令包括`lsblk`、`gdisk`、`fdisk`、`mkfs`、`blkid`、`mount`、`df`、`pvcreate`、`vgcreate`和`lvcreate`。

7. **推荐资源**：建议阅读Sander Van Vugt的《Red Hat RHCSA 9 Cert Guide: EX200》以深入学习Linux存储管理。
