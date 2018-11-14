##	Logging

To view the authentication log.

```
#	cat /var/log/secure
```

View Firewall Logs

```
#	cat /var/log/firewalld
```

####	rsyslog
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
# The authpriv file has restricted access.  
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

#### log rotation

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

#### Log Creation
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
```




####	/var/log/wmtp
These are some commands used to read data from that log file, since it isn't human readable if you just cat it.

To view the last time a user account was logged into.

```
#	last | grep <username>
```

To view all users currently logged on.

```
#	who
```

To view the last reboot

```
#	last reboot
```

To view when an account last logged in

```
#	lastlog 
```