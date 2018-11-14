# Ubuntu 14.04

## Sources List

Here is where the sources list is located:
```
#	vim /etc/apt/sources.list
```

Here is a copy of a functional sources list (was the default list for me):
```
# 

# deb cdrom:[Ubuntu-Server 14.04.5 LTS _Trusty Tahr_ - Release amd64 (20160803)]/ trusty main restricted

#deb cdrom:[Ubuntu-Server 14.04.5 LTS _Trusty Tahr_ - Release amd64 (20160803)]/ trusty main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://us.archive.ubuntu.com/ubuntu/ trusty main restricted
deb-src http://us.archive.ubuntu.com/ubuntu/ trusty main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://us.archive.ubuntu.com/ubuntu/ trusty-updates main restricted
deb-src http://us.archive.ubuntu.com/ubuntu/ trusty-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://us.archive.ubuntu.com/ubuntu/ trusty universe
deb-src http://us.archive.ubuntu.com/ubuntu/ trusty universe
deb http://us.archive.ubuntu.com/ubuntu/ trusty-updates universe
deb-src http://us.archive.ubuntu.com/ubuntu/ trusty-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu 
## team, and may not be under a free licence. Please satisfy yourself as to 
## your rights to use the software. Also, please note that software in 
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://us.archive.ubuntu.com/ubuntu/ trusty multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ trusty multiverse
deb http://us.archive.ubuntu.com/ubuntu/ trusty-updates multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ trusty-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://us.archive.ubuntu.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ trusty-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu trusty-security main restricted
deb-src http://security.ubuntu.com/ubuntu trusty-security main restricted
deb http://security.ubuntu.com/ubuntu trusty-security universe
deb-src http://security.ubuntu.com/ubuntu trusty-security universe
deb http://security.ubuntu.com/ubuntu trusty-security multiverse
deb-src http://security.ubuntu.com/ubuntu trusty-security multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu trusty partner
# deb-src http://archive.canonical.com/ubuntu trusty partner

## Uncomment the following two lines to add software from Ubuntu's
## 'extras' repository.
## This software is not part of Ubuntu, but is offered by third-party
## developers who want to ship their latest software.
# deb http://extras.ubuntu.com/ubuntu trusty main
# deb-src http://extras.ubuntu.com/ubuntu trusty main
```

## Access Control

To view all users:
```
#	cat /etc/shadow | less
```

To view all groups:
```
#	cat /etc/group | less
```

To view the sudoers file:
```
#	sudo vim /etc/sudoers
```

To specify an account can do anything in the `sudoers` file:
```
root    ALL=(ALL:ALL) ALL
```

To specify a group can use sudo in the `sudoers` file:
```
%sudo   ALL=(ALL:ALL) AL
```

To change the password for the user account `guyinatuxedo`:
```
#	passwd guyinatuxedo
```

To change the password for the root account:
```
#	passwd
```

to add a user, and assing that user a password:
```
$	sudo useradd guy
$	sudo passwd guy
```

to add the user `guy` to the group `sudo`:
```
$	sudo usermod -aG sudo guy
```

to remove the user `guy` from the group `sudo`:
```
$	sudo deluser guy sudo
```

to create the group `tuxs`:
```
$	sudo addgroup tuxs
Adding group `tuxs' (GID 1002) ...
Done.
```

to create the group `tuxedos` witht the group ID `1010`:
```
$	sudo groupadd -g 1010 tuxedos
```

to remove the group `tuxedos`:
```
$	sudo groupdel tuxedos
```

## Firewall

Normal IPTables rules do apply for IPv4 and IPv6.

To make firewall rules persistent, first install `iptables-persistent`:
```
$	sudo apt-get install iptables-persistent
```

after that you will need to save your iptables rules for both IPv4 and IPv6 to a conf file:
```
#	iptables-save > /etc/iptables.conf
#	ip6tables-save > /etc/iptables.conf
```

Here is what the files we just generated look like.

IPv4:
```
#	cat /etc/iptables.conf 
# Generated by iptables-save v1.4.21 on Tue Mar 13 18:42:13 2018
*filter
:INPUT DROP [3:234]
:FORWARD DROP [0:0]
:OUTPUT DROP [15:4920]
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 22 -j ACCEPT
COMMIT
# Completed on Tue Mar 13 18:42:13 2018
```
IPv6:
```
#	cat /etc/ip6tables.conf 
# Generated by ip6tables-save v1.4.21 on Tue Mar 13 18:43:54 2018
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT DROP [0:0]
COMMIT
# Completed on Tue Mar 13 18:43:54 2018
```
 
Next we just need to add a command to the `rc.local` file which will essentially import the two conf files we just made whenever the box starts up:

```
#	vim /etc/rc.local
```

Here are the commands that you need to input:

```
iptables-restore < /etc/iptables.conf
ip6tables-restore < /etc/ip6tables.conf
```

## Services

##### Start

This is the directory where services are stored:
```
$	ls /etc/init.d/
```

To start a service:
```
$	sudo service ssh start
```

or

```
$	sudo /etc/init.d/ssh start
```

##### Stop

To stop a service:
```
$	sudo service ssh stop
ssh stop/waiting
```

or

```
$	sudo /etc/init.d/ssh stop
ssh stop/waiting
```

##### Status

To view the status of a service:
```
$	sudo /etc/init.d/ssh status
ssh start/running, process 1034
```

or
```
$	sudo service ssh status
ssh start/running, process 1034
```

##### Boot

to check if the `ssh` service:
```
$	sudo initctl show-config ssh
ssh
  start on runlevel [2345]
  stop on runlevel [!2345]
