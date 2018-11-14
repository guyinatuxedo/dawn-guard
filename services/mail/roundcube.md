# Squirrelmail

This is a guide for Squirrelmail using Postfix version `3.1.0` and Dovecot version `2.2.22` on Ubuntu `16.04.3`

## Installation

#### DNS Record

For your Mail server, make sure you have an MX and A record for it. here is an example MX and A record for bind:
```
hopesend        IN      A       192.168.234.173
@       IN      MX      10      hopesend
```

#### Postfix

First you need to install postfix, which is used to deliver and route mail:
```
$	sudo apt-get install postfix
```

Durring your installation, it will prompt you for what configuration you want. Choose `Internet Site` and when it will prompt you for the domain name fo rwhat the email server is going to be serving (for mine I put`endless.night.nz`). After that you can restart and enable postfix:

```
$	sudo systemctl restart postfix
$	sudo systemctl enable postfix
```

#### Dovecot

Next you will need to install Dovecot, which essentially uses `IMAP` and `POP3` protocols to retrieve your email:
```
$	sudo apt-get install dovecot-imapd dovecot-pop3d
```

after that, just restart and enable the dovecot daemon:

```
$	sudo systemctl restart dovecot
```

#### Apache & PHP

You will need to install apache, php, and the php mod for apache since for the web mail client:

```
$	sudo apt-get install apache2 php libapache2-mod-php
```

#### Squirrelmail

For the next step, we can install the `squirrelmail` web app:

```
$	sudo apt-get install squirrelmail
```

proceeding that, we can enter into the nice and neat squirrelmail config menue:

```
$	sudo squirrelmail-configure
```

You should see a menu like this for the `Main Menu`:

```
SquirrelMail Configuration : Read: config.php (1.4.0)
---------------------------------------------------------
Main Menu --
1.  Organization Preferences
2.  Server Settings
3.  Folder Defaults
4.  General Options
5.  Themes
6.  Address Books
7.  Message of the Day (MOTD)
8.  Plugins
9.  Database
10. Languages

D.  Set pre-defined settings for specific IMAP servers

C   Turn color on
S   Save data
Q   Quit

Command >> 
```

press `2` to enter the `Server Settings` menu, which will display a menu like this:

```
SquirrelMail Configuration : Read: config.php (1.4.0)
---------------------------------------------------------
Server Settings

General
-------
1.  Domain                 : trim(implode('', file('/etc/'.(file_exists('/etc/mailname')?'mail':'host').'name')))
2.  Invert Time            : false
3.  Sendmail or SMTP       : SMTP

A.  Update IMAP Settings   : localhost:143 (other)
B.  Update SMTP Settings   : localhost:25

R   Return to Main Menu
C   Turn color on
S   Save data
Q   Quit

Command >> 1

The domain name is the suffix at the end of all email addresses.  If
for example, your email address is jdoe@example.com, then your domain
would be example.com.

[trim(implode('', file('/etc/'.(file_exists('/etc/mailname')?'mail':'host').'name')))]: endless.night.nz
```

press `1` to change the domain name, then input it (for my domain it is `endless.night.nz`). Then go back to the main menu by pushing `R`, then enter general settings by pushing `4` which will display a menu like this:

```
SquirrelMail Configuration : Read: config.php (1.4.0)
---------------------------------------------------------
General Options
1.  Data Directory               : /var/lib/squirrelmail/data/
2.  Attachment Directory         : /var/spool/squirrelmail/attach/
3.  Directory Hash Level         : 0
4.  Default Left Size            : 150
5.  Usernames in Lowercase       : false
6.  Allow use of priority        : true
7.  Hide SM attributions         : false
8.  Allow use of receipts        : true
9.  Allow editing of identity    : true
    Allow editing of name        : true
    Remove username from header  : false
10. Allow server thread sort     : false
11. Allow server-side sorting    : false
12. Allow server charset search  : true
13. Enable UID support           : true
14. PHP session name             : SQMSESSID
15. Location base                : 
16. Only secure cookies if poss. : true
17. Disable secure forms         : false
18. Page referal requirement     : 
19. Browser rendering mode       : quirks

R   Return to Main Menu
C   Turn color on
S   Save data
Q   Quit

Command >> 
```

