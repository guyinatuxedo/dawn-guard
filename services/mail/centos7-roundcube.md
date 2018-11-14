# Roundcube

## Installation

##### DNS Record

For your Mail server, make sure you have an MX and A record for it. here is an example MX and A record for bind:
```
hopesend        IN      A       192.168.234.173
@       IN      MX      10      hopesend
```

##### Exim

First install Exim:
```
$	sudo yum install exim
```

next open up the exim config file:
```
$	sudo vim /etc/exim/exim.conf
```

add/edit in the following settings:

```
primary_hostname = hopesend.endless.night.nz
domainlist local_domains = @ : endless.night.nz

tls_certificate = /etc/pki/tls/certs/exim.pem
tls_privatekey = /etc/pki/tls/private/exim.pem

tls_advertise_hosts =
auth_advertise_hosts = *
```

later on in the conf file you will need to add/edit in this setting so Roundcube can send and receive emails (if there are other mail severs, include the ip alongside the loopback):
```
   accept  hosts         = 127.0.0.1
```

now in the same config file, we are going to add the configuration for dovecot authentication. To do this look for the authentication section of the same conf file, and look for this string:
```
######################################################################
#                   AUTHENTICATION CONFIGURATION                     #
######################################################################

begin authenticators
```

after that add/edit in these configs:

```
dovecot_login:
        driver = dovecot
        public_name = LOGIN
        server_socket = /var/run/dovecot/auth-client
        server_set_id = $auth1

dovecot_plain:
        driver = dovecot
        public_name = PLAIN
        server_socket = /var/run/dovecot/auth-client
        server_set_id = $auth1
```

after that, you should be able to start and enable the exim service:
```
$	sudo systemctl start exim
$	sudo systemctl enable exim
```

##### Dovecot

First install dovecot:
```
$	sudo yum install dovecot
```

you need to configure dovecot to allow for plaintext authentication by editing this file:
```
$	sudo vim /etc/dovecot/conf.d/10-auth.conf
```

add/edit in the following configs:
```
disable_plaintext_auth = no
auth_mechanisms = plain login
```

next you will need to configure the mailbox location, and the group which the mail process will run as by editing this file:
```
$	sudo vim /etc/dovecot/conf.d/10-mail.conf
```

add/edit in the following configs:
```
mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_privileged_group = mail
mail_access_groups = mail
```

lastly we will need to configure Dovecot to allow for Exim to use it's authentication system, by editing this file:
```
$	sudo vim /etc/dovecot/conf.d/10-master.conf
```

search for the auth service, which will be this block of configs:
```
service auth {
	# Enter configs here
}
```

in that block add/edit in these configs:

```
        unix_listener auth-client {
                mode = 0660
                user = exim
        }
```

after that you should be able to start and enable the dovecot service:
```
$	sudo systemctl start dovecot
$	sudo systemctl enable dovecot
```


##### MariaDB

Install the Mariadb database:
```
$	sudo yum install mysql-server
```

Next you will need to start and enable the Mariadb service;
```
$	sudo systemctl start mariadb
$	sudo systemctl enable mariadb
``` 

