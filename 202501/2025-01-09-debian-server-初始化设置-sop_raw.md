Title: Debian Server 初始化设置 SOP

URL Source: https://blog.xm.mk/posts/89da/

Published Time: 2024-02-19T20:20:19+08:00

Markdown Content:
Cyrus's Blog湫谷
文章
标签
分类
友链
主页
文章
Debian Server 初始化设置 SOP
Debian Server 初始化设置 SOP
 Cyrus 收录于  技术笔记
2024-02-19 约 2500 字 预计阅读 5 分钟 - 次阅读 

最近经常配置 VPS 和 LXC 之类的，需要从头配置系统，写个文档简单记录一下。

警告
声明：本文内容为笔者个人经验记录，仅供参考和学习交流，笔者不承担因遵循本文指引而产生的任何技术问题或损失的责任。请读者在实际操作前仔细评估风险，本文不作为任何形式的技术指导或建议。
注意
配置过程中需要主机具备可以连接 GitHub 的能力，如果服务器位于中国大陆境内，请自行解决连接问题，比如使用镜像服务（如 mirror.ghproxy.com）等。
安装系统

从各大云服务商买来的预装系统一般都带有各种监控软件，无法保证系统纯净性，所以首先用 dd 重装官方系统，这边用的是 leitbogioro 的 reinstall 脚本。

1
2

	
sudo -s # 或 su - 切换到 root 账户
wget --no-check-certificate -qO InstallNET.sh 'https://raw.githubusercontent.com/leitbogioro/Tools/master/Linux_reinstall/InstallNET.sh' && chmod a+x InstallNET.sh

下面的命令安装 Debian 12 系统，并设置 SSH 端口为 25022，root 密码为 YOUR_ROOT_PASSWORD，主机名为 Aries。

1

	
bash InstallNET.sh -debian 12 -port 25022 -password "YOUR_ROOT_PASSWORD" -hostname "Aries"

可以使用 -mirror 参数指定镜像，例如 bash InstallNET.sh -debian 12 -mirror "http://ftp.hk.debian.org/debian/" （不推荐使用中国大陆的镜像），更多说明参考文档： Advanced usage。

提示
请仔细确认 dd 脚本是否正确设置了所有必要项目，部分云服务器使用内网需要额外配置 IP 等，注意核对和备份相关设置。

重启执行安装：

1

	
reboot
系统配置
提示
该部分均在 root 账户下操作
安装基本软件包
1
2

	
apt update && apt upgrade
apt install -y build-essential sudo vim curl wget ufw autojump git tmux zsh tree zstd zip unzip lsof
技巧

如果无法连接官方软件源，可以使用镜像源（非必要不更换）。

 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11

	
# 科大源
sed -i.bak 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list

# 清华源
sed -i.bak 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list

# 阿里源
sed -i.bak 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list

# 腾讯源
sed -i.bak 's/deb.debian.org/mirrors.tencent.com/g' /etc/apt/sources.list
设置主机名和用户

主机名以 Aries（白羊座），用户名以 cyrus 为例，自行替换

设置主机名
1

	
hostnamectl set-hostname Aries
设置用户
1
2
3
4
5
6
7
8

	
# 设置一个用户名变量，方便后面使用
USERNAME=cyrus
# 添加用户
useradd -m $USERNAME
# 加入 sudo 组
usermod -aG sudo $USERNAME
# 设置密码
passwd $USERNAME

如果希望用户在 sudo 操作的时候不需要输入密码，执行 visudo，并且在最后添加以下内容，保存退出。注意要加在组规则的后面，否则会被组规则覆盖掉。

1

	
cyrus ALL=(ALL) NOPASSWD: ALL
时区和 NTP 配置
设置时区
1

	
timedatectl set-timezone Asia/Hong_Kong
配置 NTP 时间同步

为确保服务器时间准确，需要配置 NTP 时间同步。可以用 systemd-timesyncd 或者 ntp，需要注意两者并不兼容，需要二选一，这里用 ntp。

1
2

	
timedatectl set-ntp false # 禁用 timesyncd 服务
apt install -y ntp

在 /etc/ntpsec/ntp.conf 中，ntp 默认配置了以下几个 ntp pool，如果没有指定服务器的需求可以直接用。

