Title: How I Set Up New MacBooks

URL Source: https://catalins.tech/how-i-setup-new-macbooks/

Published Time: 2025-04-14T17:50:47.000Z

Markdown Content:
I recently got a new MacBook and procrastinated setting it up for a few days. Installing apps individually and configuring the machine manually feels like a huge chore.

I also don't want to bring the trash from the previous machines with the Time Machine feature. I like to start fresh each time. If something is important enough, it'll find its way to the new machine.

In this article, I'll show you how to automate the process of setting up your new MacBook.

Brewfile
--------

The `Brewfile` allows you to install command line utilities, applications, fonts and even Visual Studio Code (and forks/variants) extensions in bulk rather than installing them one by one.

Here's my `Brewfile`:

```
# command-line utilities

brew "oven-sh/bun/bun"
brew "git"
brew "node"
brew "ffmpeg"
brew "cask"
brew "postgresql@17"
brew "zsh"
brew "zsh-autosuggestions"
brew "zsh-completions"
brew "zsh-syntax-highlighting"
brew "pnpm"
brew "npm"
brew "gh"

# apps

cask "cursor"
cask "discord"
cask "raycast"
cask "whatsapp"
cask "warp"
cask "cleanshot"
cask "google-chrome"
cask "postman"
cask "screen-studio"
cask "imageoptim"
cask "bitwarden"
cask "docker"
cask "obs"
cask "elgato-stream-deck"
cask "elgato-camera-hub"
cask "zoom"
cask "vlc"
cask "pgadmin4"
cask "nordvpn"
cask "zed"
cask "ngrok"

# fonts

cask "font-hack-nerd-font"
cask "font-menlo-for-powerline"
cask "font-jetbrains-mono"
cask "font-jetbrains-mono-nerd-font"
```

You can create the `Brewfile` in the root directory of your machine and run `brew bundle` once you're done with it.

The `bundle` command installs all the things you specified in the file.

Settings `defaults`
-------------------

macOS also provides the `defaults` utility to customize the settings of your MacBooks and certain applications.

Instead of updating the settings through the UI, you can achieve the same things by using the `defaults` utility in the terminal.

```
# Enable tap-to-click for the trackpad and show the correct state in System Preferences
defaults write com.apple.AppleMultitouchTrackpad Clicking -bool true
defaults -currentHost write -g com.apple.mouse.tapBehavior -int 1

# Disable the .DS file creation
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true

# Show the path bar in the Finder
defaults write com.apple.finder "ShowPathbar" -bool "true" && killall Finder

# Show hidden files in the Finder
defaults write com.apple.finder "AppleShowAllFiles" -bool "false" && killall Finder

# Keep folders on top in Finder
defaults write com.apple.finder "_FXSortFoldersFirst" -bool "true" && killall Finder

# Keep folders on top on Desktop
defaults write com.apple.finder "_FXSortFoldersFirstOnDesktop" -bool "true" && killall Finder

# Apply the settings
/System/Library/PrivateFrameworks/SystemAdministration.framework/Resources/activateSettings -u
```

This is very handy because you don't have to make all the changes manually.

Zsh Plugins
-----------

Lastly, I install my favorite 6 [Zsh plugins](https://catalins.tech/zsh-plugins/):

*   `Git` - a plugin that provides useful Git shortcuts like `gco` instead of `git checkout`, for example.
*   `Zsh-autosuggestions` - autocomplete functionality based on your history of previous commands.
*   `Zsh-syntax-highlighting` - syntax highlighting for the commands you type in the terminal.
*   `You-should-use` - it reminds you of your aliases (shortcuts) when you use the full command. If you use `git checkout` instead of `gco`, it'll display the alias for the command, for example.
*   `Zsh-bat` - it improves the output of the `cat` command.
*   [nvm](https://catalins.tech/node-version-manager-oh-my-zsh/) - easily switch between Node versions

Zsh Aliases for Terminal Productivity
-------------------------------------

I use a handful of [Zsh aliases (shortcuts)](https://catalins.tech/zsh-aliases/) to save a few keystrokes and improve my terminal productivity.

What's Next
-----------

*   Create bash scripts to easily run the installation and commands
*   Add more commands

If you have any suggestions or feedback, please leave a comment.

Pay what you like 🧡
--------------------

If you like this content and it helped you, please consider supporting this blog.