first we will probably want to set a password for root to `p@55w0rd` (by default it doesn't have one):
```
$	mysql -u root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.2.13-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> update mysql.user set password = password("p@55w0rd");
Query OK, 6 rows affected (0.00 sec)
Rows matched: 7  Changed: 6  Warnings: 0

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```

next we will need to log in to mariadb and create the database which Roundcube will be using (named `hopesend`) and the user which it will be using (named `end` from the host `127.0.0.1` and the password `p@55w0rd`):
```
# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.2.13-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database hopesend;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user 'end'@'127.0.0.1' identified by 'p@55w0rd';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all privileges on hopesend.* to 'end'@'127.0.0.1';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> exit
Bye
```
##### Apache

Install Apache:
```
$	sudo yum install httpd
```

by default Fedora's firewall rules will block apache, so you should flush the default rules and setup your own (correct firewall rules covered in `security`):
```
$	sudo iptables -F
```

start and enable the service:
```
$	sudo systemctl start httpd
$	sudo systemctl enable httpd
```

next log in and create a database for rouncube to use

##### PHP

Install php and the needed plugins for php:
```
$	sudo yum install php php-mysqlnd php-common php-json php-xml php-mbstring php-imap
```

```
$	pear install Auth_SASL NET_SMTP NET_IDNA2-0.2.0 Mail_mime
```

after that, restart `php-fpm` and enable it:
```
$	sudo systemctl restart php-fpm
$	sudo systemctl enable php-fpm
```

##### Roundcube

You will want to download and extract the latest version of Roundcube. At the time of this writeup, it is `1.3.4`:

```
$	cd /tmp/
$	wget https://github.com/roundcube/roundcubemail/releases/download/1.3.4/roundcubemail-1.3.4-complete.tar.gz
$	tar -zxvf roundcubemail-1.3.4-complete.tar.gz
```

next you are going to want to copy over the Roundcube Webapp to the web directory, change the owner to Apache's user `www-data`, and remove the default Apache page:
```
$	sudo cp -avr roundcubemail-1.3.4/* /var/www/html/
$	sudo chown -R apache:apache /var/www/html/
```

next there are two SELinux bools that you will need to set, so Apache can access the network and write to it's web directory:
```
$	sudo setsebool -P httpd_unified 1
$	sudo setsebool -P httpd_can_network_connect 1
```

After that you should be ready to install the webapp. To install it just naviagate to `http:<ip address>/installer` in a web browser. So your ip address for the mail server is `192.168.234.173` you will visit this link in a web browser:

```
http://192.168.234.173/installer/
```

The installation process has three steps:
*	Env Check:	Checks to see that you have the right enviornment, checks things like php modules
*	Generate Config: Generates Config for how roundcube will interface with the database/imap/smtp and other settings (only thing you should have to set here are the MySQL creds, the rest should be defaults)
*	Check Config: Here you can check the config by testing the DB, IMAP, and SMTP services

After that, you will just need to remove the installation directory, then you should be done:

```
$	sudo rm -rf /var/www/html/installer/
```

## Security

There are five componets that you need to secure:

*	Apache
*	PHP
*	MySQL
*	Exim
*	Dovecot

#### Apache

*	Apache's User
*	Directory Options
*	Timeout
*	Clickjacking
*	XSS	
*	Disable HTTP Trace
*	Limit Request Types
*	Server Version
*	Allow Override
*	Options
*	SSL

##### Directory Opttions


#### PHP


*	Specify what clients can run code
*	Privileges of the service
*	Where is it listening
*	Base Directory
*	Blacklist php functions
*	Remote FIle Inclusion
*	Limit File Uploads
*	Don't expose PHP
*	Limit Post Requests Size
*	Check MySQLi Confs
*	Check resources (memory usage)
*	Configure Logging

##### Clients

To specify what IP addresses php-fpm will take code from,e dit this config file:

```
#	vim /etc/php-fpm.d/www.conf
```

add/edit in this setting, it should only have the ip addresses which need this functionallity (try to keey it to 127.0.0.1):

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
user = apache
group = apache
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
listen.owner = apache
listen.group = apache
```

To sepcify an access control list user/group which the unix socket will run with. This will override the above listen.owner and listen.group setiings:

```
listen.acl_users = apache
listen.acl_groups = apache
```

##### Base Directory

You can restrict what php code can execute to that of a single directory. To do that edit this directory:

```
#	vim /etc/php.ini
```

add/edit in this setting (it should be that of the web root of the webapp, so php code outside of the web directory can't run):

```
open_basedir = /var/www/html/
```

##### Blacklist php functions

There are a lot of php functions, which you reallydon't need to have, which can be of great use to an attacker (so you should probably blacklist them). To do that, edit this file:

```
#	vim /etc/php.ini
```

add/edit in this setting:

```
disable_functions = exec, passthru, shell_exec, system, proc_open, popen, curl, exec, curl_multi_exec, parse_ini_file, show_source, pcntrl_exec, eval, assert, create_function, include, include_one, require, require_once, phpinfo,  pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
```

This function if disable will break the web app:

```
    preg_replace(): Will stop the web app from running if diabled
```

##### Remote FIle Inclusion

You should disable allow_url_fopen and allow_url_include, since they can be used by an attacker to access files on the server, which they can use in exploits (Remote File Inclusion). To do that, edit this file:

```
#	vim /etc/php.ini
```

add/edit in the following settings

```
allow_url_fopen = Off
allow_url_include = Off
``` 

##### Limit File Uploads

File uploads are needed for this web app to function, however can still be limited:

```
#	vim /etc/php.ini
```

add/edit in these settings:

```
file_uploads = On
upload_max_filesize = 1M
max_file_uploads = 5
```

##### Don't expose PHP

PHP can expose that it is installed on a server, which will be potentially helpful to an attacker. To disable it edit this conf file:

```
#	vim /etc/php.ini
```

add/edit this in:

```
expose_php = Off
```

##### Limit Post Request Size

Depending on what you are doing, you should probably limit the size of post requests which PHP will take, since this will help prevent certain type of DOS attacks. To do that edit this conf file:

```
#	vim /etc/php.ini
```

add/edit this in:

```
post_max_size = 512K
```

Check MySQLi Confs

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
```
Should probably check the port/socket mysqli is using:
```
```
mysqli.default_port = 3306
```
```
mysqli.default_socket =
```

##### Configure Logging

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
#### Issues

If you are experiencing an issue in Roundcube where you click a button (like the `send` button) and nothing happens, chances are you are missing the javascript dependiencies. To see what you're missing, look at te debug console in firefox when you visit a site. To correct the problem download a complete version of roundcube and either reinstall it or copy over the dependiecnies (certain versions of this web app do no thave all of the Javascript dependencies).
