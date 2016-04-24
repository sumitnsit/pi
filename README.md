## Create SD card on Mac

```shell
diskutil list
diskutil unmountDisk /dev/disk#
sudo dd bs=1m if=2013-07-26-wheezy-raspbian.img of=/dev/disk#
```

## Create SD card on Ubuntu
```shell
sudo fdisk -l
sudo dd if=/home/2012-10-28-wheezy-raspbian.img of=/dev/sdb
```

## Static IP
```shell
vi /etc/network/interfaces
```
replace entire text with the following

```shell
iface eth0 inet static
address 192.168.1.99
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers 192.168.1.1`
```
## Transmission
```shell
sudo apt-get install transmission transmission-daemon transmission-cli
sudo service transmission-daemon stop
sudo vi /etc/transmission-daemon/settings.json
```
Change following properties in settings.json
```json
"download-dir": "/media/locker/torrents",
"rpc-authentication-required": true,
"rpc-password": "YourPassword_in_english",
"rpc-username": "YourPrefferedUserName",
"rpc-bind-address": "0.0.0.0",
"rpc-whitelist": "*.*.*.*",
```
```shell
sudo service transmission-daemon start
```
## ownCloud
```shell
sudo apt-get update && apt-get upgrade
sudo apt-get install -y git dialog
sudo apt-get install -y nginx php5 php5-curl php5-cgi php5-fpm php5-gd php5-mysql mysql-server
```
```shell
git clone git://github.com/petrockblog/OwncloudPie.git
cd OwncloudPie/
./owncloudpie_setup.sh
```
Select 1st option enter IP address of raspberry pi then select 2nd option.

## Install minildna
```shell
sudo apt-get install minidlna
sudo vi /etc/minidlna.conf
```
Add following line, Make sure that your folders are owned by minidlna
```properties
media_dir=P,/media/locker/Photos
root_container=P
sudo service minidlna force-reload
```

Use a different container as the root of the tree exposed to clients. The possible values are:

``` shell
. to use the standard container (this is the default);
B to use the “Browse Directory” container;
M to use the “Music” container;
V to use the “Video” container;
P to use the “Pictures” container.
```
## Install VPN
```shell
sudo apt-get update
sudo apt-get upgrade
sudo modprobe ppp-compress-18
```
Above command should not display any error
```shell
sudo apt-get install pptpd
sudo vi /etc/pptpd.conf
```
At the end of the file, uncomment the following lines
```shell
localip 192.168.1.99
remoteip 192.168.1.50-238,192.168.1.255
```
And change the "localip" to your raspberry pi ip address
Remoteip = are the addresses that will be handed out to clients.
```shell
sudo vi /etc/ppp/pptpd-options
```
Add the following text at the bottom:
```shell
ms-dns 192.168.1.1
noipx
mtu 1490
mru 1490
```
Where the IP used for the ms-dns directive is the IP address of your router.
```shell
sudo vi /etc/ppp/chap-secrets
```
This is where you will place your credentials for logging into the VPN server add your authentication credentials in the following form:
```shell
username[TAB]*[TAB]password[TAB]*
sudo service pptpd restart
sudo vi /etc/sysctl.conf
```
Enable forwarding if you wish to have access to your entire home network while away.
Find `net.ipv4.ip_forward=1` and uncomment it (or change =0 to =1)
```shell
sudo sysctl -p
```

##Install Java
download java from oracle the following link and move to raspberry pi
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-arm-downloads-2187472.html

```shell
sudo tar zxvf jdk-8-linux-arm-vfp-hflt.tar.gz -C /opt
```

Set default java and javac to the new installed jdk8.

```shell
sudo update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.X/bin/javac 1
sudo update-alternatives --install /usr/bin/java java /opt/jdk1.8.X/bin/java 1
sudo update-alternatives --config javac
sudo update-alternatives --config java
```

After all, verify with the commands with -version option.

```shell
java -version
javac -version
```
## Add Apple OS X HFS+ read/write support
```shell
sudo apt-get install hfsutils hfsprogs hfsutils
```
## Add NTFS rw support
```shell
sudo apt-get install ntfs-3g
sudo mount /dev/sda1 /mnt/usbdrive
```

## Mount External Drive
```shell
sudo vi /etc/fstab
```
Add following line as per your drive and mount point

```shell
/dev/sda1	/media/locker	ext4	defaults,noatime	0	1
sudo reboot
```

## Apple TimeMachine
```shell
sudo apt-get install netatalk
mkdir /media/Locker/TimeMachine
sudo echo "/media/locker/TimeMachine \"Time Machine\" options:tm" >> /etc/netatalk/AppleVolumes.default
sudo service netatalk restart
```
Open TimeMachine on Mac, you should see a shared drive.

## RetroPie-Setup
https://github.com/petrockblog/RetroPie-Setup

## Install VNC server
```shell
sudo apt-get install xorg lxde-core lxde-icon-theme tightvncserver
sudo vi /etc/init.d/vncboot
```
Paste following text

```shell
### BEGIN INIT INFO
# Provides: vncboot
# Required-Start: $remote_fs $syslog
# Required-Stop: $remote_fs $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start VNC Server at boot time
# Description: Start VNC Server at boot time.
### END INIT INFO
#! /bin/sh
# /etc/init.d/vncboot
USER=xbian
HOME=/home/xbian
export USER HOME
case "$1" in
start)
  echo "Starting VNC Server"
  #Insert your favoured settings for a VNC session
  su - $USER -c “/usr/bin/vncserver :1 -geometry 1024x768 -depth 24 -dpi 90”
  su - $USER -c “/usr/bin/vncserver :2 -geometry 1136x640 -depth 24 -dpi 90”
