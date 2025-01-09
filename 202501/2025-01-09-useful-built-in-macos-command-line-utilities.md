# Useful built-in macOS command-line utilities
- URL: https://weiyen.net/articles/useful-macos-cmd-line-utilities/
- Added At: 2025-01-09 08:52:59
- [Link To Text](2025-01-09-useful-built-in-macos-command-line-utilities_raw.md)

## TL;DR
本文介绍了多个macOS终端命令的实用技巧，包括访问Keychain、打开文件、复制粘贴、获取UTC时间、测试网络速度、防止睡眠、生成UUID等。还推荐了一些高级命令和资源链接，帮助用户更高效地使用macOS命令行工具。

## Summary
1. **Keychain访问**：
   - 使用`security`命令可以编程方式访问存储在macOS Keychain中的秘密信息。
   - 示例命令：`security find-internet-password -s "https://example.com"`。
   - 适用于编写使用本地存储凭证的自动化脚本。
   - 链接：[https://ss64.com/mac/security.html](https://ss64.com/mac/security.html)。
   - 额外提示：1Password用户可以使用1Password CLI访问1Password项目。

2. **终端打开文件**：
   - 使用`open`命令可以从终端打开文件。
   - 示例命令：`open file.txt`。
   - 文件将在默认应用程序中打开，就像在Finder中双击一样。
   - 链接：[https://ss64.com/mac/open.html](https://ss64.com/mac/open.html)。

3. **复制和粘贴**：
   - `pbcopy`和`pbpaste`是用于复制和粘贴文本到剪贴板的命令行工具。
   - `pbcopy`将标准输入的内容复制到剪贴板。
     - 示例命令：`echo "Hello, world!" | pbcopy;`。
   - `pbpaste`将剪贴板的内容输出到标准输出。
     - 示例命令：`pbpaste`。
   - 适用于将数据从文件传输到浏览器或其他GUI应用程序。
   - 链接：
     - [https://ss64.com/mac/pbcopy.html](https://ss64.com/mac/pbcopy.html)
     - [https://ss64.com/mac/pbpaste.html](https://ss64.com/mac/pbpaste.html)

4. **UTC时间**：
   - 使用`date -u`或`TZ=UTC date`可以获取当前的UTC时间。
   - 适用于查看服务器日志等场景。
   - 链接：[https://ss64.com/mac/date.html](https://ss64.com/mac/date.html)。

5. **网络速度测试**：
   - 使用`networkQuality`命令可以直接在终端运行网络速度测试。
   - 注意命令中的大写“Q”。
   - 链接：[https://ss64.com/mac/networkquality.html](https://ss64.com/mac/networkquality.html)。

6. **防止Mac睡眠**：
   - 使用`caffeinate`命令可以防止Mac进入睡眠状态。
   - 示例命令：`caffeinate`。
   - 适用于运行服务器时防止Mac睡眠。
   - 链接：[https://ss64.com/mac/caffeinate.html](https://ss64.com/mac/caffeinate.html)。

7. **生成UUID**：
   - 使用`uuidgen`命令可以生成UUID。
   - 默认输出为大写，可以通过`tr`和`pbcopy`将其转换为小写并复制到剪贴板。
     - 示例命令：`uuidgen | tr '[:upper:]' '[:lower:]' | pbcopy`。
   - 适用于编写需要ID的单元测试。
   - 链接：[https://ss64.com/mac/uuidgen.html](https://ss64.com/mac/uuidgen.html)。

8. **其他实用命令**：
   - `mdfind`：在终端中进行Spotlight搜索。
   - `say`：让Mac朗读文本。
   - `screencapture`：截屏并保存到文件。
   - `networksetup`：编程方式配置网络设置。
   - 链接：
     - [https://ss64.com/mac/mdfind.html](https://ss64.com/mac/mdfind.html)
     - [https://ss64.com/mac/say.html](https://ss64.com/mac/say.html)
     - [https://ss64.com/mac/screencapture.html](https://ss64.com/mac/screencapture.html)
     - [https://ss64.com/mac/networksetup.html](https://ss64.com/mac/networksetup.html)

9. **更新后的推荐命令**：
   - `sips`：用于图像格式转换。
   - `afinfo`：探测音频文件的元数据。
   - `mdls`：探测各种文件的元数据。
   - `afconvert`：用于音频格式转换。
   - `diskutil`：管理磁盘卷。
   - `powermetrics`：监控系统功耗。
   - `pmset`：电源管理任务。
   - `dot_clean`：删除dot_underscore文件。
   - 链接：
     - [https://ss64.com/mac/sips.html](https://ss64.com/mac/sips.html)
     - [https://ss64.com/mac/afinfo.html](https://ss64.com/mac/afinfo.html)
     - [https://ss64.com/mac/mdls.html](https://ss64.com/mac/mdls.html)
     - [https://ss64.com/mac/afconvert.html](https://ss64.com/mac/afconvert.html)
     - [https://ss64.com/mac/diskutil.html](https://ss64.com/mac/diskutil.html)
     - [https://ss64.com/mac/powermetrics.html](https://ss64.com/mac/powermetrics.html)
     - [https://ss64.com/mac/pmset.html](https://ss64.com/mac/pmset.html)
     - [https://ss64.com/mac/dot_clean.html](https://ss64.com/mac/dot_clean.html)

10. **其他资源**：
    - [https://saurabhs.org/advanced-macos-commands](https://saurabhs.org/advanced-macos-commands)
    - [https://notes.billmill.org/computer_usage/mac_os/mac_os_command_line_programs.html](https://notes.billmill.org/computer_usage/mac_os/mac_os_command_line_programs.html)
