# PrestaShop

For this writeup, I will assume you have already setup PHP, MySQL, and Apache. If not go look at those writeups and setup those services first , or run the following command to install:
```
#	apt-get install apache2 libapache2-mod-php php-mysql mysql-server
```

## Installation


### Mysql Setup
First you will need to setup a database in MySQL for Prestashop to use. First connect to the database using the user `root` (which has a password), and is at the ip address `192.168.234.154` (if your db is on the same machine, you can leave out the `-h` flag):
```
mysql -u root -p -h 192.168.234.154
```

next create a database named `horror` for Prestashop:
```
MariaDB [(none)]> create database horrors;
Query OK, 1 row affected (0.00 sec)
```

then create a new user named `silent`, which can access it from the ip address `192.168.234.154` (if it is on the same box, put `localhost`), with the password `password` and grant it the privileges which Prestashop will use to interact with the database `horrors`:
```
MariaDB [(none)]> create user `silent`@`localhost` identified by 'password';
MariaDB [(none)]> grant all privileges on horrors.* to 'silent'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

after that, just flush the privileges and you will be good to go:
```
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```

### PHP

You will need to install these php extensions for Prestashop to function:
```
#	apt-get install php-xml 
#	apt-get install php-zip
#	apt-get install php-curl
#	apt-get install php-gd
#	apt-get install php-intl
```

also durring the setup, these PHP settings from your `php.ini` will need to be configured in order for the installation to work:
```
open_basedir = 
allow_url_fopen = On
file_uploads = On
max_file_uplodas = 20
upload_max_filesize = 10M
```

also the function `preg_replace` cannot be disabled, so it can't be listed under `disable_functions`

Don't forget to restart apache:
```
#	systemctl restart apache2
```

### WebApp Setup

Make sure the Apache module `mod_rewrite` is enabled:
```
#	a2enmod rewrite
#	systemctl restart apache2
```

First download the newest version of prestashop (at this time it is 1.7.3.0):
```
#	cd /tmp
#	wget https://download.prestashop.com/download/releases/prestashop_1.7.3.0.zip
#	unzip prestashop_1.7.3.0.zip
  inflating: prestashop.zip 
  inflating: index.php