;;
stop)
  echo "Stopping VNC Server"
  /usr/bin/vncserver -kill :0
;;
*)
  echo "Usage: /etc/init.d/vncboot {start|stop}"
  exit 1
;;
esac
exit 0
```
```shell
sudo chmod +x /etc/init.d/vncboot
sudo update-rc.d vncboot defaults
sudo /etc/init.d/vncboot start
sudo reboot
```

## NFS (Network File System)
```shell
sudo apt-get install nfs-kernel-server
sudo vi /etc/exports
/media/sumit/locker/Media/Movies *(rw,all_squash,insecure,no_subtree_check)
sudo update-rc.d rpcbind enable && sudo update-rc.d nfs-common enable
sudo service rpcbind start
sudo service nfs-kernel-server restart
sudo vi /etc/netconfig
```
Comment following two lines (add #)
```shell
#udp6       tpi_clts      v     inet6    udp     -       -
#tcp6       tpi_cots_ord  v     inet6    tcp     -       -
```
## On Mac OS X
```shell
sudo mount_nfs -o resvport pi:/media/locker/nfs /Volumes/NFS/
```

## MP4 to MKV
```shell
avconv -i input.mkv -c copy output.m4v
```

## Disk Operations
### Mount a USB drive
```shell
sudo mount /dev/sda1 /mnt/usbdrive
```
### List your file systems
```shell
sudo fdisk -l
sudo mount -l
df -h
```
### Unmount a USB drive
```shell
sudo umount /dev/sda1
```
You may need to use the -f force option if the drive will not dismount.
```shell
sudo umount -f /dev/sda1
```
### Format a drive
First you must unmount the drive you wish to format.
#### Format a drive to EXT4
```shell
sudo mkfs.ext4 /dev/sda1 -L untitled
```
#### Format a drive to HFS+
```shell
sudo mkfs.hfsplus /dev/sda1 -v untitled
```
#### Format a drive to NTFS
```shell
sudo mkfs.ntfs /dev/sda1 -f -v -I -L untitled

-f Fast Format. Due to the poor performance of 3g.ntfs on the Pi I highly recommend using the less CPU intensive fast format mode.
-v Verbose. By default the NTFS status output is limited so this lets you know what is happening.
-I Disable Windows Indexing. This improves the write performance of the drive but it will mean Windows Search queries used on this drive will take longer.
```

#### Add Windows/DOS FAT32 read/write support
```shell
sudo apt-get install dosfstools
```
#### Format a drive to FAT32
```shell
sudo mkfs.vfat /dev/sda1 -n untitled
```

#### Automatically Mount A Drive
```shell
sudo vi /etc/fstab
```
Add the following to the bottom of the file.
```shell
/dev/sda1 /mnt/usbdisk auto defaults,user 0 1
/dev/sda1 is the location of the drive to mount.
/mnt/usbdisk is the mount point, which is the folder to access the content of the drive.

#auto Is the file system type, here you can set ‘auto‘ or force a file system type such as ext2, ext3, ext4, hfsplus, ntfs, vfat.
#defaults,user Are mount options. You normally need to only supply ‘defaults‘. Though there are some others that may be useful such as ‘ro‘ for read-only or ‘user‘ to enable write permission for all users. Use a non-spaced comma to separate multiple options.
# 0 A binary value used for debugging. It is best to keep this set at zero.
# 1 Pass number for a filesystem check at boot. ‘0‘ (zero) to disable or ‘2‘ to enable.
```
