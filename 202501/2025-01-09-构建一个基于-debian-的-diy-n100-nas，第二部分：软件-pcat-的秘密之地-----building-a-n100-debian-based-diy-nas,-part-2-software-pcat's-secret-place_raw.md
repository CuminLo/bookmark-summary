Title: Building a N100 Debian-based DIY NAS, Part 2: Software

URL Source: https://tongkl.com/building-a-nas-part-2/

Published Time: 2024-09-17

Markdown Content:
[![Image 3: hero image](https://tongkl.com/static/11e26a77cc04e004b70d377e77443968/a91f8/hero-image.png)](https://tongkl.com/static/11e26a77cc04e004b70d377e77443968/ac7a9/hero-image.png)

Andrew Ng correctly recognized as a face by Immich

Welcome back to our NAS-building series! In [part 1](https://tongkl.com/building-a-nas-part-1/), we covered the hardware setup. Now, let's dive into the software configuration to transform our hardware into a fully functional NAS.

OS Installation
---------------

We'll start with a fresh Debian 12 installation. The installation of the OS has been covered in [part 1](https://tongkl.com/building-a-nas-part-1/). Let's just assume we have the OS ready with the 2.5G network interface named `enp2s0`.

Install Necessary Tools
-----------------------

First, let's add the `contrib` components to `sources.list` to enable ZFS installation:

```
# Replace /etc/apt/sources.list with the following content

# deb cdrom:[Debian GNU/Linux 12.6.0 _Bookworm_ - Official amd64 NETINST with firmware 20240629-10:18]/ bookworm contrib main non-free-firmware

deb http://deb.debian.org/debian/ bookworm main contrib non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main contrib non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free-firmware

# bookworm-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html # _updates_and_backports
deb http://deb.debian.org/debian/ bookworm-updates main contrib non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-updates main contrib non-free-firmware

# This system was installed using small removable media
# (e.g. netinst, live or single CD). The matching "deb cdrom"
# entries were disabled at the end of the installation process.
# For information about how to configure apt package sources,
# see the sources.list(5) manual.
```

Now, update and install the tools we'll need. Some of the tools aren't strictly necessary, but I install them for all the machines in my homelab anyway.

```
sudo apt update
apt full-upgrade -y
sudo apt install ca-certificates curl openssh-server vim htop dnsutils iperf3 iptables man zip unzip wget fonts-noto-cjk rsync gnupg2 debian-archive-keyring python3-pip python3-venv python-is-python3 iotop smartmontools hdparm
```

> Tip: If you don't need Asian subtitles in your media, skip fonts-noto-cjk.

Configure HDD Spindown
----------------------

With `hdparm`, we can spin down idle disks after a period of inactivity. Once spun down, a disk uses ~0.5W compared to the ~3W it usually consumes. Ideally we should use the `-B` param to configure the Advanced Power Management value of the disks, but unfortunately my WD Red Plus disks don't support it. So I fell back to the good old `-S` param. I set it to spindown after 20 minutes of inactivity.

Add the following to `/etc/hdparm.conf`:

```
# Note that spindown_time = 240 doesn't mean it's 240 seconds but 1200 seconds.
# Check the man page of hdparm for more details

/dev/disk/by-id/ata-WDC_WD60EFPX-68C5ZN0_WD-XXXXXXXXXXXX {  
    spindown_time = 240
}

/dev/disk/by-id/ata-WDC_WD60EFPX-68C5ZN0_WD-YYYYYYYYYYYY {
    spindown_time = 240
}
```

Reboot the system to apply changes.

> Unfortunately, I never observed the disk spindown myself after setting up Frigate -- there're always activities to the disks!

Network Configuration
---------------------

Now let's move on to networking. First, I set the hostname to `nas`.

```
hostnamectl set-hostname nas
```

Then, I configured the IP addresses of the machine to `192.168.0.3`, `fd00::3`, and a dynamic GUA using SLAAC. I also configured VLAN interfaces for the two IoT VLANs. Change `/etc/network/interfaces` to the following:

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp2s0
iface enp2s0 inet static
  address 192.168.0.3
  netmask 255.255.255.0
  gateway 192.168.0.1
iface enp2s0 inet6 static
  address fd00::3
  netmask 64
  autoconf 1
  accept_ra 2

# IoT IPv4-only VLAN interface
auto enp2s0.3
iface enp2s0.3 inet static
  address 192.168.2.3
  netmask 255.255.255.0
  vlan_raw_device enp2s0

# IoT IPV6-only VLAN interface
auto enp2s0.4
iface enp2s0.4 inet6 static
  address fd10::3
  netmask 64
  vlan_raw_device enp2s0
```

Apply the changes with `sudo systemctl restart networking`.

Then, I set DNS servers to the router. Remember, on Debian we use the old `/etc/resolv.conf` to manually configure DNS servers:

```
# /etc/resolv.conf

nameserver fd00::1
nameserver 192.168.0.1
```

Disable SSH Server Paswword Authentication
------------------------------------------

Add `PasswordAuthentication no` to `/etc/ssh/sshd_config`, then `sudo systemctl restart ssh`.

Install NGINX and Docker
------------------------

I'll skip this part since there are countless tutorials on setting up NGINX and Docker. In a nutshell, I installed the latest version of NGINX, generated Diffie-Hellman parameters, synced SSL certificates to the machine, and brought up the NGINX server.

I added Docker IPv6 support following [this guide](https://docs.docker.com/engine/daemon/ipv6/#use-ipv6-for-the-default-bridge-network), because the qBittorrent container will need IPv6 for much better P2P connectivity.

Install ZFS and Creating a Pool
-------------------------------

Moving on to the main course. Let's install necessary packages:

```
sudo apt install zfsutils-linux linux-headers-amd64 zfs-dkms zfs-zed zfs-auto-snapshot
```

The ZFS kernel module must be enabled manually for the first time:

We are ready to create the ZFS pool. Since I only have a pair of disks, I'd make it a mirror pool:

```
sudo zpool create -o ashift=12 mypool mirror /dev/disk/by-id/ata-WDC_WD60EFPX-68C5ZN0_WD-XXXXXXXXXXXX /dev/disk/by-id/ata-WDC_WD60EFPX-68C5ZN0_WD-YYYYYYYYYYYY
# If creating a RAID10 with multiple disks, use the following command instead
sudo zpool create -o ashift=12 mypool mirror <device1> <device2> mirror <device3> <device4> ...
sudo zfs set compression=lz4 mypool
sudo zfs set xattr=sa mypool
sudo zfs set atime=off mypool
sudo zfs set mountpoint=/pool mypool
sudo chown -R $USER:$USER /pool
```

I'm using `lz4` compression because it's super fast. Since most of the data on my NAS will be movies and photos, which are highly compressed already, another level of compression won't help much. `xattr=sa` allows storing SMB metadata in the inodes, otherwise the metadata will be present as many messy folders and files under the same folders as the original files.

Then, I created some more datasets so I can set individual backup policies of each dataset:

```
sudo zfs create mypool/backup  # Service data backup
sudo zfs create mypool/videos  # For Linux ISOs
sudo zfs create mypool/shared  # NFS folder
# And so on
```

Configure Network Sharing
-------------------------

I chose NFS as the network sharing protocol for now but may switch to SMB later, because I found even Android doesn't work well with NFS. Anyway, here's how to configure NFS sharing.

Install NFS server:

```
sudo apt install nfs-kernel-server
```

Then, edit `/etc/exports`:

```
# /etc/exports

/pool/shared *(rw,sync,no_subtree_check)
# Add other folders you want to share
```

Restart the NFS server:

```
sudo systemctl restart nfs-kernel-server
```

Done! On my Arch laptop I can mount the shared folder using the following command:

```
sudo mount -t nfs fd10::3:/pool /nasshare
# Or alternatively, only mount the shared folder
sudo mount -t nfs fd10::3:/pool/shared /nasshare
```

Install Other Services
----------------------

Skipping this part because you should follow the up-to-date official guides of the services. In summary, here are what I've installed

*   Immich
*   Jellyfin (moved from app server to NAS)
*   Audiobookshelf (moved from app server)
*   Frigate (moved from app server)
*   qBittorrent (moved from app server)

Configure Periodic Snapshots
----------------------------

One of the powerful features of ZFS is painless snapshots. I can take snapshots at a pool or a dataset atomically and almost immediately, so that I can send this snapshot to another place or recover from the snapshot if I fucked up the dataset. It also prevents me from accidentally deleting a dataset when there's at least one linked snapshot unless I intentionally add an `-r` to the `zfs destroy` command.

There are a few tools that take automatic snapshot taking and retention. I decided to use the `zfs-auto-snapshot` package.

The package creates `/etc/cron.daily`, `/etc/cron.hourly`, and a few more folders that contain the corresponding periodic jobs. Simply modify the files in those folders to change the retention policy.

```
$ cat /etc/cron.d/zfs-auto-snapshot  # The 15-min snapshot is defined in cron.d
PATH="/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin"

*/15 * * * * root which zfs-auto-snapshot > /dev/null || exit 0 ; zfs-auto-snapshot --quiet --syslog --label=frequent --keep=192 //

$ cat /etc/cron.weekly/zfs-auto-snapshot
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=weekly --keep=12 //
```

I also have some datasets I don't want to snapshot, such as the downloaded movies. To disable automatic snapshots for specific datasets: `sudo zfs set com.sun:auto-snapshot=false mypool/some/dataset`.

Configure Periodic Scrub and Disk SMART Tests
---------------------------------------------

The NAS disks might fail over time, and I don't want my data to be gone without any prior warnings. We've got two ways to stay ahead of disk failures:

1.  ZFS scrubbing to detect and fix filesystem-level bit rot. It can send a notification if error is detected.
2.  Regular SMART tests to catch disk issues early and send a notification.

Let's set up periodic scrubs and notifications using ZED (ZFS Event Daemon). I've installed `zfs-zed` above alongside other ZFS packages. By default, ZED comes with a handful of event handling scripts that will notify the root user by sending emails. Scrub finishing is one of the events among them. I only need to change the way of notification to Telegram instead.

Create this script at `/usr/local/bin/telegram-notify.sh`:

```
#!/bin/sh

# Replace with your bot token and chat ID
TELEGRAM_BOT_TOKEN="YOUR_BOT_TOKEN"
TELEGRAM_CHAT_ID="YOUR_CHAT_ID"  # If using a channel, add -100 before the actual channel ID. e.g. -10012345678

# Get the message from stdin, making the first line bold.
message=$(cat - | sed '1s/\(.*\)/*\1*/')

if [ -z "$message" ]; then
    # If message is empty, use $3 as the subject and $1 as body
    message="$(cat $1)"
    shift 2
    if [ "$#" -gt 0 ]; then
        message="*$@*
$message"
    fi
fi

# Send message to Telegram
curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
    -d chat_id="$TELEGRAM_CHAT_ID" \
    -d text="$message"
```

The script can parse inputs from both stdin and command line arguments, because I want to re-use the same script later for `smartd` notifications.

Modify `/etc/zfs/zed.d/zed.rc` to use our shiny new Telegram script.

```
# /etc/zfs/zed.d/zed.rc

##
# Notification verbosity.
#   If set to 0, suppress notification if the pool is healthy.
#   If set to 1, send notification regardless of pool health.
#
ZED_NOTIFY_VERBOSE=1  # Change this to 1 if you'd like

##
# Name or path of executable responsible for sending notifications via email;
#   the mail program must be capable of reading a message body from stdin.
# Email will only be sent if ZED_EMAIL_ADDR is defined.
#
ZED_EMAIL_PROG="/usr/local/bin/telegram-notify.sh"  # Change to the script above
```

Restart ZED and we're good to go.

```
sudo systemctl restart zed
```

You can run a scrub task now to see if it sends the notification successfully.

Finally, I scheduled a cron job that scrubs the entire pool at 3:30 AM on the 1st and 16th of every month.

> Note: ZFS automatically schedules a monthly scrub task upon installation in `/etc/cron.d/zfsutils-linux`. Leaving it as-is is usually good enough. I just want to customize it a bit:

```
# /etc/cron.d/zfsutils-linux

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Scrub on day 1 and 16 every month
30 3 1,16 * * root if [ -x /usr/lib/zfs-linux/scrub ]; then /usr/lib/zfs-linux/scrub; fi
```

Cool. ZED done. Enter `smartd`.

You've seen me running `smartctl` commands last time. `smartd` is another program from the same package, `smartmontools`, that provides periodic check ability.

First, create a symbolic link from the telegram notification script we created just now to `smartd` notification folder. `smartd` will execute all the scripts under the folder when something goes wrong.

```
sudo ln -s /usr/local/bin/telegram-notify.sh /etc/smartmontools/run.d/10telegram
```

Then, edit `/etc/smartd.conf` to add the tests we want:

```
# /etc/smartd.conf

# First, comment out this line as it will block all the tests defined after it.

DEVICESCAN -d removable -n standby -m root -M exec /usr/share/smartmontools/smartd-runner  # Comment it!!

# Add these at the end of the file. These should be the only lines that are not commented.

DEFAULT -S on -a -I 194 -I 190 -R 5 -s (S/../../05/03|S/../../12/03|S/../../19/03|S/../../26/03|L/../../08/03|L/../../22/03) -m root -M exec /usr/share/smartmontools/smartd-runner
/dev/disk/by-id/ata-WDC_WD60EFPX-68C5ZN0_WD-XXXXXXXXXXXX  # List all the disks here
/dev/disk/by-id/ata-WDC_WD60EFPX-68C5ZN0_WD-YYYYYYYYYYYY
```

The command is a bit diffucult to read, but you can understand it after reading the comments in the file. Basically, it performs short SMART tests on 5th, 12th, 19th, and 26th of a month, and performs long tests on 8th and 22nd. This way we spread the tests and scrubs (1st and 16th) evenly across the month. The command also rules out some non-critical values, such as the drive temperatures, so I don't get a warning about drive being colder or hotter than last time every few days.

To test it works, we can add a `-M test` to the default command and restart `smartd`:

```
# /etc/smartd.conf
DEFAULT -S on -a -I 194 -I 190 -R 5 -s (S/../../05/03|S/../../12/03|S/../../19/03|S/../../26/03|L/../../08/03|L/../../22/03) -m root  -M exec /usr/share/smartmontools/smartd-runner -M test  # Add `-M test` here
```

```
sudo systemctl restart smartd
```

You should receive test notifications from your Telegram channel now. After verifying it's working, remove the `-M test` directive and restart `smartd`.

Performance Benchmark
---------------------

I did some simple benchmarking using `dd`.

Testing write performance by creating a `mypool/io` dataset, then creating a script to write to a huge file and `time`\-ing it:

```
# Save the following line as test-write.sh
dd if=/dev/urandom of=/pool/io/test bs=1M count=40000 oflag=direct status=progress; sync;

chmod +x test-write.sh
sudo zfs create mypool/io
sudo sync
time sudo ./test-write.sh
```

Test result:

```
41932554240 bytes (42 GB, 39 GiB) copied, 245 s, 171 MB/s
40000+0 records in
40000+0 records out
41943040000 bytes (42 GB, 39 GiB) copied, 245.052 s, 171 MB/s

real    4m12.634s
user    0m0.008s
sys     0m0.019s
```

`dd` reports a write speed of 171 MB/s, but according to `time` the actual write speed is ~158 MB/s. It's rather good, considering I'm writing to a mirror vdev, and ZFS took a snapshot of the dataset in the middle.

Now, testing read performance:

```
# Save the following line as test-read.sh
dd if=/pool/io/test of=/dev/null bs=1M status=progress; sync;

chmod +x test-read.sh
sudo sync
time sudo ./test-read.sh
```

```
41798336512 bytes (42 GB, 39 GiB) copied, 217 s, 193 MB/s
40000+0 records in
40000+0 records out
41943040000 bytes (42 GB, 39 GiB) copied, 217.698 s, 193 MB/s

real    3m37.720s
user    0m0.008s
sys     0m0.029s
```

That's a 183 MB/s according to `time` and `193 MB/s` from `dd`. Lives up to the rated speed of 180 MB/s of the drives. If I add one more vdev in the future, it'll be able to saturate the 2.5 Gbps link.

Finally, don't forget to tear down the dataset.

```
sudo zfs destroy mypool/io
zfs list -H -o mypool/io   -t snapshot | sudo xargs -n1 zfs destroy
```

Conclusion
----------

And that's it, a fully working custom-built NAS running Debian. The good thing is I have full control over the system, unlike prebuilt appliance NAS operating systems such as TrueNAS (which doesn't come with Docker and many have to run Docker inside a VM). There's one important part that I left out though: backup, and I'll describe that [in another post](https://tongkl.com/nas-s3-backup/). Until next time!