You will need to enable `server-side sorting` by entering `11` to change the setting and then `y` to set it equal to true. Then you can save and exit by pushing `R` to return to the main menu, then `S` to save, and finally `Q` to quit:

After that, you will need to copy the apache virtual host file for squirrelmail into the `sites-available` directory, then enable it:

```
$	sudo cp /etc/squirrelmail/apache.conf /etc/apache2/sites-available/squirrelmail.conf
$	sudo a2ensite squirrelmail.conf
$	sudo a2dissite 000-default.conf
$	sudo systemctl restart apache2
```
 
 after that, you should be able to reach the web app by naviagating to the `<ipaddress>/squirrelmail`. So if your IP address is `192.168.234.169`, you would reach the web app by going here:
```
http://192.168.234.169/squirrelmail
```

When you log in with a user, you do not need to include the domain suffix (just the user name):

##### Users

By default, the mail client will use local users of the mail server for the users to the mail client. To enable a local user of the mail server user to use this mail client, you will need to add it to the mail group:
```
$	usermod -aG mail guyinatuxedo
```

to see who can use the mail client ()in this case it is the users `guyinatuxedo` and `guy`:
```
#	cat /etc/group | grep mail
mail:x:8:guyinatuxedo,guy
```

## Security

For securing Squirrelmail, there are 5 different things you will need to secure:

*	Postfix
*	Dovecot
*	Apache
*	PHP
*	Squirrelmail Webapp

#### Postfix

First off, you will want to disable the `vrfy` command, which essentially allows for the mail server to report if a user exists or not. This command is typically not needed for normal functionallity, and can help an attacker figure out what users the mails server serves:

```
disable_vrfy_command = yes
```

make sure that postfix is set to run with the user id of postfix:
```
mail_owner = postfix
```

Make sure to configure postfix to only listen on the interfaces which it is needed to. Here we have it configured to only listen on `127.0.0.1`:

```
inet_interfaces = 127.0.0.1
```

It also is a good idea to configure your server to send a `HELO` when it contacts another server. This is a sign that the server has been properly configured, and is not just sending spam:
```
smtpd_helo_required = yes
```

##### Relaying

You should avoid being an open mail relay which will allow for anyone to proxy their email throug you (essentially allowing anyone to use you to send mail anywhere). Your server should be configured to only send mail to users of your domain. 

You can specify what ip ranges you want to relay mail to with the `mynetworks` command. Typically it should only be set to your looback address (unless there are other mail servers that your domain has):

```
mynetworks = 127.0.0.0/8
```

You can also specify what domains your server can relay mail to:
```
relay_domains = endless.night.nz
```

Lastly you should specify to reject mail that is sent from sources such as ones that the sender/domain/destination hostname is unknown

```
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination reject_non_fqdn_recipient reject_non_fqdn_sender reject_unknown_recipient_domain reject_unknown_sender_domain reject_unauth_destination reject_invalid_hostname reject_unknown_helo_hostname
```

##### Denial of Service

Here are settings that will help prevent a Denial of Service by limiting the amount of work a client can put on the server:

Specify the maximum amount of connections a client is allowed to make per unit of time specified by `anvil_rate_time_unit`:

```
smtpd_client_connection_rate_limit = 30
```

This is the amount of time in secons that is used by other settings such as `smtpd_client_connection_rate_limit`:

```
anvil_rate_time_unit = 10
```

This specifies the clients that are excluded from the connection and rate limit commands (by default it just has the clients from trusted connections):

```
smtpd_client_even_limit_exceptions = 
```

This specifies the maximum number of message deliveries a client is allowed to request per unit of time specified by (`anvil_rate_time_unit`) regardless of if they are approved or rejected:

```
smtpd_client_message_rate_limits = 30
```

This specifies the maximum number of Postfix child preocesses that can proovide a given service:

```
default_process_limit = 100
```

This specifies the minimum amount of free space in bytes in the queue file system thta must be present to receive mail

