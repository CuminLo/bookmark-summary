# leitbogioro/Tools: Something about tools
- URL: https://github.com/leitbogioro/Tools
- Added At: 2025-01-09 08:33:19
- [Link To Text](2025-01-09-leitbogioro-tools-something-about-tools_raw.md)

## TL;DR
“Tools”项目提供了一套名为`InstallNET.sh`的脚本，支持多种Linux发行版和Windows系统的重装。脚本支持自定义镜像源、密码、端口等参数，并提供了详细的安装指南和常见错误解决方案。项目采用Unlicense许可证，允许自由使用和修改，适用于研究和评估目的。

## Summary
1. **项目概述**：
   - 项目名为“Tools”，主要提供与脚本相关的工具和资源。
   - 包含Linux重装系统的使用指南和脚本。

2. **Linux重装系统使用**：
   - 提供了详细的Linux重装系统脚本（`InstallNET.sh`）的使用说明。
   - 支持多种Linux发行版，包括Debian、Ubuntu、Kali、CentOS、AlmaLinux、RockyLinux、Fedora等。
   - 提供了不同发行版的安装命令示例，如：
     - Debian 12：`bash InstallNET.sh -debian`
     - Kali Rolling：`bash InstallNET.sh -kali`
     - Alpine Linux Edge：`bash InstallNET.sh -alpine`
     - CentOS 9 Stream：`bash InstallNET.sh -centos`
     - Fedora 39：`bash InstallNET.sh -fedora`
     - Ubuntu 22.04：`bash InstallNET.sh -ubuntu`
     - Windows 11 Pro for Workstations：`bash InstallNET.sh -windows`

3. **依赖和操作系统支持**：
   - 列出了支持的Linux发行版及其依赖关系：
     - Debian系列（Debian、Ubuntu、Kali）
     - RedHat系列（CentOS、AlmaLinux、RockyLinux、OracleLinux、Fedora等）
     - Alpine Linux

4. **下载和安装脚本**：
   - 提供了脚本的下载命令：
     - 全球服务器：`wget --no-check-certificate -qO InstallNET.sh 'https://raw.githubusercontent.com/leitbogioro/Tools/master/Linux_reinstall/InstallNET.sh' && chmod a+x InstallNET.sh`
     - 中国大陆服务器：`wget --no-check-certificate -qO InstallNET.sh 'https://gitee.com/mb9e8j2/Tools/raw/master/Linux_reinstall/InstallNET.sh' && chmod a+x InstallNET.sh`

5. **默认配置**：
   - 提供了默认的SSH和RDP服务配置：
     - 默认用户名：Linux为`root`，Windows为`Administrator`。
     - 默认密码：Linux为`LeitboGi0ro`，Windows为`Teddysun.com`。
     - 默认端口：Linux为系统原有端口，Windows为3389。

6. **参数详细说明**：
   - 提供了脚本的详细参数说明，包括：
     - `-debian`：安装Debian 7及更高版本。
     - `-kali`：安装Kali Rolling、Development和Experimental版本。
     - `-centos`：安装CentOS 7及更高版本。
     - `-alpine`：安装Alpine Linux 3.16及更高版本。
     - `-almalinux`：安装AlmaLinux 8及更高版本。
     - `-rockylinux`：安装RockyLinux 8及更高版本。
     - `-fedora`：安装Fedora 38及更高版本。
     - `-ubuntu`：安装Ubuntu 20.04及更高版本。
     - `-windows`：安装Windows 10/11/2012/2016/2019/2022。

7. **高级用法**：
   - 提供了针对不同地区的镜像源选择，如日本、香港、新加坡、韩国、台湾、美国、加拿大、英国、德国、法国、荷兰、瑞士、俄罗斯、澳大利亚等。
   - 示例命令：
     - 日本：`bash InstallNET.sh -debian 12 -mirror "http://ftp.riken.jp/Linux/debian/debian/"`
     - 香港：`bash InstallNET.sh -debian 12 -mirror "http://ftp.hk.debian.org/debian/"`
     - 美国：`bash InstallNET.sh -debian 12 -mirror "https://mirrors.ocf.berkeley.edu/debian/"`

8. **错误和解决方案**：
   - 提供了常见错误的解决方案，如CentOS 8源失效问题：
     - 修复命令：
       ```bash
       cd /etc/yum.repos.d/
       sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
       sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
       ```

9. **完整使用示例**：
   - 提供了完整的脚本使用示例，包括指定操作系统类型、版本、架构、端口、密码、镜像源、DD镜像、时区等参数。

10. **法律声明**：
    - 项目采用Unlicense许可证，允许任何人自由使用、修改和分发。
    - 项目不提供任何Windows激活服务，仅用于研究和评估目的。
    - 项目不承担因使用脚本导致的任何法律责任。

11. **功能和更新**：
    - 支持从Linux系统安装到Windows系统。
    - 支持IPv6单栈和双栈网络配置。
    - 支持低内存设备，推荐Debian系列内存512MB以上，RedHat系列内存2GB以上。
    - 支持自动检测CPU架构并安装对应的系统版本。

12. **其他**：
    - 提供了XanMod内核的安装教程，优化网络连接和硬件兼容性。
    - 提供了BBR（Bottleneck Bandwidth and Round-trip propagation time）网络优化的配置方法。
