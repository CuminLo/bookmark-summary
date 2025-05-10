# How I Set Up New MacBooks
- URL: https://catalins.tech/how-i-setup-new-macbooks/
- Added At: 2025-05-10 07:52:21
- [Link To Text](2025-05-10-how-i-set-up-new-macbooks_raw.md)

## TL;DR
作者分享了用`Brewfile`和`defaults`命令自动化设置新MacBook的流程，包括批量安装应用、配置系统设置、优化Zsh插件和别名，并计划进一步用脚本简化流程。

## Summary
1. **背景介绍**：作者分享了自己设置新MacBook的自动化流程，避免了手动逐个安装应用和配置的繁琐工作，也不使用Time Machine恢复旧数据，而是选择全新开始。

2. **Brewfile使用**：
   - **功能**：通过`Brewfile`批量安装命令行工具、应用、字体和VS Code扩展。
   - **示例内容**：
     - **命令行工具**：包括`git`、`node`、`ffmpeg`、`zsh`及其插件等。
     - **应用**：如`Cursor`、`Discord`、`Raycast`、`Docker`等。
     - **字体**：如`Hack Nerd Font`、`JetBrains Mono`等。
   - **使用方法**：在根目录创建`Brewfile`后运行`brew bundle`命令即可安装所有内容。

3. **默认设置配置**：
   - **工具**：使用`defaults`命令行工具自定义MacBook和应用的设置。
   - **示例配置**：
     - 启用触控板轻点点击。
     - 禁止生成`.DS`文件。
     - 在Finder中显示路径栏和隐藏文件。
     - 保持文件夹在顶部显示。
   - **应用设置**：通过`activateSettings`命令使更改生效。

4. **Zsh插件安装**：
   - **推荐插件**：
     - `Git`：提供Git快捷命令。
     - `Zsh-autosuggestions`：基于历史命令的自动补全。
     - `Zsh-syntax-highlighting`：命令语法高亮。
     - `You-should-use`：提醒使用别名替代完整命令。
     - `Zsh-bat`：增强`cat`命令输出。
     - `nvm`：快速切换Node版本。

5. **Zsh别名设置**：
   - **用途**：通过别名（快捷命令）减少输入次数，提升终端效率。
   - **参考链接**：作者提供了详细别名列表的博客链接。

6. **后续计划**：
   - 创建bash脚本简化安装和命令执行流程。
   - 添加更多命令以进一步完善自动化设置。

7. **反馈与支持**：
   - 邀请读者提供建议或反馈。
   - 支持博客的付费选项（非强制）。
