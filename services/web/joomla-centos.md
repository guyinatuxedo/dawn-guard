# Joomla

This is a guide for Joomla 3.8.5 on CentOS 7. This uses Nginx, PHP-FPM. and MySQL to support the web app.

## Installation

In order to do this, we will need to install LEMP (Linux Nginx MySQL PHP):


#### Nginx

First we will need to install nginx. To do that:

```
#	yum install epel-release
#	yum install nginx
```

Next we will need to edit the ngninx console to setup passing PHP scripts to php-fpm:

```
#	vim /etc/nginx/nginx.conf
```

add/edit these settings in to reflect your needed settings in the `server` block:

```
server {
    index index.php;
#Other Server configs
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
#Other Server configs
}
```

Next we can start and enable the nginx service:

```
#	systemctl start nginx
#	systemctl enable nginx
```

#### PHP

First we need to install php-fpm, and php-mysql so php can interface with the mysql database: 

```
#	yum install php-fpm php-mysqlnd
```

After installing it, there are a few settings that we need to change which can be found in this config file:

```
#	vim /etc/php-fpm.d/www-conf
```

First check to see what it is listening on. This setting and the `fastcgi_pass` setting from the nginx installation part should be the same, so nginx and php-fpm can talk to each other:

```
listen = 127.0.0.1:9000
```

You will see these settings, which specify what user the `php-fpm` worker process will be running as.

```
user = apache
group = apache
```

They should be changed to this, so they have the same permissions as nginx:

```
;user = apache
;group = apache
user = nginx
group = nginx
```

In addition to that, you will need to edit the `php.ini` file:
```
#	vim /etc/php.ini
```

You will need to add/edit in this setting for the session save path:

```
session.save_path = "/var/lib/php/session"
```

Next we will need to create the directory `/var/lib/php/session` and change the owner to `nginx`:
```
#	mkdir /var/lib/php/session
#	chown nginx:nginx -R /var/lib/php/session/
```

After that, we will be good to start and enable the php-fpm service:

```
#	systemctl start php-fpm
#	systemctl enable php-fpm
```
#### Mysql

Next we will need to install mariadb-server, which can be done with this command:

```
#	yum install mariadb-server
```

Next you can start and enable the mariadb service:

```
#	systemctl start mariadb
#	systemctl enable mariadb
```

