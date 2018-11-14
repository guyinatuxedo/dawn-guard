# SSH-Server

## Installation:
On Ubuntu 16.04:
```
$	sudo apt-get install openssh-server
```

on rhel:
```
$	sudo yum install openssh-server
```

## Config

To edit the SSH config file, and some of the settings are document below:
```
$	sudo vim /etc/ssh/sshd_config
```

To specify that root cannot login over ssh, add/edit in the following config line:
```
PermitRootLogin no
```

to whitelist users so only the user `guinatuxedo` can log in:
```
AllowUsers guyinatuxedo
```
to blacklist users so only the user `guinatuxedo` can't log in:
```
DenyUsers guyinatuxedo
```

You can disable password authentication, so users can only authenticate using keys, with the follwoing line:
```
PasswordAuthentication no
```

It is possible to tunnel network traffic through ssh, which can obscure what the traffic is which can be helpful for an attacket. To disable ssh forwarding:
```
AllowTCPForwarding no
X11Forwarding no
```

If you want to include a banner with the contents on `/etc/banner.txt`, which will be displayed every time a user tries to authenitcate:
```
Banner /etc/banner.txt
```

If you want to enable verbose logging (so you log a lot):
```
LogLevel verbose
```

You will need to restart the daemon for the config change to take effect:
```
#	systemctl restart sshd
```

## Security

One type of backdooring technique involves planting an ssh public key on the server, then using that to log in. To look for all public ssh keys:
```
$	sudo find / -name "*pub"
```

#### fail2ban

`fail2ban` is a utillity that runs with services like SSH, and helps block attacks by monitoring failed login attempts, and then automatically blocking them using iptables. To install it:

on Debian systems:
```
$	sudo apt-get update
$	sudo apt-get install fail2ban
```

on rhel systems:
```
$	sudo yum install epel-release
$	sudo yum install fail2ban
```

The conf file we will be dealing with is `jail.local`. `fail2ban` comes with a sample config file already, however we have to copy it over:
```
$	sudo cp /etc/fail2ban/jail.com /etc/fail2ban/jail.local
```

the first setting is what ips we will be ignoring (fail2ban will not take any action to block these). For this, we will just have the ip block for our loopback address, since an attacker shouldn't be trying to brute force credentials on that IP ranges (since they won't have access to it, unless they are on the box):
```
ignoreip = 127.0.0.1/8
```

the next line specifies how many seconds a network connection should be baneed, if they are deemed malicious. The default value is 600 seconds, or ten minutes:
```
bantime = 600 # seconds
```

The next two options specifiy the criteria for deeming if a connection is malicious. If a user from an ip address fails to login more times than the value of `maxretry` in the time window given by `findtime`, they are banned: 
```
findtime = 600 # 600 Second window
maxretry = 5 # Number of attemps withing findtime window before network connection is bloxked
```

the next three settings speicfy how `fail2ban` will send email notifications when it bans somebody. The `destemail` option specifies who it will send email to, `sender` specifies who it will send the email as, and `mta` specifies the mail client which it will use to send the email:
```
destemail = root@localhost
sender = root@localhost
mta = sendmail
```

lastly we will need to enable it for ssh. Look for this section of the config file, then add the `enabled` line:
```
[sshd]

enabled = true
port	= ssh
logpath = %(sshd_log)s
backed = %(sshd_backend)s
```

after that, just restart fail2ban and you should be good to go:
```
$	sudo systemctl restart fail2ban
```

## Logging

In Centos, SSH auth logs are store here:
```
#	cat /var/log/secure
```

In Ubutnu, SSH auth logs are store here:
```
#	cat /var/log/auth.log
```

Here is an example of a successful ssh login attempt:
```
Feb 23 00:48:51 localhost sshd[2187]: Connection from 192.168.234.1 port 42450 on 192.168.234.151 port 22
Feb 23 00:48:51 localhost sshd[2187]: Address 192.168.234.1 maps to tux, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!
Feb 23 00:48:56 localhost sshd[2187]: Accepted password for guyinatuxedo from 192.168.234.1 port 42450 ssh2
Feb 23 00:48:57 localhost sshd[2187]: pam_unix(sshd:session): session opened for user guyinatuxedo by (uid=0)
Feb 23 00:48:57 localhost sshd[2187]: User child is on pid 2191
Feb 23 00:48:57 localhost sshd[2191]: Starting session: shell on pts/1 for guyinatuxedo from 192.168.234.1 port 42450 id 0
Feb 23 00:50:03 localhost sshd[2191]: Received disconnect from 192.168.234.1 port 42450:11: disconnected by user
Feb 23 00:50:03 localhost sshd[2191]: Disconnected from 192.168.234.1 port 42450
Feb 23 00:50:03 localhost sshd[2187]: pam_unix(sshd:session): session closed for user guyinatuxedo
```

Here is an example of a failed ssh login attempt:
```
Feb 23 00:50:16 localhost sshd[2213]: Connection from 192.168.234.1 port 42466 on 192.168.234.151 port 22
Feb 23 00:50:16 localhost sshd[2213]: Address 192.168.234.1 maps to tux, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!
Feb 23 00:50:18 localhost unix_chkpwd[2215]: password check failed for user (guyinatuxedo)
Feb 23 00:50:18 localhost sshd[2213]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.234.1  user=guyinatuxedo
Feb 23 00:50:20 localhost sshd[2213]: Failed password for guyinatuxedo from 192.168.234.1 port 42466 ssh2
```

#### IPTables

Here are the rules which will allow an ssh server that listens on port `22` to run with ingress and egress filtering. Essentially block everything excpet for inbound and outbound traffic on `22`:

Iptables rules:
```
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```

and you can just block everything else:
```
iptables -P FORWARD DROP
ip6tables -P FORWARD DROP
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
```