```

To enable the service `ssh` on boot:
```
$	sudo update-rc.d ssh defaults
```

To disable the service `ssh` on boot:
```
$	sudo update-rc.d -f ssh remove
```

## Logging

This is the default directory for logs to be stored:
```
$	ls /var/log/
alternatives.log  btmp          dmesg.2.gz  fsck       syslog
apt               dist-upgrade  dmesg.3.gz  installer  udev
auth.log          dmesg         dmesg.4.gz  kern.log   unattended-upgrades
boot.log          dmesg.0       dpkg.log    landscape  upstart
bootstrap.log     dmesg.1.gz    faillog     lastlog    wtmp
```

##### Authentication

This is what the user `guyinatuxedo` logging on looks like:
```
$	cat /var/log/auth.log
Mar 13 19:43:05 vangaurd sshd[1326]: Accepted password for guyinatuxedo from 192.168.234.1 port 45460 ssh2
Mar 13 19:43:05 vangaurd sshd[1326]: pam_unix(sshd:session): session opened for user guyinatuxedo by (uid=0)
```
This is what the user `guyinatuxedo` logging off looks like:
```
$	cat /var/log/auth.log
Mar 13 19:43:56 vangaurd sshd[1391]: Received disconnect from 192.168.234.1: 11: disconnected by user
Mar 13 19:43:56 vangaurd sshd[1326]: pam_unix(sshd:session): session closed for user guyinatuxedo
```

##### System

This is what the service `bind9` starting up looks like:
```
$	sudo cat /var/log/syslog
Mar 13 20:25:38 vangaurd named[3523]: command channel listening on 127.0.0.1#953
Mar 13 20:25:38 vangaurd named[3523]: command channel listening on ::1#953
Mar 13 20:25:38 vangaurd named[3523]: managed-keys-zone: loaded serial 2
Mar 13 20:25:38 vangaurd named[3523]: zone 0.in-addr.arpa/IN: loaded serial 1
Mar 13 20:25:38 vangaurd named[3523]: zone 127.in-addr.arpa/IN: loaded serial 1
Mar 13 20:25:38 vangaurd named[3523]: zone 255.in-addr.arpa/IN: loaded serial 1
Mar 13 20:25:38 vangaurd named[3523]: zone localhost/IN: loaded serial 2
Mar 13 20:25:38 vangaurd named[3523]: all zones loaded
Mar 13 20:25:38 vangaurd named[3523]: running
```

This is what the service `bind9` shutting down looks like:
```
$	sudo cat /var/log/syslog
Mar 13 20:20:21 vangaurd named[3435]: received control channel command 'stop -p'
Mar 13 20:20:21 vangaurd named[3435]: shutting down: flushing changes
Mar 13 20:20:21 vangaurd named[3435]: stopping command channel on 127.0.0.1#953
Mar 13 20:20:21 vangaurd named[3435]: stopping command channel on ::1#953
Mar 13 20:20:21 vangaurd named[3435]: no longer listening on ::#53
Mar 13 20:20:21 vangaurd named[3435]: no longer listening on 127.0.0.1#53
Mar 13 20:20:21 vangaurd named[3435]: no longer listening on 192.168.234.171#53
Mar 13 20:20:21 vangaurd named[3435]: exiting
```

## Cronjobs

To edit the crontabs for the user `root` (each user has their own crontabs):
```
$	sudo crontab -u root -e
```

Comment out stuff with `#` character.

## Startup Processes

Commands that are in the `rc.local` file should be automatically run at startup. To see what is in there:

```
$	sudo vim /etc/rc.local
```

## Networking

To configure the network addapters:
```
$   sudo vim /etc/network/interfaces
```
if you want the adapter eth0 configured with dhcp:

```
auto eth0
iface eth0 inet dhcp
```
If you want the adapter eth0 configured with a static ip address with the following information:
```
auto eth0
iface eth0 inet static
        address 192.168.234.172
        gateway 192.168.234.1
        dns-nameservers 192.168.234.2
        netmask 255.255.255.0
        network 192.168.234.0
        broadcasts 192.168.234.255
```
You will need to restart the networking daemon for affects to take change:
```
$   sudo /etc/init.d/networking restart
```
to configure the nameservers, you can just edit the resolv.conf config file:

```
$   sudo vim /etc/resolv.conf
search localdomain
nameserver 192.168.234.2
```