Next you should be able to log into the mysql console as root without a password (since there hasn't been one set yet):

```
#	mysql -u root
```

You will probably want to change the root password right off the bat (I will be changing it to `5up3r_53cur3_p@55w0rd`)

```
MariaDB [(none)]> update mysql.user set password = password('5up3r_53cur3_p@55w0rd') where user = 'root';
Query OK, 4 rows affected (0.00 sec)
Rows matched: 4  Changed: 4  Warnings: 0

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
```

Next it should require you to use the password you just set in order to log into mysql as root:

```
#	mysql -u root -p
```

Now that we got that taken care of, we should be able to just create a new database for joomla to use (called 'black_flame') and a user which has all privileges to that database (called `'dragonslayer'@'127.0.0.1'` so it will only be able to log in from `127.0.0.1`, with the password `5up3r-53cur3-p@55w0rd`)

```
MariaDB [(none)]> create database black_flame;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user 'dragonslayer'@'127.0.0.1' identified by "5up3r-53cur3-p@55w0rd";
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all privileges on black_flame.* to 'dragonslayer'@'127.0.0.1';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```



#### Joomla

First we are going to need `wget` and `unzip`

```
#	yum install wget unzip
```

Next we can download the web app, and extract it to the web root for nginx. The current latest version at the time of the web app is `3.8.5`, change that as need be:

```
#	cd /tmp/
#	wget https://github.com/joomla/joomla-cms/releases/download/3.8.5/Joomla_3.8.5-Stable-Full_Package.zip
#	unzip Joomla_3.8.5-Stable-Full_Package.zip -d /usr/share/nginx/html/
```

We will need to change the ownership of the web app to the user `nginx`, so they can install the web app:

```
#	chown nginx:nginx -R /usr/share/nginx/html/
```

By default CentOS has firewall rules which will block http access to the web app. You can flush the rules, and set your own later:

```
#	iptables -F
```

Before you finish with the installation, you will need to change these selinux bools to allow nginx to connect to a database and can write to it's config files:
```
#	setsebool -P httpd_can_network_connect_db 1
#	setsebool -P httpd_unified 1
```

if that doesn't work, you might want to consider allowing nginx to connect to the network through SELinux:
```
#	setsebool -P httpd_can_network_connect 1
```

After that, you should be able to go to the url in a web browser and finish the installation:

```
http://http://192.168.234.167/
```
*	Site Configuration:	Input information about the site, and admin credentials
*	Database:	Input MySQL DB credential information (and check to ensure connectivity)
*	FTP: Input FTP Credentials (Optional)
*	Overview: Review installation options, and maybe select a few others

After that, the web app may ask you to manually create the `configuration.php` file. Just create the file:
```
#	vim /usr/share/nginx/htmlconfiguration.php
```

and copy and past in the configs it asks you (I hope you are using SSH). After that you can just remove the `installation` folder, and you should be good to go:


```
#	rm -rf /usr/share/nginx/html/installation/
```

#### Installation Errors

One issue that might come up durring installation (durring the MySQL step) it might ask you to create this file:

```
#	touch /usr/share/nginx/html/installation/_JoomlaCn6zUdTJ8fxSYI0yTcmLc.txt
```

Also if you are stuck on the MySQL step, for debug purposes you might want to try disabling SELinux (do not leave it disabled, once you confirmed that it is the issue properly configure it to work):
```
#	setenforce 0
```

## Config
##### Error Reporting

Error reporting will essentially have php output it's errors, if it encounters one. This can be pretty helpful for debugging the web app, however should be turned off when not being used. To enable it, edit this file:
```
#	vim /usr/share/nginx/html/configuration.php
```

add/edit in the following settings:


```
public $error_reporting = 'E_ALL';
```

##### MySQL

If you need to change the credentials which Joomla will use to log into the MySQL db, you can do it by editing this file: 

```
#	vim /usr/share/nginx/html/index.php
```

add/edit in the following settings to match the settings you need:

```
        public $dbtype = 'mysqli';
        public $host = '127.0.0.1';
        public $user = 'dragonslayer';
        public $password = '5up3r_53cr35_p@55w0rd';
        public $db = 'black_flame';
        public $dbprefix = 'tf48r_';
```

## Administration

The IP address of the Joomla instance is `192.168.234.163`

there should be a folder titled `administrator` in the web directory for Joomla. If you go there in a web browser, you can log in to the admin panel:
```
http:192.168.234.163/administrator/
```

There should be a menu on the lower left hand side of the string. Here are some things that might be of interest:

*	Modules:	Menu > Structure > Modules (Default Modules `Breadcrumbs `, `
Main Menu `, and `Login Form `)
*	Extensions:	Menu > Extensions
*	Users > Menu > Users > Users


## Security

For this service, there are four different things which need to be secured:

*	PHP-FPM
*	Nginx
*	Joomla 
*	MySQL

#### PHP-FPM

*	Specify what clients can run code
*	Privileges of the service
*	Where is it listening
*	Base Directory
*	Blacklist php functions
*	Remote FIle Inclusion
*	Disable File Uploads
*	Don't expose PHP
*	Limit Post Requests Size
*	Check MySQLi Confs
*	Check resources (memory usage)
*	Configure Logging


##### Clients

To specify what IP addresses `php-fpm` will take code from,e dit this config file:
```
#	vim /etc/php-fpm.d/www.conf
```

add/edit in this setting, it should only have the ip addresses which need this functionallity (try to keey it to `127.0.0.1`):

```
listen.allowed_clients = 127.0.0.1
```

##### Privileges

You can sepcify the permissions which php-fpm will run with (and the socket) by changing the user/group which it runs as. To do that edit this file:

```
#	vim /etc/php-fpm.d/www.conf
```

to configure the user/group which php-fpm will run as:

```
user = nginx
group = nginx
```



##### Listening

Make sure that php is listening on a socket that will expose it the least (so use a socket that isn't binded to an IP address, or use once that is binded to 127.0.0.1 if you need to, (unless if the php-fpm and nginx servers are on different boxes):

```
#	vim /etc/php-fpm.d/www.conf
```

to have it talk over a socket that isn't binded to an IP address:

```
listen = /run/php-fpm/www.sock
```

to have it talk over a socket binded to the ip address and port 127.0.0.1:9000:

```
listen = 127.0.0.1:9000
```


To sepcify the user/group which the unix socket will run with:
```
listen.owner = nginx
listen.group = nginx
```

To sepcify an access control list user/group which the unix socket will run with. This will override the above listen.owner and listen.group setiings:

```
listen.acl_users = nginx
listen.acl_groups = nginx
```

##### Base Directory

You can restrict what php code can execute to that of a single directory. To do that edit this directory:

```
#	vim /etc/php.ini
```

add/edit in this setting (it should be that of the web root of the webapp, so php code outside of the web directory can't run):

```
open_basedir = /usr/share/nginx/html/
```

#####	Blacklist php functions

There are a lot of php functions, which you reallydon't need to have, which can be of great use to an attacker (so you should probably blacklist them). To do that, edit this file:

```
#	vim /etc/php.ini
```

add/edit in this setting:
```
disable_functions = exec, passthru, shell_exec, system, proc_open, popen, curl, exec, curl_multi_exec, parse_ini_file, show_source, pcntrl_exec, eval, assert, create_function, include, include_one, require, require_once, phpinfo,  pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
```

This function if disable will break the web app:

*	`preg_replace()`:	Will stop the web app from running if diabled

#####	Remote FIle Inclusion

You should disable allow_url_fopen and allow_url_include, since they can be used by an attacker to access files on the server, which they can use in exploits (Remote File Inclusion). To do that, edit this file:

```
#	vim /etc/php.ini
```

add/edit in the following settings

```
allow_url_fopen = Off
allow_url_include = Off
```

#####	Disable File Uploads

File uploads really are not needed for our competition, so they should be disabled (which will disable an attacker from uploading things such as web shell). To do that, edit this file:
```
#	vim /etc/php.ini
```

add/edit in these settings:
```
file_uploads = Off
upload_max_filesize = 0M
max_file_uploads = 0
```

#####	Don't expose PHP

PHP can expose that it is installed on a server, which will be potentially helpful to an attacker. To disable it edit this conf file:

```
#	vim /etc/php.ini
```

add/edit this in:

```
expose_php = Off
```

#####	Limit Post Requests Size

Depending on what you are doing, you should probably limit the size of post requests which PHP will take, since this will help prevent certain type of DOS attacks. To do that edit this conf file:

```
#	vim /etc/php.ini
```

add/edit this in:

```
post_max_size = 512K
```

#####	Check MySQLi Confs

You should probably take a second and look over the conf files for mysqli (or whatever php mysql library you are using). Here are the important lines to check:

If these are set to valid credentials, then red team just needs to call the mysqli connect function to pivot to the db. To view these settings, check this conf file:
```
#	vim /etc/php.ini
```

here are some of the important settings you should check:

```
mysqli.default_user =
```

```
mysqli.default_host =
```

```
mysqli.default_pw =
```

Should probably check the port/socket mysqli is using:

```
mysqli.default_port = 3306
```
```
mysqli.default_socket =
```

#####	Check resources (memory usage)

By default, a php script can consume up to 128 Megabytes, which for our needs is too much. This can be descreased to 40M, in order to help prevent DOS attacks/uneeded stress. To do that, edit this config file:

```
#	vim /etc/php.ini
```

add/edit this setting in:

```
memory_limit = 40M
```

#####	Configure Logging

Make sure to enable logging. To do that, check this config file:

```
#	vim /etc/php.ini
``` 

Here are the settings you should check

```
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
```

```
log_errors = On
```

also check to ensure that the logging location is acurate (should probably be commented out by default):

```
; error_log = syslog
```


#### Nginx


*    Disable Server Tokens
*    Disable unnecissary HTTP methods
*    Clickjacking Prevention
*    XSS Protection
*    SSL

##### Disable Server Tokens

Server tokens will display information about the web server, including the version umber (whcih can be helpful for picking which metasploit module to use). Edit the nginx conf file:

```
#	vim /etc/nginx/nginx.conf
```

add/edit in this setting in the `server` blocks that you want to protect:

```
server_tokens off;
```

##### Disable unnecissary HTTP methods

For the most part, the only HTTP methods you need are GET, HEAD, and POST. So you should whitelist these three methods and disable all others, since they could be fairly helpful for an attacker (such as TRACE, DELETE, PUT, and OPTIONS). To do that, add in this setting to server blocks you wish to protect. Edit the main nginx conf file:

```
#	vim /etc/nginx/nginx.conf
```

add/edit in these settings in the `server` blocks that you want to protect:

```
        if ($request_method !~ ^(GET|HEAD|POST)$)
        {
                return 405;
        }
```

##### Clickjacking Prevention

Nginx can defend against clickjacking, by specifing that browsers can only load resources from the same origin, by adding the following config to the server block to the nginx conf file `/etc/nginx/nginx.conf`:

```
add_header X-Frame-Options "SAMEORIGIN";
```

##### XSS Protection

Nginx has some XSS protection (protection against Cross Site Scripting). To enable it, edit this config file:
```
#	vim /etc/nginx/nginx.conf
```

add/edit this setting into the `server` blocks that you want to protect:
```
add_header X-XSS-Protection "1; mode=block";
```
##### SSL

SSL will essentially encrypt all traffic coming to and from the clients. To enable this, first generate a public cert and private key:

```
#	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/certs/nginx.key -out /etc/ssl/certs/nginx.crt
```

Then generate a Diffie-Hellman group:

```
#	openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Next create a ssl conf for nginx:

```
#	vim /etc/nginx/conf.d/ssl.conf
```

It should have these values:

```
ssl_protocols TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_ecdh_curve secp384r1;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header Strict-Transport-Security "max-age=6307200; includeSubdomains";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
```

Next edit the main nginx config file:

```
#	vim /etc/nginx/nginx.conf
```

for now, you ware going to want to make a copy of the `server` block for the web app, and paste it below thus making a new `server` block. Add/edit these changes in to enable SSL for the new server:

```
listen       443 ssl http2 default_server;
ssl_certificate /etc/ssl/certs/nginx.crt;
ssl_certificate_key /etc/ssl/certs/nginx.key;
ssl_dhparam /etc/ssl/certs/dhparam.pem;
include /etc/nginx/default.d/*.conf;
```

After that restart nginx:

```
#	systemctl restart nginx
```

The last thing you will need to do is enable SSL on the Joomla site. To do that, go to this URL:
```
https://192.168.234.163/administrator/
```

navigate through the menu to here:

System > Global Configuration > Server

There is an option for `Force HTTPS`. You can configure it to enforce it on the entire site, then save. After that, you will be good to go (if you want, you should be able to disable the http server (the `server` block that you copied from) by just commenting it out of the config, and restarting nginx: 

#### Joomla

For Joomla, we are going to worry about the files for the web app, and the IPTables rules:

*	Webshells
*	Immutable
*	IPTables

##### Webshells

Webshells are going to be your biggest theat. You can try looking through the first couple of directoreies for any "out of place" files (and trylooking through the code for the files). Another way to proactively search for web shells is by grepping for functions that are typically used by webshells (waring, this will return a lot of false positives, so you need to be able to sort through them):

```
#	grep -iR "exec*(" /usr/share/nginx/html/
#	grep -iR "shell_exec*(" /usr/share/nginx/html/
```

another way of finding webshells is by monitoring the access logs. Essentially you just look for any requests that does not seem to be a scorring check, and you look at the file which they are talking to (this is not malicious).

```
#	cat /var/log/nginx/access.log
192.168.234.1 - - [10/Mar/2018:18:23:59 -0500] "GET /administrator/index.php?option=com_ajax&format=json HTTP/2.0" 200 59 "https://192.168.234.163/administrator/index.php?option=com_config" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
```

or you could use `tail`, which will display the logs as the come (this is not malicious):

```
#	tail -f /var/log/nginx/access.log
192.168.234.1 - - [10/Mar/2018:18:29:20 -0500] "GET /administrator/ HTTP/2.0" 200 28086 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
192.168.234.1 - - [10/Mar/2018:18:29:20 -0500] "GET /media/plg_quickicon_joomlaupdate/js/jupdatecheck.js?05f2a3c22ea234b26edfbd4904473a01 HTTP/2.0" 200 2108 "https://192.168.234.163/administrator/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
192.168.234.1 - - [10/Mar/2018:18:29:20 -0500] "GET /media/plg_quickicon_extensionupdate/js/extensionupdatecheck.js?05f2a3c22ea234b26edfbd4904473a01 HTTP/2.0" 200 1734 "https://192.168.234.163/administrator/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
192.168.234.1 - - [10/Mar/2018:18:29:21 -0500] "GET /administrator/index.php?option=com_installer&view=update&task=update.ajax&41c66129850521975113f4693de87ffa=1&eid=0&skip=700 HTTP/2.0" 200 2 "https://192.168.234.163/administrator/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
192.168.234.1 - - [10/Mar/2018:18:29:22 -0500] "GET /administrator/index.php?option=com_installer&view=update&task=update.ajax&41c66129850521975113f4693de87ffa=1&eid=700&cache_timeout=3600 HTTP/2.0" 200 2 "https://192.168.234.163/administrator/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
```

##### Immutable

After you setup the web app, you really don't have a reason to change it at all. What you can do is you can use `chattr` which can make the web directory immutable, so it cannot be changed. This will prevent an attacker from editing the web app (or uploading shells):

To make the directory immutable:
```
#	chattr -R +i /usr/share/nginx/html/
```

to undo the Chattr command so the web directory can be editied:

```
#	chattr -R -i /usr/share/nginx/html/
```
##### IPtables Rules

For the IPTables rules, we will just whitelist the minimal connections necissary for the web app to run, then block everything else. As for connections we will allow inbound/outbund `80` and `443` for HTTP and HTTPS (if you are not using one, you can just block it). We will also be allowing inbound/outbound `9000` and `3306` (both as source and destination ports) only from the loopback address `127.0.0.1` so Nginx, PHP-FPM. and MySQL can talk to each other:


Ingress Rules:
```
#	iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 443 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 9000 -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 9000 -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 3306 -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 3306 -s 127.0.0.1 -j ACCEPT
```

Egress Rules:
```
#	iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 9000 -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 9000 -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 3306 -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 3306 -d 127.0.0.1 -j ACCEPT
```

Block everything else:
```
#	iptables -P INPUT DROP
#	iptables -P OUTPUT DROP
#	iptables -P FORWARD DROP
#	ip6tables -P INPUT DROP
#	ip6tables -P OUTPUT DROP
#	ip6tables -P FORWARD DROP
```


#### MySQL

For locking down mysql, ensure that the user you are using to log into mysql has a strong user, and only has access to the database for the web app (check MySQL documentation for how to do that). To change your MySQL credentials for your Joomla config, edit this file:
```
#	vim /usr/share/nginx/html/configuration.php 
```

edit these settings as needed:

```
public $dbtype = 'mysqli';
public $host = '127.0.0.1';
public $user = 'dragonslayer';
public $password = 'Holiday8';
public $db = 'black_flame';
public $dbprefix = 'tf48r_';
```

## Logs

For logs, you should try checking Nginx, and php-fpm logs, along with syslogs:

You can view the access logs to see who viewed what, when and where:
```
#	cat /var/log/nginx/access.log
192.168.234.1 - - [10/Mar/2018:18:51:59 -0500] "GET /administrator/index.php?option=com_ajax&format=json HTTP/2.0" 200 59 "https://192.168.234.163/administrator/index.php?option=com_config" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
192.168.234.1 - - [10/Mar/2018:18:52:13 -0500] "GET /administrator/index.php?option=com_ajax&format=json HTTP/2.0" 200 59 "https://192.168.234.163/administrator/index.php?option=com_config" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
192.168.234.1 - - [10/Mar/2018:18:55:12 -0500] "GET /index.php/component/ajax/?format=json HTTP/1.1" 404 162 "http://192.168.234.163/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
192.168.234.1 - - [10/Mar/2018:18:57:32 -0500] "GET /index.php/component/ajax/?format=json HTTP/2.0" 404 162 "https://192.168.234.163/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
```

You can view the `nginx` error log, to see errors such as a blacklisted php function being called:
```
#	cat /var/log/nginx/error.log | grep security
PHP message: PHP Warning:  preg_replace() has been disabled for security reasons in /usr/share/nginx/html/libraries/src/Log/LogEntry.php on line 11
```

Here is an example log of an improper php session save path being set durring an installation:
```
#	cat /var/log/nginx/error.log
018/03/10 14:55:40 [error] 12562#0: *1 FastCGI sent in stderr: "PHP message: PHP Warning:  session_start(): open(/var/lib/php/session/sess_hefn6kr612qt1scltl1rb6c4o3, O_RDWR) failed: No such file or directory (2) in /usr/share/nginx/html/libraries/joomla/session/handler/native.php on line 260" while reading response header from upstream, client: 192.168.234.1, server: _, request: "GET /installation/index.php HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "192.168.234.163"
```

You can view the `messages` to view logs about how the daemons that support the web app are running:
```
#	cat /var/log/messages | grep nginx
Mar 10 18:08:58 localhost systemd: Stopping The nginx HTTP and reverse proxy server...
Mar 10 18:08:58 localhost systemd: Starting The nginx HTTP and reverse proxy server...
```

## Migration

We are migrating from one CentOS 7 box to another.

First install the needed componets:

```
#	yum install nginx
#	yum install php-fpm php-mysqlnd
#	yum install mariadb-server
```

You will need to configure Nginx and PHP-FPM to work together, look at the `Installation` section at the top of the writeup for that. Also for MySQL you will need to create a new Database, and user for Joomla which can also be found in `Installation`. 

#### MySQL

Next on the original machine (machine we are migrating from) we are going to dump the database which Joomla uses (in this case named `black_flame`), so we can transfer it to the server we are migrating (this command is ran on the old server):
```
#	mysqldump black_flame -u root -p > /tmp/dbtransfer
```

Next we are going to copy the database dump over from the original machine to the machine we are migrating to:

```
#	scp guyinatuxedo@192.168.234.163:/tmp/dbtransfer /tmp/
```

Then finally you just need to import the db dump into the new database:

```
#	mysql -u root -p black_flame < /tmp/dbtransfer
```

##### Files

For this, we are just going to copy over the web directory from the old server to the new server:
```
#	scp -rp root@192.168.234.163:/usr/share/nginx/html/* /usr/share/nginx/html/
#	chown -R nginx:nginx /usr/share/nginx/html/
```

If the mysql creds are different on the new server than the old server, you will need to change the mysql cred setting in the web app to reflect the new changes:
```
#	vim /usr/share/nginx/html/configuration.php
```

##### Networking

We will now need to switch the ip address to that of the old server (at this point the old server should either be off, or have a different IP). To do that, edit the config file for the ethernet adapter (in this case `ens33`):

```
#	sudo vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

switch it over to a static ip address of the old server:

```
BOOTPROTO=static
IPADDR=192.168.234.163
NETMASK=255.255.255.0
GATEWAY=192.168.234.1
```

If this is a new server, you may want to flush the firewall rules and put up your own (discussed in `Security`) so you can reach the web server:

```
#	iptables -F
```

Also make sure this SELinux bool is set so your web app can reach the db:

```
#	setsebool -P httpd_can_network_connect_db 1
```

at this point, the migration is complete!