```
queue_minfree = 20971520
```

This specifies the maximum size in bytes of a message header:

```
header_size_limit = 51200
```

This specifies the maximum size in bytes of a message:

```
message_size_limit = 10485760
```

#### Dovecot

First check to ensure that the background Dovecot process is running as the correct user. To do that edit this file:

```
#	vim /etc/dovecot/conf.d/10-master.conf
```

then check this setting:
```
default_internal_user = dovecot
```

##### Dovecot Users

You should specify the process users for the authentication. You can do that by editing this file:
```
#	vim /etc/dovecot/conf.d/10-master.conf
```

For the `service-auth` process, it should have this configured as it's user:

```
service auth {
  user = $default_internal_user
}
```

For the `service-worker` process, it should have this configured as it's user and group (the default user is root):

```
service auth-worker {
  user = $default_internal_user
  group = shadow
}
```

#### Apache


*	Virtual Host
*	Apache's user
*	Directoriy options
*	Timeout
*	ClickJacking
*	XSS
*	Disable HTTP Trace
*	Limit Request Types
*	Server Version
*	Allow Override
*	Options
*	SSL

##### Virtual Host

The virtual Host config file by default comes missing a couple of parameters that we would want to specify (like the access and error logs). To change this edit this file:

```
#	vim /etc/apache2/sites-enabled/squirrelmail.conf
```

add/edit in these settings:

```
<VirtualHost 192.168.234.169:80>
Alias /squirrelmail /usr/share/squirrelmail
ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
<Directory /usr/share/squirrelmail>
 # Options FollowSymLinks
   Options -Indexes -Includes
 <IfModule mod_php.c>
    php_flag register_globals off
  </IfModule>
  <IfModule mod_dir.c>
    DirectoryIndex index.php
  </IfModule>

  # access to configtest is limited by default to prevent information leak
  <Files configtest.php>
    order deny,allow
    deny from all
    allow from 127.0.0.1
  </Files>
</Directory>
</VirtualHost>
```

##### Apache's User

You should check what user apache is runnin as (on this system it should be `www-data`). To do that, check this config file for Apache's enviornment variables:

```
#	cat /etc/apache2/envvars | grep USER
export APACHE_RUN_USER=www-data
#	cat /etc/apache2/envvars | grep GROUP
export APACHE_RUN_GROUP=www-data
```

here we can see that the enviornment variable `APACHE_RUN_USER` and `APACHE_RUN_GROUP` is set equal to `www-data`, which is what it should be. Now let's just make sure that apache is actually using those variables:

```
#	cat /etc/apache2/apache2.conf | grep USER
User ${APACHE_RUN_USER}
#	cat /etc/apache2/apache2.conf | grep GROUP
Group ${APACHE_RUN_GROUP}
```

Here we confirmed that the apache service is indeed running as the correct users:

##### Directory Options

