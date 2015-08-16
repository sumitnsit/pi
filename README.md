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