1
2
3
4

	
pool 0.debian.pool.ntp.org iburst
pool 1.debian.pool.ntp.org iburst
pool 2.debian.pool.ntp.org iburst
pool 3.debian.pool.ntp.org iburst

如果需要指定 NTP 服务器，用 # 注释掉以上的 pool，并添加服务器，例如：

1
2
3
4

	
server 0.pool.ntp.org iburst
server time.cloudflare.com iburst
server time.apple.com iburst
server time1.google.com iburst

更多公共服务可以参考 国内外常用公共NTP网络时间同步服务器地址

配置完成后重启服务，可以使用 ntpq -p 查看同步状态。

1

	
systemctl restart ntpd
UTF-8 和中文支持

刚安装完的系统由于尚未配置为支持 UTF-8编码的中文字符集，可能无法显示中文字符，这里配置一下。

技巧
该部分可以通过 dpkg-reconfigure locales 交互式配置，或者使用以下命令设置，请确保理解其含义。
配置字符集
1
2
3

	
sed -i '/^#.*en_US.UTF-8 UTF-8/s/^#//' /etc/locale.gen
sed -i '/^#.*zh_CN.UTF-8 UTF-8/s/^#//' /etc/locale.gen
locale-gen
配置默认语言

如果想要使用中文，将下面命令的 en_US 替换为 zh_CN：

1
2

	
echo 'LANG=en_US.UTF-8' > /etc/default/locale
source /etc/default/locale
配置 Swap（虚拟内存）

许多小内存机器内存不够用，需要配置交换空间缓解一定的内存压力。

首先使用 free -h 确定是否开启了 Swap，如果最后一行输出是 Swap 0 0 0，说明没有开启，继续后续步骤。

1
2
3

	
                total        used        free      shared  buff/cache   available
Mem:           960Mi       608Mi        88Mi       1.6Mi       423Mi       352Mi
Swap:              0          0        0

创建一个 4G 的 Swap 文件，如果要调整大小，修改对应的值即可。

1

	
fallocate -l 4G /swapfile

建立 Swap 文件系统

1
2

	
chmod 600 /swapfile
mkswap /swapfile

开启 Swap

1

	
swapon /swapfile

再次使用 free -h 确认 Swap 开启状态，应当为对应设置的大小。

1
2
3

	
               total        used        free      shared  buff/cache   available
Mem:           960Mi       608Mi        88Mi       1.6Mi       423Mi       352Mi
Swap:          4.0Gi       569Mi       3.4Gi

swapon 开启的 Swap 重启后会失效，为了开机启动时自动开启 Swap，需要在 /etc/fstab 中进行配置。

1
2
3

	
# 备份 fstab
cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
安装 Docker 和 docker-compose
安装 Docker 并配置用户权限

使用官方安装脚本安装 docker，并将自己的用户添加到 docker 组：

1
2

	
curl -fsSL https://get.docker.com | bash -s docker
usermod -aG docker $USERNAME
安装 docker-compose

安装 docker-compose：

1
2
3
4

	
COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
sh -c "curl -L https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
chmod +x /usr/local/bin/docker-compose
sh -c "curl -L https://raw.githubusercontent.com/docker/compose/${COMPOSE_VERSION}/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose"
防火墙配置

使用 UFW 配置防火墙，可参考 UFW - Community Help Wiki。

警告
配置防火墙需谨慎，并放通 SSH 端口后再启用防火墙，否则可能会导致失联。

放通端口的基本命令格式 ： ufw allow <port>/<optional: protocol>

限定 IP 访问指定端口：ufw allow proto tcp from 10.0.0.1 to any port 22

基本配置
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14

	
# 放通 SSH 端口，自行修改对应端口
ufw allow 25022/tcp

# 放通 Web 服务端口（80/tcp 443/tcp）
ufw allow 'WWW Full' 

# 启用日志
ufw logging on

# 启用防火墙
ufw enable 

# 查看状态
ufw status verbose 
仅允许 CF 访问 Web

