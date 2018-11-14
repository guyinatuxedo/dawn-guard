# Rouncube on Fedora 20 using Postfix, Dovecot, Nginx, MySQL, PHP-FPM

### Postfix Installation

First install postfix:
```
#	yum install postfix dovecot vim
```

Next edit this file `/etc/postfix/main.cf`:
```
myhostname = localhost.localdomain
mydomain = dinosaur.lan
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain 
mynetworks = 172.16.103.0/24, 127.0.0.0/8
```

Now you will want to add users to send mail to. These users will not be able to log in, or have a home directory, and will be a member of the mail group (will be useful later):
```
#	useradd -M purple
#	usermod -a -G mail purple
#	passwd purple
```

Now restart and enable postfix:
```
#	systemctl restart postfix
#	systemctl enable postfix
```

Now you should be able to send a test email using `Mail`:
```
#	hostname
localhost.localdomain
#	mail	purple@localhost.localdomain
Subject: I'm going to greap you
IN THE MOUTH!!!!
.
EOT
#	cat /var/mail/purple
From root@dinosaur.lan  Thu Jun 15 21:10:10 2017
Return-Path: <root@dinosaur.lan>
X-Original-To: purple@localhost.localdomain
Delivered-To: purple@localhost.localdomain
Received: by localhost.localdomain (Postfix, from userid 0)
	id 6E0E640697; Thu, 15 Jun 2017 21:10:10 -0700 (PDT)
Date: Thu, 15 Jun 2017 21:10:10 -0700
To: purple@localhost.localdomain
Subject: I'm going to grape you
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Message-Id: <20170616041010.6E0E640697@localhost.localdomain>
From: root@dinosaur.lan (root)

IN THE MOUTH!!!!

```

If that works, then Postfix is working successfully. Now onto DNS.

### DNS Records
In order to effectively use your email server, you will need to configure DNS records for it (an MX and an A record)

If you are using PowerDNS, here are the DNS records you need to insert:

```
mysql> insert into records (domain_id, name, type, content, ttl, prio) values (1, "mail.hack.lan", "A", "192.168.6.102", 120, NULL);
mysql> insert into records (domain_id, name, type, content, ttl, prio) values (1, "hack.lan", "MX", "mail.hack.lan", 86400, 10);
```

If you are using Bind, here are the DNS records you need to have in your zone file: 

```
mail.hack.lan.			IN	A		192.168.6.102
hack.lan.				IN	MX	10	mail.hack.lan.
```

After you configure the dns records, you should be able to do the same test as before, just replace the name of `localhost` with the domain name:
```
#	mail purple@dinosaur.lan
```

### RoundCube Setup
##### Downloading

First add this repo to `/etc/yum.repos.d/fedora.repo` under `[fedora]`:
```
baseurl=http://archives.fedoraproject.org/pub/archive/fedora/linux/releases/20/Everything/x86_64/os/
```
Then install Nginx, php, php extensions, and php-fpm
```
#	yum install ngninx 
#	yum install php php-fpm 
#	yum install php-common php-pdo php-mbstring php-mysql php-mcrypt php-intl php-pear
#	systemctl enable nginx
#	systemctl start nginx
#	systemctl enable php-fpn
#	systemctl restart php-fpm
```

Next download roundcube


```
#	cd /tmp
#	wget wget wget https://github.com/roundcube/roundcubemail/releases/download/1.2.5/roundcubemail-1.2.5.tar.gz
#	tar zxvf roundcubemail-1.2.5.tar.gz
#	cp -avr roundcubemail-1.2.5/* /var/www/html/
#	chown -R nginx:nginx /var/www/html/
```

##### PHP

Edit this file:
```
$	vi /etc/php.ini
```

