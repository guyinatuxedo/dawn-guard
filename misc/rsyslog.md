#	rsyslog & logrotate CentOS 7
rsyslog is the daemon that is responsible for logging. A couple of things about rsyslog, it does not generate the log messages. It simply listens to utillities such as cron, and records their messages. Since it is a daemon, after you make any changes to the config file you have to restart rsyslog for it to take effect like this. 

```
#	systemctl restart rsyslog
```

To view the manpage
```
#	man rsyslog.conf
```

To edit the rsyslog.conf file

```
#	vim /etc/rsyslog.conf
```

To change the logging file for authentication to "/var/log/knightauth" change

```
# The authpriv file has res#dnsmasq_OpenSUSE_11
Dnsmasq is a SOHO dns solution, where it allows you to have custom DNS entires (which can be kept in the Host File of the dnsmasq server), but forwards the rest of the dns traffic that it can't resolve to another DNS Server (like 8.8.8.8). This way you can still have custom DNS infrastructure without having to have a full blown DNS Server.

###Installation
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

#####Configure:

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

###Network Configuration

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

###Extra

To disable a service from starting automatically:
```
$	sudo /sbin/insserv -r /etc/init.d/dnsmasq
```

#####IPtables
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




#####Daemon

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
```tricted access.  
authpriv.*                                              /var/log/secure
```

to

```
# The authpriv file has restricted access.  
authpriv.*                                              /var/log/knightauth
```

Now rsyslog can take different types of logs, which each have a different priority level. Here is the list in descending priority level.

```
emerg:	Emergency

alert:	Requires immediate intervention

crit:	A critical condition

err:	An Error

warn:	A warning

notice:	A condition that might require attention

info:	just an information message, no attention required

debug:	Debug information
```

Now you can configure rsyslog to only log specific messages in the rsyslog.conf file. This is specified by whatever is after the dot on the line in rsyslog.conf. For instance if you wanted to log only warning messages or greater for cron, you could change this

```
# Log cron stuff  
cron.*                                                  /var/log/cron  
``` 

to this

```
# Log cron stuff  
cron.warn                                             /var/log/cron.warn  
```

Now if you only want to log a specific priority such as alert and nothing else for boot, change this

```
# Save boot messages also to boot.log  
local7.*                                                /var/log/boot.log  
```

to this

```
# Save boot messages also to boot.log  
local7.=alert                                        /var/log/boot.log.alert
```

Now if you want, you can specify that you want to log everything except for a specific level (let's say notice). To do that for news change this

```
uucp,news.crit                                          /var/log/spooler  
```

to this (keep in mind, that you don't need the equal sign)

```
uucp,news.=!notice                                          /var/log/spooler.notice 
```

## log rotation

Log rotation is the process of creating new log files, backing up old ones, then evantually removing them. By default, the rotated logs are stored in the same directory as the logs "/var.log". There is a manpage for lograotare.

```
#	man logrotate
```

To edit the logrotare config file.

```
#	vim /etc/logrotate.conf
```

Now in the log file, you should see a line like this.

```
include /etc/lograte.d
```

Now that line means that we include logrotate config files in the "/etc/logrotate.d" direcorty. These should specify that certain log files are to be rotated. I'm now going to go over some of the common settings for a logrotate file.

```
#	cat /etc/logrotate.d/yum
/var/log/yum.log {
	rotate 3
	missingok
	notifempty
	compress
	size 30k
	weekly
	postrotate
		echo "Yum logs have been rotated" > /tmp/yumlog
	endscript
	create 0600 root root
}
```

This specifies the number of logs it will keep, before either removing them or mailing them. In this case, it will only keep 3 logs (and will remove the oldest when a 4th log is rotated).

```
	rotate 3
```

This line specifies that if this log file is missing, logrotate should not generate an error and proceed to the next file.

```
	missingok
```

This line specifies that if the log file is empty, do not rotate the log. This overrides the ifempty option.

```
	notifempty
```

This will compress log files that have been rotated out with gzip.

```
	compress
```

This specifies that the logs should only be rotate if the grow bigger than 30k. It would also take 30M or 30G as acceptable sizes for 30 Megabytes or 30 Gigabytes

```
	size 30k
```

This line specifies that the logs should be rotated weekly.

```
	weekly
```

This specifies that the command "echo "Yum logs have been rotated" > /tmp/yumlog" should be run using /bin/sh after this log has been rotated. You could also use prerotate/endscript to run a bash command before a log is rotataed, but only if it will actually be rotated.

```
    postrotate
        echo "Yum logs have been rotated" > /tmp/yumlog
    endscript
```

This specifies that the log file after being rotated, is supposed to be created again with root being it's owner and group, and with the permission 0600 (only the user can read and write, and it's a normal file).

```
    create 0600 root root
```

and if you want to manually rotate all of your logs

```
#	logrotate -fv /etc/logrotate.conf
```

## Log Creation
This will be reviewing how to create your own logs using local4, which is a utillity that can be used to generate system messages. First we need to edit "/etc/rsyslog.conf" and add these two lines so rsyslog will log it.

```
#Log only emergencies
local4.emerg	/var/log/local4emerg.log
#Log only errors
local4.=err	/var/log/local4err.log
```

Next we need to restart the rsyslog daemon to apply the changes.

```
#	systemctl restart rsyslog
```

Now we need to generate the actual system messages using local4.

```
#	logger -p local4.err "Error: You are getting 4channed harder than pepe on /b/"
#	logger -p local4.emerg "Emergency: You are getting 4channed harder than pepe on /b/"
```

Now if we look in "/var/log", we should see the log files, and the messages we just generated.

```
#	ls -asl /var/log | grep local4
  4 -rw-------.	1 root	root	198 May 21 16:57 local4emerg.log
  4 -rw-------.	1 root	root	190 May 21 16:57 local4err.log
 #	cat /var/log/local4emerg.log
 May 21 16:57:56	localhost	guyinatuxedo:	Emergency:	You are getting 4changed harder than pepe on /b/
 #	cat /var/log/local4err.log
  May 21 16:57:58	localhost	guyinatuxedo:	Error:	You are getting 4changed harder than pepe on /b/
```

##Config Info

Here is a list of some of the things that rsyslog can log.

```
auth or authpriv:	These are messages regarding authentication or security

kern:				Messages relating to the kernel

mai:				Messages relating to the mail system

cron:				Messages from crontabs

daemon:				Messages relating to daemons (messages for centos, syslog for debian)

news:			 	Logging from network news subsystem (NNTP)

lpr:				printing logs

uucp:				log informationg from Unix to Unix Copy Program, which is an old e-mail distribution protocol.

user: 				Generic user messages

local0 to local7:	This is logging reserved for local purposes.
```

Now here is an example default rsyslog config file.

```

```