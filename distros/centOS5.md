# Centos 5

## Password Policy



To edit Password Max/Min Age, Minimum Length edit this file

```
#	vim /etc/login.defs
```

Edit these lines

```
PASS_MAX_DAYS 30
PASS_MIN_DAYS 1
PASS_MIN_LEN  10
PASS_WARN_AGE 10
```

To edit password complexity, edit this file

```
#	vim /etc/pam.d/system-auth
```

edit this line

```
#	password requisite pam_cracklib.so try_first_pass
```

and add this

```
retry = 3 type= ucredit=-2 lcredit=-2 dcredit=-2 ocredit=-2
```

## Proxy Configuration on CentOS 5 for yum

```
#	vim /et/yum.conf
```

and add this line to the end

```
#	proxy-http://<proxy ip>:<proxy port>
```

example line

```
#	proxy=http://192.168.7.61:3128
```

## wget

edit this config file

```
#	vim /etc/wgetrc
```

you will need to add these lines

```
#	http_proxy = http://<ip addr>:<port>/
#	https_proxy = https://<ip addr>:<port>/
#	ftp_proxy = http://<ip addr>:<port>/
```

```
#	http_proxy = http://192.168.7.61:3128/
#	https_proxy = https://192.168.7.61:3128/
#	ftp_proxy = http://192.168.7.61:3128/
```

## Configure Users and Accounts:

Add a user:

```
$	sudo adduser <username>
```

Give that user a password:

```
$	sudo passwd <username>
```

Give that user sudo rights (members of the wheel group have sudo rights):

```
$	sudo usermod -aG wheel <username>
```

Remove a user from the wheel (sudo) group:

```
$	sudo gpassword -d <username> wheel
```
## Services

The path to systemctl is /bin/systemctl

View all services (if you want to view a certain type like all running services, use grep)

```
#	systemctl -a -l
```

View a service's status (and dia)

```
#	systemctl status <service_name>
```

start a service

```
#	systemctl start <service_name>
```

stop a service

```
#	systemctl stop <service_name>
```

enable a service (have it automatically start)

```
#	systemctl enable <service_name>
```

disable a service (stop it from automatically starting)

```
#	systemctl disable <service_name>
```

## Logging

To view the authentication logs (users log ins, users' shells changing)

```
#	cat /var/log/secure
```

To view System Logs (services failling, network interfaces going down)

```
#	cat /var/log/messages
```

To view the boot log

```
#	cat /var/log/boot.log
```

To view the mail log

```
#	cat /var/log/maillog
```