make sure this setting is set to 0 (it allows attackers to access php files they don't have the full name for):

```
cgi.fix_pathinfo=0
```

Next edit the main Nginx conf file:
```
$	vi /etc/nginx/nginx.comf
```

Add this location inbetween the server brackets, but not in another location:
```
location ~ \.php$ {
                root    /var/www/html;
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME   $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
```
Next edit the `/etc/php-fpm.d/www.conf` file and make sure these settings are set to nginx:
```
user = nginx
group = nginx
```

Start and enable the service
```
#	systemctl enable php-fpm
#	systemctl start php-fpm
```

##### RoundCube Installation

To install 

```
#	pear upgrade --force --alldeps http://pear.php.net/get/PEAR-1.10.1
```

```
#	pear install  Auth_SASL Net_SMTP Net_IDNA2-0.1.1 Mail_mime Mail_mimeDecode
```

After that, restart nginx and php-fpm:
```
#	systemctl restart nginx
#	systemctl restart php-fpm
```

Now PHP should be working:

##### MySQL:

Install MySQL:
```
#	yum install mysql mysql-server
```

Start and enable MySQL:
```
#	systemctl start mysql
#	systemctl enable mariadb
```

do a secure installation:
```
#	mysql_secure_installation
```

now log on to mysql:
```
#	mysql -u root -p
```

create the user, and the db which roundcube will use:

```
MariaDB [(none)]> create database mail;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user 'mail'@'localhost' identified by 'mail';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all on mail.* to 'mail'@'localhost';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

##### Dovecot:

To enable error logging to `/var/log/maillog` and rsyslog:
```
#	echo "log=path = syslog" >> /etc/dovecot/conf.d/10-logging.conf
```

First add this line signifying to use imap to the `dovecot.conf` file:

```
#	echo "protocols = imap" >> /etc/dovecot/dovecot.conf
```

Next you are going to want to edit this file:
```
#	vim /etc/dovecot/conf.d/10-master.conf
```

and uncomment/add these lines:
```
service imap-login {
	inet_listener imap {
		port = 143
}
```

##### Creating mail users
To creat the user `purple` which can be used by the mail configuration.

```
#	useradd -M purple
#	usermod -a -G mail purple
#	passwd purple
```

##### Dovecot part 2

Next you are going to want to add this line to the bottom of the `/etc/dovecot/conf.d/10-mail.conf` file:
```
mail_location = mbox:/var.mail/mailbox/%u:INBOX=/var/mail/%u
```

Now configure the permssions of the `/var/mail` directory:
```
#	mkdir /var/mail/mailbox
#	chgrp mail /var/mail/*
#	chmod -R 670 /var/mail 
#	rm -rf /var/mail/*
#	mkdir /var/mail/mailbox/
#	chmod -R 670 /var/mail/mailbox/
#	chgrp mail /var/mail/mailbox
``` 

First add this line signifying to use imap to the `dovecot.conf` file:

```
#	echo "protocols = imap" >> /etc/dovecot/dovecot.conf
```

Next you are going to want to edit this file:
```
#	vim /etc/dovecot/conf.d/10-master.conf
```

and uncomment/add these lines:
```
service imap-login {
	inet_listener imap {
		port = 143
}
```

Next you are going to want to add this line to the bottom of the `/etc/dovecot/conf.d/10-mail.conf` file:
```
mail_location = mbox:/var.mail/mailbox/%u:INBOX=/var/mail/%u
```

### Mail Security

##### Config

To configure if it is an open relay, edit this file:
```
#	vim /etc/postfix/main.cf
```

Add this to the bottom (if there are other ip's in there, it can act as a relay to them):
```
mynetworks = 127.0.0.0/8
```

also check if there are any lines that have this (this can enable mail relays):
```
smtpd_recipient_restrictions = 
```

##### IPTables

For input:
```
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 25 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 9000 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 9000 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 143 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 143 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 3306 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 3306 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 25 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 25 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
```

output:
```
iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 25 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 9000  -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 9000 -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 143 -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 143 -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 3306 -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 3306 -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 25 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 25 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT
```

### WebApp Security
##### Nginx
Edit this file:
```
#	vi /etc/nginx/nginx.conf
```

and add this inbetween the `server` brackets to disbale showing the version number:
```
server_tokens off;
```

In between the `server` brackets uncomment/add this line:
```
#		access_log	/var/log/nginx/host.access.log	main;
```

add this inbetween the `server` brackets to help secure against buffer overflow attacks:
```
client_body_buffer_size	1k;
client_header_buffer_size	1k;
client_max_body_size	1k;
large_client_header_buffers	2	1k;
```

add this inbetween the `server` brackets to disable

add this inbetween the server brackets to disable other types of data submission (other thean GET, POST, and HEAD) from being used:
```
if ($request_method !~ ^(GET|HEAD|POST)$ )
{
	return 444;
}
```

then restart the nginx service:
```
#	systemctl restart nginx
```

##### PHP

edit this file:
```
#	vi /etc/php.ini
```

Add/Edit this settings

```
display_errors 0
```

```
max_execution_time = 30
```

```
max_input_time = 60
```

```
allow_url_fopen = Off
```

```
allow_url_include = Off
```

```
sql.safe_mode = On
```

```
memory_limit = 128M
```

These settings will disable file uploading, so if you need it for the service , configure the filesize to be `2M` and uploads to be `1`:

```
file_uploads = Off
```

```
upload_max_filesize = 0M
```

```
max_file_uploads = 0
```






### Networking

##### IPtables

To get your network to a functioning state:
```
#	yum remove firewalld
#	iptables -F
#	systemctl restart iptables
#	reboot
```

Now we will need to configure the nginx conf file `/etc/nginx/nginx.conf`:
```
#	vi /etc/nginx/nginx.conf
```

make sure to start and enable the dovecot service:
```
#	systemctl start dovecot
#	systemctl enable dovecot
```

##### Static IP Addressing

Add these lines:
```
IPADDR=172.16.103.155
GATEWAY=172.16.103.2
NETMASK=255.255.255.0
PEERDNS=no
```

Also make sure to add your nameservers:
```
#	cat /etc/resolv.conf
nameserver 172.16.103.154
```

Reboot to make it static.