如果整站都在 CF 后面，安全起见可以只允许 CF 访问 80/443 端口，可以使用如下脚本配置。脚本会从 https://www.cloudflare.com/ips-v4 和 https://www.cloudflare.com/ips-v6 获取 CF 的 IP 范围，添加到 UFW 规则中并 reload UFW。还会配置每周一零点的定时任务，刷新 IP 段。

1
2
3
4

	
wget -O ~/.cloudflare-ufw.sh https://gist.githubusercontent.com/Xm798/12560579ce11f62027ea8da1fae37456/raw/b07ac8cfe09badf02fc70e2d8bc2da68cabbda50/cloudflare-ufw.sh
chmod +x ~/.cloudflare-ufw.sh
~/.cloudflare-ufw.sh
(crontab -l 2>/dev/null; echo "0 0 * * 1 /root/.cloudflare-ufw.sh > /dev/null 2>&1") | crontab -
用户配置

上面配置完了系统部分，下面切换到用户配置部分。用户名以 cyrus 为例。

1
2

	
# 切换用户（注意是 su - xx，而不是 su xx，su 不会切换环境变量）
su - cyrus
SSH 配置

这里会配置 SSH 密钥登录，并且禁用密码登录和 root 账户登录。

准备公钥

默认已经准备好了 SSH 密钥，如果没有，可以使用 ssh-keygen -t ed25519 生成，并拷贝公钥。

1
2
3
4
5

	
mkdir -p ~/.ssh && chmod 700 ~/.ssh && cd ~/.ssh
# 添加公钥
echo 'ssh-ed25519 XXXXXXXXXXXXXXXXX' > authorized_keys
# 修改权限
chmod 600 ~/.ssh/authorized_keys
关闭密码和 root 登录
1
2
3
4

	
sudo sed -ri "s/^#? *PasswordAuthentication *yes.*/PasswordAuthentication no/" /etc/ssh/sshd_config
sudo sed -ri "s/^#? *PermitRootLogin *yes.*/PermitRootLogin no/" /etc/ssh/sshd_config
# 检查一下配置对不对
cat /etc/ssh/sshd_config | grep -E '^PasswordAuthentication|^PermitRootLogin'
重新加载 SSH 配置
危险
操作前，请务必测试使用对应账户和公钥能够正常登录系统，否则可能导致无法登录的后果！
1
2
3

	
# 测试并重启服务
sudo sshd -t
sudo service sshd restart
ZSH 配置
安装 omz
1

	
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
安装&配置插件

安装 zsh-autosuggestions、zsh-syntax-highlighting 和 autojump：

1
2
3
4
5

	
git clone https://github.com/zsh-users/zsh-autosuggestions.git ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting

sudo apt install -y autojump
echo ". /usr/share/autojump/autojump.sh" >> ~/.zshrc

启用一些插件：

1

	
sed -i.bak 's/plugins=(\(.*\))/plugins=(\1 autojump zsh-autosuggestions zsh-syntax-highlighting z extract sudo cp aliases docker docker-compose)/' ~/.zshrc
配置主题

安装 p10k 主题：

1
2

	
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
sed -i 's#robbyrussell#powerlevel10k/powerlevel10k#g' ~/.zshrc

这里提供一个预先配置好的 p10k 主题，想偷懒的话可运行以下命令：

1

	
wget -O ~/.p10k.zsh https://gist.githubusercontent.com/Xm798/9afccda2e2b26b913854d36c96438344/raw/9acd0a1316895b6bc5485af043d38025f54926bd/.p10k.zsh
应用配置
1

	
source ~/.zshrc

应用完成后会进入 p10k 的配置过程，根据向导设置自己喜欢的主题即可。推荐终端安装并使用 Nerd-Fonts 字体，获得最佳图标显示效果。如果想要重新配置主题，使用 p10k configure 命令。

结束

到此配置已经基本可用了，后续可以进行进一步的配置和服务搭建等。

参考资料
How To Add Swap Space on Ubuntu 18.04 | DigitalOcean
更新于 2024-02-19 
CC BY-NC 4.0
Linux
返回 | 主页
离线安装 rust 并编译 ripgrep
Microsoft E5 订阅禁用数据抢救和服务迁移小记
由 Hugo 强力驱动 | 主题 - FixIt
 2018 - 2024CC BY-NC 4.0
网站运行中……