If you are only hosting Squirrelmail (which it's default web directory is `/usr/share/squirrelmail/`) you should have that configured as the only directory apache is serving. In addition to that you should disable Directory Indexing and Server side includes for all directories which you are serving. So just comment out any directories other than `/usr/share` and `/` and make sure they look like this:

```
<Directory />
        Options FollowSymLinks
        AllowOverride None
        Require all denied
</Directory>

<Directory /usr/share>
        AllowOverride None
        Require all granted
</Directory>
```

##### Timeout

Essentially shorten the timeout window, in order to help prevent DOS attacks. Edit the main apache config file:

```
#	vim /etc/apache2/apache2.conf
```

and add/edit in this config

Timeout 60

##### ClickJacking

Clickjacking is essentially when an attacker embeddes pages as frames. To prevent this, you can you first have to enable the headers (might be included in security.conf under conf-available):

```
$   sudo a2enmod headers
$   sudo systemctl restart apache2
```

then edit your main apache conf file:

```
#   vim /etc/apache2/apache2.conf
```

add/edit this in:

```
Header set X-Frame-Options: "sameorigin"
```

##### XSS


Cross Site Scripting (XSS) is a type of attack where code is injected into a website, then executed. Apache can help mitigate this threat. First ensure the mod headers is enabled:
```
$   sudo a2enmod headers
$   sudo systemctl restart apache2
```
then edit your main apache conf file:
```
#   vim /etc/apache2/apache2.conf
```
add/edit this in:
```
Header set X-XSS-Protection "1; mode=block"
```

##### Disable HTTP Trace


Http trace can be used in Cross-Site scripting attacks, since the Trace request includes all http headers and cookie contents include authentication authentication data, so this should be disabled. To disable it edit your main apache conf file (this may be included in your security.conf file unser conf-available):

then edit your main apache conf file:

```
#   vim /etc/apache2/apache2.conf
```

add/edit this in:
```
TraceEnable Off
```

##### Limit Request Types

For most web apps, the only requests you need are Get, Head, and Post. However Http 1.1 supports additional requests such as Options, Put, Delete, Trace, and Connect which can be helpful to an attacker. We can whitelist the requests which we need. To do this edit the main apache conf file:
```
#   vim /etc/apache2/apache2.conf
```
add/edit in the following lines to in the directory sections in the directort sections you wish to protect:

The options to whitelist the requests
```
	<LimitExcept GET POST HEAD>
	deny from all
	</LimitExcept>
```

##### Server Version

By default, apache will display a banner which prints out the server version apache is running, which can be helpful for an attacker to figure out which metasploit module to use to pwn you. To remove this, edit the main apache conf file:

```
#   vim /etc/apache2/apache2.conf
```

add/edit in the following settings:

```
ServerTokens Prod
ServerSignature Off
```

#### php

*    Blacklist php functions
*    Remote FIle Inclusion
*    Disable File Uploads
*    Set Base Dir
*    Don't expose PHP
*    Limit Post Requests Size
*    Check MySQLi Confs
*    Check resources (memory usage)
*    Configure Logging

All of the following configs will take place in the following config file (the `php.ini` file):
```
#	vim /etc/php/7.0/apache2/php.ini
```

##### Blacklist php functions

You should disable certain php functions such as exec. Here is a list of php functions you should blacklist:

```
disable_functions = pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
```
##### Remote FIle Inclusion

You should disable allow_url_fopen and allow_url_include, since they can be used by an attacker to access files on the server, which they can use in exploits (Remote File Inclusion):

```
allow_url_fopen = Off
allow_url_include = Off
```

##### Disable File Uploads

To disable file uploads (which will disable an attacker from uploading things such as web shell) add/edit in these settings:

```
file_uploads = Off
upload_max_filesize = 0M
max_file_uploads = 0
```

##### Set Base Dir

You can limit the file operations of php to a defined directory (or directories), which can limit what an attacker can do. We should limit it to the directories which squirrelmail needs to run, which are the web directory `/usr/share/squirrelmail/`, the squirrelmail config directory `/etc/squirrelmail/`, and the directory which squirrelmail can write data to `/var/lib/squirrelmail/data/`:

```
open_basedir = /usr/share/squirrelmail/:/etc/squirrelmail/:/var/lib/squirrelmail/data/
```

##### Don't expose PHP

PHP can expose that it is installed on a server, which will be potentially helpful to an attacker. To disable it add/edit in this conf:

```
expose_php = Off
```

##### Limit Post Requests Size

Depending on what you are doing, you should probably limit the size of post requests which PHP will take, since this will help prevent certain type of DOS attacks:

```
post_max_size = 512K
```

##### Check resources (memory usage)

By default, a php script can consume up to 128 Megabytes, which for our needs is too much. This can be descreased to 10M, in order to help prevent DOS attacks:

```
memory_limit = 10M
```

##### Configure Logging

Make sure to enable logging with these commands

```
log_errors = On
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
```

also check to ensure that the logging location is acurate (should probably be commented out by default):

```
; error_log = syslog
```

#### Squirrelmail

##### Webshells

Your biggest threat here is going to be webshells. For this, try looking through the directories and the files (at least for the first couple of directory layers, it's not pheasable to go through everything). You could also try using the grep command to search for things (may not catch obfuscated code):

search for `exec`
```
#	grep -iR "exec*(" /usr/share/squirrelmail/
/usr/share/squirrelmail/plugins/squirrelspell/modules/check_me.mod:    exec("$sqspell_command < $floc 2>&1", $sqspell_output, $sqspell_exitcode);
/usr/share/squirrelmail/plugins/fortune/fortune_functions.php:        $sMsg = sm_encode_html_special_chars(shell_exec($fortune_location . ' -s'));
```

search for `shell_exec`:
```
#	grep -iR "shell_exec*(" /usr/share/squirrelmail/
/usr/share/squirrelmail/plugins/fortune/fortune_functions.php:        $sMsg = sm_encode_html_special_chars(shell_exec($fortune_location . ' -s'));
```

search for `eval`:
```
#	grep -iR "eval*(" /usr/share/squirrelmail/
/usr/share/squirrelmail/functions/addressbook.php:        eval('$newback = new ' . $backend_name . '($param);');
/usr/share/squirrelmail/config/conf.pl:    unless (eval("use IO::Socket; 1")) {
```

search for `assert`:
```
#	grep -iR "assert*(" /usr/share/squirrelmail/
```

search for `gzdeflate`:
```
#	grep -iR "gzdeflate*(" /usr/share/squirrelmail/
```

search for `rot` ciphers:
```
#	grep -iR "rot*(" /usr/share/squirrelmail/
Binary file /usr/share/squirrelmail/locale/pt_PT/LC_MESSAGES/squirrelmail.mo matches
Binary file /usr/share/squirrelmail/locale/pt_BR/LC_MESSAGES/squirrelmail.mo matches
```

Another way to detect webshells (with Prestashop, will probaly be better) is to simply monitor Apache's access logs (make sure that you have logging configured for this virtual host), and notice anything suspicious (these are examples of viewing the logs, these request are not malicious):

```
#	cat /var/log/apache2/access.log
192.168.234.1 - - [12/Mar/2018:20:43:24 -0700] "GET /squirrelmail/src/compose.php?mailbox=INBOX&startMessage=1 HTTP/1.1" 200 2754 "http://192.168.234.169/squirrelmail/src/right_main.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

or you could use tail, which will display the logs to you as they come:

```
$	sudo tail -f /var/log/apache2/access.log
92.168.234.1 - - [12/Mar/2018:20:43:14 -0700] "GET /squirrelmail/src/login.php HTTP/1.1" 200 2251 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [12/Mar/2018:20:43:14 -0700] "GET /squirrelmail/images/sm_logo.png HTTP/1.1" 304 165 "http://192.168.234.169/squirrelmail/src/login.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

##### Immutable

You can make the web directory immutable, which means that no one can create/edit/delete files in it. This will pose another barrier to attackers, since it will stop them from changing the files of the webapp. To make the web directory immutable:

```
#	chattr -R +i /usr/share/squirrelmail/
chattr: Operation not supported while reading flags on /usr/share/squirrelmail//config
chattr: Operation not supported while reading flags on /usr/share/squirrelmail//plugins/squirrelspell/sqspell_config.php
chattr: Operation not supported while reading flags on /usr/share/squirrelmail//plugins/filters/setup.php
```

to undo the chattr command:

```
#	chattr -R -i /usr/share/squirrelmail/
chattr: Operation not supported while reading flags on /usr/share/squirrelmail//config
chattr: Operation not supported while reading flags on /usr/share/squirrelmail//plugins/squirrelspell/sqspell_config.php
chattr: Operation not supported while reading flags on /usr/share/squirrelmail//plugins/filters/setup.php
```

##### IPTables

For this we will configure Ingress and Egress whitelist filtering. Here are the services which we will allow through the firewall:

*	HTTP: `80` so clients can reach the Web Client
*	SMTP: `25` so Postfix can use Simple Mail Transfer Protocol to send and receive emails
*	IMAP: `143` so outside servers can reach IMAP(probably can be restricted to loopback address, unless it needs to pass scoring checks)
*	POP3: `110` so outside servers can reach POP3 (probably can be restricted to loopback address, unless it needs to pass scoring checks)

Keep in mind that for `SMTP`, `IMAP`, and `POP3` our server is going to need to talk to those services in order to function. So we will have to punch holes in our firewall so they can talk to those services as a client, however we will restrict it only to the loopback address so this won't open up much of an attack space:

Input:
```
#	iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 25 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 25 -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 143 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 143 -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 110 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 110 -s 127.0.0.1 -j ACCEPT
```

Output:
```
#	iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 25 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 25 -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 143 -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 110 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 110 -d 127.0.0.1 -j ACCEPT
```

block everything else:
```
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
ip6tables -P FORWARD DROP
```

## Logs

##### Postfix

All of these logs are from the viewpoint of Postfix

This is an example of an email being sent by Postfix:
```
#	cat /var/log/syslog
Mar 12 21:31:38 ubuntu postfix/smtpd[61443]: connect from localhost[127.0.0.1]
Mar 12 21:31:38 ubuntu postfix/smtpd[61443]: 8A14464E91: client=localhost[127.0.0.1]
Mar 12 21:31:38 ubuntu postfix/cleanup[61447]: 8A14464E91: message-id=<91e1af5ba607281200c09f2acb5f299e.squirrel@192.168.234.169>
Mar 12 21:31:38 ubuntu postfix/qmgr[61435]: 8A14464E91: from=<guy@endless.night.nz>, size=757, nrcpt=1 (queue active)
Mar 12 21:31:38 ubuntu postfix/smtpd[61443]: disconnect from localhost[127.0.0.1] ehlo=1 mail=1 rcpt=1 data=1 quit=1 commands=5
```

This is an example of an email being received by Postfix:
```
#	cat /var/log/syslog
Mar 12 21:31:38 ubuntu postfix/local[61448]: 8A14464E91: to=<guyinatuxedo@endless.night.nz>, relay=local, delay=0.07, delays=0.05/0.01/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
Mar 12 21:31:38 ubuntu postfix/qmgr[61435]: 8A14464E91: removed

```

##### Dovecot

All of these logs are from the viewpoint of Dovecot

here is an example of a user logging in:
```
#	cat /var/log/syslog
Mar 12 21:24:16 ubuntu dovecot: imap-login: Login: user=<guyinatuxedo>, method=PLAIN, rip=127.0.0.1, lip=127.0.0.1, mpid=60902, secured, session=</OOvoUNnJLZ/AAAB>
```

here is an example of a user logging out:
```
#	cat /var/log/syslog
Mar 12 21:24:16 ubuntu dovecot: imap(guyinatuxedo): Logged out in=296 out=5835
```

when a user reads an email, it will appear as though they just logged in and out, like this:
```
#	cat /var/log/syslog
Mar 12 21:25:27 ubuntu dovecot: imap-login: Login: user=<guyinatuxedo>, method=PLAIN, rip=127.0.0.1, lip=127.0.0.1, mpid=60909, secured, session=<t9XjpUNnJrZ/AAAB>
Mar 12 21:25:27 ubuntu dovecot: imap(guyinatuxedo): Logged out in=143 out=2033
```

This is what it looks like when they receive an email:
```
#	cat /var/log/syslog
Mar 12 21:31:38 ubuntu dovecot: imap-login: Login: user=<guy>, method=PLAIN, rip=127.0.0.1, lip=127.0.0.1, mpid=61450, secured, session=<FEUHvENnRLZ/AAAB>
Mar 12 21:31:38 ubuntu dovecot: imap(guy): Logged out in=641 out=573
Mar 12 21:31:38 ubuntu dovecot: imap-login: Login: user=<guy>, method=PLAIN, rip=127.0.0.1, lip=127.0.0.1, mpid=61452, secured, session=<U2cIvENnRrZ/AAAB>
Mar 12 21:31:38 ubuntu dovecot: imap(guy): Logged out in=296 out=7652
```

##### Apache

All of these logs are from the viewpoint of apache:

here is an example of a user logging in:
```
#	cat /var/log/apache2/access.log
192.168.234.1 - - [12/Mar/2018:21:18:44 -0700] "GET /squirrelmail/src/login.php HTTP/1.1" 200 2331 "http://192.168.234.169/squirrelmail/src/signout.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [12/Mar/2018:21:18:44 -0700] "GET /squirrelmail/images/sm_logo.png HTTP/1.1" 304 165 "http://192.168.234.169/squirrelmail/src/login.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [12/Mar/2018:21:18:53 -0700] "POST /squirrelmail/src/redirect.php HTTP/1.1" 302 2254 "http://192.168.234.169/squirrelmail/src/login.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [12/Mar/2018:21:18:53 -0700] "GET /squirrelmail/src/webmail.php HTTP/1.1" 200 932 "http://192.168.234.169/squirrelmail/src/login.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [12/Mar/2018:21:18:53 -0700] "GET /squirrelmail/src/left_main.php HTTP/1.1" 200 1863 "http://192.168.234.169/squirrelmail/src/webmail.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [12/Mar/2018:21:18:53 -0700] "GET /squirrelmail/src/right_main.php HTTP/1.1" 200 3598 "http://192.168.234.169/squirrelmail/src/webmail.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

here is an example of a user logging out:
```
#	cat /var/log/apache2/access.log
192.168.234.1 - - [12/Mar/2018:21:20:01 -0700] "GET /squirrelmail/src/signout.php HTTP/1.1" 200 1458 "http://192.168.234.169/squirrelmail/src/right_main.php?mailbox=INBOX&sort=6&startMessage=1" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

here is an example of a user sending an email:
```
#	cat /var/log/apache2/access.log
192.168.234.1 - - [12/Mar/2018:21:19:33 -0700] "GET /squirrelmail/src/compose.php?mailbox=INBOX&startMessage=1 HTTP/1.1" 200 2755 "http://192.168.234.169/squirrelmail/src/read_body.php?mailbox=INBOX&passed_id=10&startMessage=1" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [12/Mar/2018:21:19:48 -0700] "POST /squirrelmail/src/compose.php HTTP/1.1" 302 793 "http://192.168.234.169/squirrelmail/src/compose.php?mailbox=INBOX&startMessage=1" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [12/Mar/2018:21:19:48 -0700] "GET /squirrelmail/src/right_main.php?mailbox=INBOX&sort=6&startMessage=1 HTTP/1.1" 200 3685 "http://192.168.234.169/squirrelmail/src/compose.php?mailbox=INBOX&startMessage=1" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

here is an example of a user reading an email:
```
#	cat /var/log/apache2/access.log
192.168.234.1 - - [12/Mar/2018:21:19:16 -0700] "GET /squirrelmail/src/read_body.php?mailbox=INBOX&passed_id=10&startMessage=1 HTTP/1.1" 200 2397 "http://192.168.234.169/squirrelmail/src/right_main.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

## Migration

To migrate the server, simple install squirrelmail on the server that you wish to migrate to, then copy over the mail files, which should be found here:
```
#	ls /var/mail/
```

## Extra

##### Versions

To find out what version of postfix you are running:
```
#	postconf -d | grep mail_version
mail_version = 3.1.0
milter_macro_v = $mail_name $mail_version
```

To find out what version of Dovecot you are running:
```
#	dovecot --version
2.2.22 (fe789d2)
```

##### Break Apache

These security settings which are listed in my apache security writeup if enabled will break the web app:

*	If you disable HTTP 1.0, you will break the web app.
*	XSS Protection with `Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure` (the one line that is up there in this writeup should be fine)

##### Break Dovecot

Mail Process Chrooting (when I tired it) did break the web app.

##### misc.

If you have two `smtpd_relay_restrictions` declared in your postfix conf, you will get the following error:
```
#	cat /var/log/syslog
Mar 12 21:29:49 ubuntu postfix/trivial-rewrite[61097]: warning: /etc/postfix/main.cf, line 33: overriding earlier entry: smtpd_relay_restrictions=permit_mynetworks permit_sasl_authenticated defer_unauth_destination reject_non_fqdn_recipient reject_non_fqdn_sender reject_unknown_sender_domain
```