```

Next extract the contents of `prestashop.zip` to the web directory (in this case `/var/www/html`):

```
#	unzip prestashop -d /var/www/html/
```

Next you will probably wan to remove the default `index.html` file:
```
#	rm /var/www/html/index.html
```

Next you will need to grant the user apache runs as (in this case `www-data`) permissions to the file directory PrestaSpreg_replace()hop is stored in:
```
#	chown www-data:www-data /var/www/html/
#	chown www-data:www-data -R /var/www/html/*
```

proceeding that you should be able to navigate to the web server in a web browser, and finish the installation. If the IP address of your server is `192.168.234.154` (and your setup mimics ours) you should just be able to borwse to this in a web browser:
```
http://http://192.168.234.154
```

When you go there, you should go through the following steps:

*	Choose Language:	Choose what language you want to use
*	 License agreements:	Agree to the EULA
*	System Compatibillity:	May need to install, and/or change some settings
*	Store Information:	Enter in infomration as to stores purpose, and first admin account
*	Store configuration: Enter in information to MySQL Server
*	Installation:	This is where the store installs itself

Proceesing the installation, you will need to remove the `install` folder:
```
#	rm -rf /var/www/html/install/
```

## Config

### Debug

One thing that might really help, if you are just getting a blank white screen when you go to a webpage, that is because there is a php error, and it is not being displayed. You can change the settings of the webapp to display errors, however this should only be done for diagnostic purposes:

On older versions of prestashop (1.5-), edit this file

```
#	vim /var/www/html/config/config/config.inc.php
```

Switch this setting to `on` to enable it:

```
@ini_set('display_errors', 'off')
```
On newer versions of prestashop (1.5+) , edit this file

```
#	vim /var/www/html/config/defines.inc.php
```
Switch this setting to `true` to enable it:

```
define('_PS_MODE_DEV_', false);
```
### MySQL

The `MySQL` creds and server ip/port information are stored in a config file. To find it, and the relevant information:
```
#	grep -iR "horrors" /var/www/html/
/var/www/html/app/cache/prod/appParameters.php:    'database_name' => 'horrors',
/var/www/html/app/config/parameters.php:    'database_name' => 'horrors',
```

```
#	vim /var/www/html/app/config/parameters.php
```

change these settings as needed:
```
<?php return array (
  'parameters' => 
  array (
    'database_host' => '192.168.234.155',
    'database_port' => '',
    'database_name' => 'horrors',
    'database_user' => 'horror',
    'database_password' => 'Holiday7',
```

## Administration

The IP address of this prestashop instance is `192.168.234.155`

The admin folder should be under a directory titled `admin` with a random string appended to the end (or maybe not). To login to the admin pannel:
```
http://192.168.234.155/admin82869ysj5/
```

From there you should see the admin pannel. Below is various things regarding the gui that you can see:


*	Orders: Dashboard > Orders > Orders
*	Customers: Dashboard > Customers > Customers
*	Installed Modules: Dashboard > Modules > Modules Selecion menu above > Installed modules
*	Payment Methods: Dashboard > Paymnet > Payment Methods
*	Admin Users: Dashboard > Advanced Parameters > Team
*	Logs:	Dashboard > Advanced Parameters > Logs

## Security

Prestashop has essentially four seperate componets which each need to be secured:
*	PHP
*	Apache
*	Prestashop Itself
*	MySQL

### PHP

For PHP, the following configurations will assist with securring it. For this version, all of these configurations can be found in the following file:
```
#	vim /etc/php/7.0/apache2/php.ini
```

##### Blacklist PHP functions

You should blacklist unneeded php functions which can be helpful for an attacker:
```
disable_functions = exec, passthru, shell_exec, system, proc_open, curl, exec, curl_multi_exec, parse_ini,file, show_source, pcntrl_exec, eval, include, include_once, require, require_once, phpinfo, pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
```

Keep in mind that this function (which is present in a lot of PHP blacklist guides) is used:
```
preg_replace:	Will break the entire site if blacklisted
create_function:	Will break creating a new account if blacklisted
assert:	
```

##### Remote File Includion
You should disable these options, which will help an attacker access certain files.


```
allow_url_fopen = Off
allow_url_include = Off
```

##### Disable File Uploads

Disabling these options will prevent an attacker from uploading a shell:

```
file_uploads = Off
upload_max_filesize = 0
max_fille_uploads = 0
```

##### Set Base Directory

You should set the base directory to the web directory (directory where the prestashop files are, in this case it's `/var/www/html`). PHP code can only execute in this directory:

```
open_basedir = /var/www/html/
```

##### Expose PHP

You should disable the option, which will allow php to display itself:

```
expose_php = Off
```

##### DOS Prevention

You should limit the resources which PHP uses to execute script, execution to 5 seconds, and the memory to 30 Megabytes

```
max_execution_time = 5
max_input_time = 5
memory_limit = 30M
```

Also you should limit the size of post requests to `512` Kilobytes:

```
post_max_size = 512K
```

##### mysqli 

Prestashop (at least this version) uses `mysqli`. To confirm that, grep for `mysqli` and see what comes up. If it isn't `mysqli`, try `mysqlnd`:
```
#	grep -iR "mysqli" /var/www/html/
```

There are some default settings, make sure they are configured like this. Reason for that being, you don't want default creds configured (allows easier pivot to MySQL box), and you will want to keep the standar `3306` mysql port most likely:

```
mysqli.default_port = 3306
mysqli.default_socet =
mysqli.default_host =
mysqli.default_user = 
mysqli.default_pw =
```

##### Logs

You will want to make sure logs are enabled. Add/edit in these settings which will enable most logs:
```
log_errors = On
error_reporting = E_ALL & ~E_DEPRECEATED & ~E_STRICT
```
You will also want to disable these options, whcih can provide an attacker with valuable knowledge:
```
displaye_errors = Off
display_startup_erors = Off
```

Also check to see where error logs are begin sent to. If it is commented out, it should be stored in the main conf file (you should also be able to see errors in Apache's error log, since it is an apache module):

```
;error_log = syslog
```
restart apache to apply these changes:
```
#	systemctl restart apache2
```

### Apache

*	Directory Options
*	ClickJacking
*	HTTP 1.0
*	XSS
*	Disable HTTP Trace
*	Server Version
*	TimeOut
*	Disable HTTP 1.0
*	Logs
*	SSL

##### Directory Options

You want to make sure your apache directories are configured this way (provided that you only have one web directory, which is under `/var/www/`). This will disable directory indexing, server side includes, `.htaccess` override, all requests other than `GET` `POST` and `HEAD`, and other directories from being hosted: 
```
<Directory />
        <LimitExcept GET POST HEAD>
        deny from all
        </LimitExcept>
        Options -Indexes -Includes
        AllowOverride None
        Require all denied
</Directory>


<Directory /var/www/>
        <LimitExcept GET POST HEAD>
        deny from all
        </LimitExcept>
        Options -Indexes -Includes
        AllowOverride None
        Require all granted
</Directory>
```

##### Clickjacking

Clickjacking is essentially when an attacker embeddes pages as frames. To prevent this, you can you first have to enable the headers (might be included in security.conf under conf-available):
```
$   sudo a2enmod headers
$   sudo systemctl restart apache2
```
then edit your main apache conf file:

```
#   vim /etc/apache2/apache2.conf
```

add/edit this in to the end:
```
Header set X-Frame-Options: "sameorigin"
```
##### Disable HTTP 1.0

You should disable HTTP 1.0, since it is an outdated inseucre protocol (not the biggest security hole, and it can break stuff). 

First enable the `rewrite` module:
```
#	a2enmod rewrite
#	systemctl restart apache2
```

Next edit the main apache conf file:
```
#	vim /etc/apache2/apache2.conf
```

append this to the end to block all requests not HTTP 1.1:
```
RewriteEngine on
RewriteCond %{THE_RQUEST} !HTTP/1.1$
RewriteRule .* - [F]
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
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
```

##### Trace

Http trace can be used in Cross-Site scripting attacks, since the Trace request includes all http headers and cookie contents include authentication authentication data, so this should be disabled. To disable it edit your main apache conf file (this may be included in your security.conf file unser conf-available):

then edit your main apache conf file to the end:
```
TraceEnable Off
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

##### Timeout

Essentially shorten the timeout window, in order to help prevent DOS attacks. Edit the main apache config file:
```
#	vim /etc/apache2/apache2.conf
```
and add/edit in this config

```
Timeout 60
```

##### Logs

First, with the default confiruation Apache's log storages are dependent on the enviornment variable `APACHE_LOG_DIR` which can be found here (abridged):
```
#	cat /etc/apache2/envars
export APACHE_LOG_DIR=/var/log/apache2$SUFFIX
```

The actual log setting is typically set in the virtual host config file (abridged):
```
#	cat /etc/apache2/sites-enabled//000-default.conf
ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
``` 

Just check and make sure the logs are going somewhere that you want.

##### SSL

To configure SSL (should porbably only be done if it is an inject):

First off create the public certificaye, private key, and Defie-Hellman group. This is explained the in the apache writeup.

Proceeding that, make a new virtual host file, which for now is just going to be a copy. So copy your virtual host file (mine is named `silent-horror.conf`):
```
#	cp /etc/apache2/sites-enabled/silent-horror.conf /etc/apache2/sites-enabled/ssl-silent-horror.conf
```

edit the new virtual host file and add/edit in the following configurations to enable SSL (and for it to listen on port `443`):
```
<VirtualHost 0.0.0.0:443>
	SSLEngine on
	SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
	SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
```

then restart apache to make the conf changes:
```
#	systemctl restart apache2
```

Next navigate to the admin pannel of the Prestashop site. In it find the general tab from the Dashboard:
```
Dashboard > Configure > Shop Parameters > General
```

On the general tab, there should be a button to test SSL. Click on it, and it should redirect you to an https link of the admin pannel. From there you should be able to enable SSL, and then enable it for all pages. If you want to leave normal http, you cna just stop here, however if you want to disable http you will need to remove the virtual host file for it:
```
#	rm /etc/apache2/sites-enabled/silent-horror.conf
```

### Prestashop

Here we are going to focus on securing the web app directory itself, along with the iptables rules for everything:

*	Webshells	
*	Immutable
*	IPTables

##### Webshells

Your biggest threat here is going to be webshells. For this, try looking through the directories and the files (at least for the first couple of directory layers, it's not pheasable to go through everything). You could also try using the `grep` command to search for things (may not catch obfuscated code):

```
$	sudo grep -iR "shell_exec*(" .
./vendor/phpoffice/phpexcel/Examples/runall.php:	echo shell_exec('php ' . $sTest);
./vendor/nikic/php-parser/grammar/rebuildParsers.php:    $output = trim(shell_exec("$kmyacc $additionalArgs -l -m $skeletonFile -p $name $tmpGrammarFile 2>&1"));
./vendor/nikic/php-parser/grammar/rebuildParsers.php:    $output = trim(shell_exec("$kmyacc -l -m $tokensTemplate $tmpGrammarFile 2>&1"));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:        $sttyMode = shell_exec('stty -g');
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:        shell_exec('stty -icanon -echo');
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:        shell_exec(sprintf('stty %s', $sttyMode));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:            $value = rtrim(shell_exec($exe));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:            $sttyMode = shell_exec('stty -g');
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:            shell_exec('stty -echo');
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:            shell_exec(sprintf('stty %s', $sttyMode));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:            $value = rtrim(shell_exec($command));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/QuestionHelper.php:                if ('OK' === rtrim(shell_exec(sprintf($test, $sh)))) {
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:            $sttyMode = shell_exec('stty -g');
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:            shell_exec('stty -icanon -echo');
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:            shell_exec(sprintf('stty %s', $sttyMode));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:            $value = rtrim(shell_exec($exe));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:            $sttyMode = shell_exec('stty -g');
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:            shell_exec('stty -echo');
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:            shell_exec(sprintf('stty %s', $sttyMode));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:            $value = rtrim(shell_exec($command));
./vendor/symfony/symfony/src/Symfony/Component/Console/Helper/DialogHelper.php:                if ('OK' === rtrim(shell_exec(sprintf($test, $sh)))) {
```
try going through some of the functions listed under the `php` blacklist, and see if anything comes up. Another way to detect webshells (with Prestashop, will probaly be better) is to simply monitor Apache's access logs, and notice anything suspicious (these are examples of viewing the logs, these request are not malicious):

```
$	cat /var/log/apache2/access.log
192.168.234.1 - - [09/Mar/2018:18:46:25 -0800] "GET / HTTP/1.1" 500 9238 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

or you could use `tail`, which will display the logs to you as they come:
```
$	sudo tail -f /var/log/apache2/access.log
192.168.234.1 - - [09/Mar/2018:18:46:25 -0800] "GET / HTTP/1.1" 500 9238 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [09/Mar/2018:18:46:50 -0800] "GET / HTTP/1.1" 500 7772 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```


##### Immutable

You can make the web directory immutable, which means that no one can create/edit/delete files in it. This will pose another barrier to attackers, since it will stop them from changing the files of the webapp. To make the web directory immutable:
```
$	sudo chattr -R +i /var/www/html/
```
to undo the `chattr` command:
```
#	chattr -R -i /var/www/html/
```

##### IPTables

Here are the IPTables rules to have a whitelist firewall for ingress and egress filtering. This is for a datbase server which is at the ip address `192.168.234.164`. This is also for a server that listens on both HTTP (port `80`) and HTTPS (port `443`). If you aren't using one of the three, you can leave it out"

```
$	sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
$	sudo iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
$	sudo iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --sport 3306 -s 192.168.234.164 -j ACCEPT
$	sudo iptables -A OUTPUT -p tcp --dport 3306 -d 192.168.234.164 -j ACCEPT
```

and just block everything else:

```
$	sudo iptables -P INPUT DROP
$	sudo iptables -P OUTPUT DROP
$	sudo iptables -P FORWARD DROP
$	sudo ip6tables -P INPUT DROP
$	sudo ip6tables -P OUTPUT DROP
$	sudo ip6tables -P FORWARD DROP
```

### MySQL

For locking down mysql, ensure that the user you are using to log into mysql has a strong user, and only has access to the database for the web app (check MySQL documentation for how to do that). To find the files which have the mysql user creds, you can just grep for the username in the web directory:
```
#	grep -iR "silent" /var/www/html/ | grep database
/var/www/html/app/cache/prod/appProdProjectContainer.php:            'database_user' => 'silent',
/var/www/html/app/cache/prod/appParameters.php:    'database_user' => 'silent',
/var/www/html/app/cache/dev/appParameters.php:    'database_user' => 'silent',
/var/www/html/app/cache/dev/appDevDebugProjectContainer.php:            'database_user' => 'silent',
/var/www/html/app/config/parameters.php:    'database_user' => 'silent',
```

for this instance, I only had to change the file `/var/www/html/app/config/parameters.php`

## Migration

This Migration is to migrate the site over to an Ubuntu 16.04.3 enviornment with LAMP installed (and the php extensions mentioned at the top)

This will essentially involve copying over the content of the webapp (documents in the web directory), and changing the MySQL settings (both on the WebServer and the MySQL server) to reflect the needed changes.

### MySQL

First you will need to create a new user for the new server (or you could just edit the existing user). first login to MySQL:
```
#	mysql -u root -p
```

```
disable_functions = exec, passthru, shell_exec, system, proc_open, curl, exec, curl_multi_exec, parse_ini,file, show_source, pcntrl_exec, eval, assert, create_function, include, include_once, require, require_once, phpinfo, pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
```

Then create the new user `horror`, which will be logging in from `%` (in mysql, this means any IP address, we want it this way because we will be changing the ip address of the server we are migrating to shortly), with the password `password`:
```
MariaDB [(none)]> create user 'horror'@'%' identified by 'password';
Query OK, 0 rows affected (0.01 sec)
```

Next grant the user privileges, then flush privileges:
```
MariaDB [(none)]> grant all privileges on horrors.* to 'horror'@'%';
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```

then from the server that you are migrating to, make sure you can log in and see the database:
```
#	mysql -u horror -p -h 192.168.234.155
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 10.1.26-MariaDB-0+deb9u1 Debian 9.1

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| horrors            |
| information_schema |
+--------------------+
2 rows in set (0.00 sec)
```

So we can see here that we were able to login and see the database for the webapp `horrors`, so the test was sucessful

### Copy Documents 

Next we will need to copy the actual webapp itself. Keep in mind, we are copying from an apache server where the default web directory is `/var/www/html` to nginx where the default web directory is `/usr/share/nginx/html`. This command is ran from the server we are migrating to:
```
#	scp -rp guyinatuxedo@192.168.234.155:/var/www/html/* /var/www/html/
```

You will need to make the user `www-data` the owner of those directories/documents:
```
#	chown -R www-data:www-data /var/www/html/
```

Next we will need to edit the config of the webapp to reflect the new MySQL creds. First find the conf files which are relevant with this grep command (replace 'horrors' with the name of the database):
```
#	grep -iR "horrors" /var/www/html/
/var/www/html/app/cache/prod/appParameters.php:    'database_name' => 'horrors',
/var/www/html/app/config/parameters.php:    'database_name' => 'horrors',
```

```
#	vim /var/www/html/app/config/parameters.php
```

change these settings as needed:
```
<?php return array (
  'parameters' => 
  array (
    'database_host' => '192.168.234.155',
    'database_port' => '',
    'database_name' => 'horrors',
    'database_user' => 'horror',
    'database_password' => 'Holiday7',
```

### Change IP

The last thing that we will need to do is change the IP address. If we don't do this, then not only will service checks not go to this box, however the web app Prestashop points to this address with a lot of links, so we would need to either change all of those, or redirect those to match the new IP address:

To change  the ip address in Ubuntu, edit this file:
```
$	sudo vim /etc/network/interfaces
```


add/edit these settings in as needed. This setup is for the adapter `ens33`:
```
auto ens33
iface ens33 inet static
        address 192.168.234.162
        netmask 255.255.255.0
        network 192.168.234.0
        broadcast 192.168.234.255
        gateway 192.168.234.1
        dns-nameserver 8.8.8.8
```

after that you should just have to bring the interface down, then back up with the new IP address:
```
$	sudo ifdown ens33
$	sudo ifup ens33
```

after that, you should bed good to go. You will probably want to lock it down using the security guide above.

## Logging

For Logs, your best bet would be to check the apache/nginx/php/php-fpm logs. Look at the corresponding guides for those things form more details.
