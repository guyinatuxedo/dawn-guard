# dnsmasq_OpenSUSE_11
Dnsmasq is a SOHO dns solution, where it allows you to have custom DNS entires (which can be kept in the Host File of the dnsmasq server), but forwards the rest of the dns traffic that it can't resolve to another DNS Server (like 8.8.8.8). This way you can still have custom DNS infrastructure without having to have a full blown DNS Server.

### Installation
To install dnsmasq:
```
$	sudo zypper in dnsmasq
```

To start dnsmasq
```
$	sudo /etc/init.d/dnsmasq start-
```

To enable dnsmasq (run this as root):
```
$	sudo /sbin/insserv /etc/init.d/dnsmasq
```

##### Configure:

Edit this config file:
```
$	sudo vim /etc/dnsmasq.conf
```

Uncomment these lines:
```
#some pointless comment
domain-needed
#another pointless comment
bogus-priv
```
```
strict-order
```

You will have to add your adapter and uncomment this one:
```
interface=eth0
```

Also you will need to add this line to designate what dns server you want to forward to:
```
server=8.8.8.8
```
After that, you should just have to restart the dnsmasq service and you should be good to go.

```
$	sudo /etc/init.d/dnsmasq restart
```

### Network Configuration

You need to configure a static IP address. To do so edit your network adpater config file like this: 

```
$	cat /etc/sysconfig/network/ifcfg-eth0
BOOTPROTO='static'
MTU=''
REMOTE_IPADDR=''
STARTMODE='onboot'
IPADDR=172.16.103.74
NETMASK=255.255.255.0
GATEWAY=172.16.103.1
```

You will also need to add your default router to this file (may have to make the file):

```
$	cat /etc/sysconfig/network/routes
default 172.16.103.2 - -
```

If you don't want to permanetly add the route, you can also just add a tempory route (will get erased with reboots, and network daemon restarts)

Add a default route
```
#	/sbin/route add default gw 172.16.103.2 eth0
```

### Extra

To disable a service from starting automatically:
```
$	sudo /sbin/insserv -r /etc/init.d/dnsmasq
```

##### IPtables
```
$	sudo /usr/sbin/iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     udp  --  anywhere             anywhere            udp dpt:domain ctstate NEW,ESTABLISHED 
ACCEPT     icmp --  anywhere             anywhere            ctstate NEW,ESTABLISHED 

Chain FORWARD (policy DROP)
target     prot opt source               destination         

Chain OUTPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     udp  --  anywhere             anywhere            udp spt:domain ctstate NEW,ESTABLISHED 
ACCEPT     icmp --  anywhere             anywhere            ctstate NEW,ESTABLISHED 
```

To save iptables rules:
```
$	sudo /usr/sbin/iptables-save
```




##### Daemon

If you want to create an OpenSUSE daemon that is really a bash script treated like a daemon, first create a file under `/etc/init.d` like this:

```
$	cat /etc/init.d/gw
#!/bin/bash

### BEGIN INIT INFO
# Provides:	nothing
# Required-Start: $all
# Default-Start: 3 5
# Default-Stop:
# Description: Daemon that pipes text into a file
### END INIT INFO

echo 'tux' > /tmp/guy
```

After that you shoul be able to start it like this
```
$	sudo /etc/init.d/route start
$	cat /tmp/guy
tux
```

Or that you should be able to enable it on startup like this.
```
$	sudo /sbin/insserv /etc/init.d/route
```

After you do that, you should see it appear in the coressponding rcx.d directory that corresponds with it's Start/End level. The Sciptsshould start (and mabey end) at a certain level, which you should be able to find them in here. If they start with a `K`, it's to kill them, and if it's `S`, to run them. The number signigies the order in which they are executed (I believe lower has priority).

```
$	ls /etc/rc.d/rc3.d/ | grep gw
K01gw
S21gw
```