# Roundcube Mail

## Installation

First install `postfix` fro SMTP, and `dovecot` for POP3. Durring the Postfix installation select `Internet Site` and input the name of the mail server:
```
$	sudo apt-get install postfix postfix-mysql dovecot-core dovecot-pop3d dovecot-imapd dovecot-lmtpd dovecot-mysql
```

#### DNS Records

You will need to create an MX and A record for the mail server. Here are some sample records:
```
; MX Record
hopes.end.      IN      MX      10 charisma.hopes.end.

;A Record
charisma        IN      A       192.168.234.186
```

#### Postfix

If when you installed `postfix`, you did select `Internet Site` and inputed the name of the mail server, these are all of the config changes you should need to make:
```
#	vi /etc/postfix/main.cf 
```

add/edit in these settings:
```
inet_protocols = ipv4
mydestination = hopes.end, charisma.hopes.end, charisma, localhost.localdomain, localhost
```

```
$	sudo service postfix start
 * Starting Postfix Mail Transport Agent postfix                                              [ OK ] 
```

#### Roundcube php

first install pear:
```
$	sudo apt-get install php-pear
```

you will need to use pear to install an updated version of itself:
```
$	pear install PEAR-1.10.1
```
Next you will need to use pear to install the following 

```
$	sudo pear install Mail_mime Mail_mimeDecode Auth_SASL Net_SMTP Net_IDNA2-0.2.0
```

You will also need to configure the `date.timezone` setting:
```
$	sudo vim /etc/php5/apache2/php.ini 
```

add/edit in this setting:
```
date.timezone = "US/Eastern"
```

make sure to restart apache:
```
$	sudo service apache2 restart
```

#### Dovecot

you need to configure dovecot to allow for plaintext authentication by editing this file:

```
$   sudo vim /etc/dovecot/conf.d/10-auth.conf
```

add/edit in the following configs:

```
disable_plaintext_auth = no
auth_mechanisms = plain login
```

next you will need to configure the mailbox location, and the group which the mail process will run as by editing this file:

```
$   sudo vim /etc/dovecot/conf.d/10-mail.conf
```

add/edit in the following configs:

```
mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_privileged_group = mail
mail_access_groups = mail
```

unless you plain on incorporating it into your mail config (I don't) you should disable SSL, since it will cause errors later on. To do that, edit this file:
```
$	sudo sudo vim /etc/dovecot/conf.d/10-ssl.conf 
```

add/edit in the following changes:
```
ssl = no
```

also you may want to create this file, which will store a SSL key. If it doesn't exist and Dovecot tries to open it, you will get errors:
```
$	sudo touch /etc/dovecot/private/dovecot.pem
```

after that you can start the `dovecot` service:
```
$	sudo service dovecot start
dovecot start/pre-start, process 5842
```

#### Lamp

```
$	sudo apt-get install apache2 php5 php-pear mysql-client php-mysql
```

#### Roundcube Setup

```
$	cd/tmp/
$	wget https://github.com/roundcube/roundcubemail/releases/download/1.1-beta/roundcubemail-1.1-beta.tar.gz
$	tar -zxvf roundcubemail-1.1-beta.tar.gz 
$	sudo cp -avr roundcubemail-1.1-beta-dep/* /var/www/html/
$	sudo chown -R www-data:www-data /var/www/html/
```

```
$	sudo service apache2 start
 * Starting web server apache2                                                                        * 
```

## AD Authentication Setup

First you will need to install `winbind` and `samba`:

```
#	apt-get install winbind samba smbclient libnss-winbind libpam-winbind heimdal-clients
```
Next you will need to edit the samba config file:

```
#	vim /etc/samba/smb.conf 
```

add/edit in the following settings. This is for the domain `unity.start`, with the domian controller at `192.168.234.187`:

```
[global]
        security = ads
        realm = unity.start
        password server = 192.168.234.187
        workgroup = unity
        winbind separator = +
        idmap uid = 10000-20000
        idmap gid = 10000-20000
        winbind enum users = yes
        winbind enum groups = yes
        template homedir = /home/%D/%U
        template shell = /bin/bash
        client use spnego = yes
        client ntlmv2 auth = yes
        encrypt passwords = yes
        restrict anonymous = 2
```

after that you need to restart the `winbind` and `samba` domains:

```
#	/etc/init.d/winbind stop
#	/etc/init.d/samba restart
#	/etc/init.d/winbind start
```

proceeding that you can test your configuration by acquiring a ticket:

```
#	kinit unity@unity.start
unity@unity.start's Password: 
```

If your time on the AD server and the linux server is off, you may get this error:

```
kinit: krb5_get_init_creds: Requested effective lifetime is negative or too short
```

If you do get the error, you can just set the time on the server to math the time on the ad server:

```
date --set "29 March 2018 16:01:00"
Thu Mar 29 16:01:00 EDT 2018
```

Proceeding that you can join the computer to the domain with the domain `unity` (sepcified with the `-U` flag) and the server `unity` (specified with `-S` flag). Check if the computer has been added to the domain from the AD server itself (even though there was an error, it still succesffuly added itself):

```
#	net ads join -U unity -S unity
Enter unity's password:
gss_init_sec_context failed with [ Miscellaneous failure (see text): Server (ldap/unity@UNITY.START) unknown]
kinit succeeded but ads_sasl_spnego_gensec_bind(KRB5) failed: An internal error occurred.
Failed to join domain: failed to connect to AD: An internal error occurred
```

after that we need to edit this file to configure winbind authentication:

```
#	vim /etc/nsswitch.conf
```

add/edit in these settings:

```
passwd:         compat winbind
group:          compat winbind
shadow:         compat winbind
```

then you can see the users/groups which have been pulled from AD:

```
wbinfo -u
wbinfo -g
```

After that you need to update pam using this. Select all of the options:

```
#	pam-auth-update
```

Lastly we will need to this pam config file:

```
#	vim /etc/pam.d/common-password 
```

remove this:
```
use_authtok
```

from this line:
```
password        [success=1 default=ignore]      pam_winbind.so use_authtok try_first_pass
```

##### Dovecot

Edit this file:
```
#	vim /etc/dovecot/conf.d/10-auth.conf
```

add/edit in this setting:
```
auth_realms = unity.start
```

## Security

#### Apache

##### Apache's User

You should check what user apache is runnin as (on this system it should be www-data). To do that, check this config file for Apache's enviornment variables:

```
#	cat /etc/apache2/envvars | grep USER
export APACHE_RUN_USER=www-data
#	cat /etc/apache2/envvars | grep GROUP
export APACHE_RUN_GROUP=www-data
```

here we can see that the enviornment variable APACHE_RUN_USER and APACHE_RUN_GROUP is set equal to www-data, which is what it should be. Now let's just make sure that apache is actually using those variables:

```
#	cat /etc/apache2/apache2.conf | grep User
User ${APACHE_RUN_USER}
#	cat /etc/apache2/apache2.conf | grep Group
Group ${APACHE_RUN_GROUP}
```

Here we confirmed that the apache service is indeed running as the correct users.

##### Directory Options

To configure those options, edit the main apache config file:
```
$	sudo vim /etc/apache/apache.conf
```

You should comment out directories that are not being used. In addition to that you should include the `-Indexes` and `-Includes` options to disable directory listing and  file inclusion. In addition to that, we limit the possible requests to `GET`, `POST`, and `HEAD`:

```
<Directory />
        Options -Includes -Indexes
        AllowOverride None
        Require all denied
</Directory>

<Directory /var/www/>
        Options -Indexes -Includes
        AllowOverride None
        Require all granted
        <LimitExcept GET POST HEAD>
        deny from all
        </LimitExcept>
</Directory>
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

##### Server Version

By default, apache will display a banner which prints out the server version apache is running, which can be helpful for an attacker to figure out which metasploit mod*ule to use to pwn you. To remove this, edit+*-+*+ the main apache conf file:

```
#   vim /etc/apache2/apache2.conf
```

add/edit in the following settings:

```
ServerTokens Prod
ServerSignature Off
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
Header set X-Frame-O+-+ptions: "sameorigin"
```
#### PHP

All of these options can be found in the following file:
```
#	vim /etc/php5/apache2/php.ini
```

##### Blacklist PHP functions

You should blacklist unnecissary php functions, that way it will 

```
disable_functions = exec, passthru, shell_exec, system, proc_open, curl, exec, curl_multi_exec, parse_ini,file, show_source, pcntrl_exec, eval, include, include_once, require, require_once, phpinfo, pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
```

##### Remote File Inclusion


You should disable allow_url_fopen and allow_url_include, since they can be used by an attacker to access files on the server, which they can use in exploits (Remote File Inclusion):

```
allow_url_fopen = Off
allow_url_include = Off
```

##### Disable File Uploads

To disable file uploads (which will disable an attacker from uploading things such as web shell) add/edit in these settings:

```
file_uploads = On
upload_max_filesize = 1M
max_file_uploads = 1
```

##### Set Base Dir

You can limit the file operations of php to a defined directory (or directories), which can limit what an attacker can do. We should limit it to the directories which Roundcube needs to run:

```
open_basedir = /var/www/html/:/usr/share/:/usr/bin/:/usr/src/
```

##### Don't Expose PHP

PHP can expose that it is installed on a server, which will be potentially helpful to an attacker. To disable it add/edit in this conf:
```
expose_php = Off
```
##### Limit Post Requests Size

Depending on what you are doing, you should probably limit the size of post requests which PHP will take, since this will help prevent certain type of DOS attacks:

```
post_max_size = 512K
```

##### Check Resources (memory usage)

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

#### Postfix

For this, you are going to check two settings (I'm not entirely sure on what the email server will need to send email to, ao that will affect what is done here). Edit this file:

```
#	vim /etc/postfix/main.cf
```

Check these settings, if you can figure out where the mail server needs to send mail to then edit the configs accordingly:
```
mynetworks = 127.0.0.0/8
relay_domains = hopes.end unity.start
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

For the service-auth process, it should have this configured as it's user:

```
service auth {
  user = $default_internal_user
}
```

For the service-worker process, it should have this configured as it's user and group (the default user is root):

```
service auth-worker {
  user = $default_internal_user
}
```

#### Roundcube

##### Webshells

For Roundcube your biggest threat is going to be php shells. To find them, what you can do is you can use grep to search through all of the files in the web directory for certian functions which are common in php shells:

Search for the `exec` function:
```
#	grep -iR "exec*(" /var/www/html/  | grep php
./plugins/password/drivers/virtualmin.php:        exec("$curdir/chgvirtualminpasswd modify-user --domain $domain --user $username --pass $newpass", $output, $returnvalue);
./plugins/password/drivers/plesk.php:        $retval = curl_exec($this->curl);
./plugins/password/drivers/domainfactory.php:            if ($result = curl_exec($ch)) {
./plugins/password/drivers/domainfactory.php:                if ($result = curl_exec($ch)) {
./plugins/password/drivers/dbmail.php:        exec("$curdir/chgdbmailusers -c $username -w $newpass $args", $output, $returnvalue);
./program/include/rcmail_output_html.php:                        if (preg_match('/Revision:\s(\d+)/', @shell_exec('svn info'), $regs))
./program/include/rcmail_output_html.php:                        if (preg_match('/Date:\s+([^\n]+)/', @shell_exec('git log -1'), $regs)) {
./program/lib/Roundcube/rcube_imap.php:            $results = $searcher->exec(
./program/lib/Roundcube/rcube_imap.php:            $index = $searcher->exec($folder, $str, $this->default_charset);
./program/lib/Roundcube/rcube_imap_search.php:    public function exec($folders, $str, $charset = null, $sort_field = null, $threading=null)
./program/lib/Roundcube/rcube_image.php:                        $result = rcube::exec($convert . ' 2>&1 -flatten -auto-orient -colorspace sRGB -strip'
./program/lib/Roundcube/rcube_image.php:            $result = rcube::exec($convert . ' 2>&1 -colorspace sRGB -strip -quality 75 {in} {type}:{out}', $p);
./program/lib/Roundcube/rcube_image.php:            $id   = rcube::exec($cmd. ' 2>/dev/null -format {format} {in}', $args);
./program/lib/Roundcube/rcube_spellcheck_pspell.php:        exec('aspell dump dicts', $dicts);
./program/lib/Roundcube/rcube_db_sqlite.php:                $q = $dbh->exec($data);
./program/lib/Roundcube/rcube.php:    public static function exec(/* $cmd, $values1 = array(), ... */)
./program/lib/Roundcube/rcube.php:        return (string)shell_exec($cmd);
./program/lib/Roundcube/rcube_utils.php:            $password = rtrim(shell_exec($command));
./program/lib/Roundcube/rcube_utils.php:            if (rtrim(shell_exec($command)) !== 'OK') {
./program/lib/Roundcube/rcube_utils.php:            $password = rtrim(shell_exec($command));
```

search for the `shell_exec` function:
```
#	grep -iR "shell_exec*(" /var/www/html  | grep php
/var/www/html/program/include/rcmail_output_html.php:                        if (preg_match('/Revision:\s(\d+)/', @shell_exec('svn info'), $regs))
/var/www/html/program/include/rcmail_output_html.php:                        if (preg_match('/Date:\s+([^\n]+)/', @shell_exec('git log -1'), $regs)) {
/var/www/html/program/lib/Roundcube/rcube.php:        return (string)shell_exec($cmd);
/var/www/html/program/lib/Roundcube/rcube_utils.php:            $password = rtrim(shell_exec($command));
/var/www/html/program/lib/Roundcube/rcube_utils.php:            if (rtrim(shell_exec($command)) !== 'OK') {
/var/www/html/program/lib/Roundcube/rcube_utils.php:            $password = rtrim(shell_exec($command));
```

search for the `base64_decode` function:
```
#	grep -iR "base64_decode*(" /var/www/html  | grep php
/var/www/html/plugins/database_attachments/database_attachments.php:            $args['data'] = base64_decode($data);
/var/www/html/plugins/redundant_attachments/redundant_attachments.php:            $args['data'] = base64_decode($data);
/var/www/html/program/lib/Roundcube/rcube_charset.php:                $res .= self::utf16_to_utf8(base64_decode($ch));
/var/www/html/program/lib/Roundcube/rcube_imap_generic.php:                    md5($this->_xor($pass, $ipad) . base64_decode($challenge))));
/var/www/html/program/lib/Roundcube/rcube_imap_generic.php:                    base64_decode($challenge), $this->host, 'imap', $user));
/var/www/html/program/lib/Roundcube/rcube_imap_generic.php:                $challenge = base64_decode($challenge);
/var/www/html/program/lib/Roundcube/rcube_imap_generic.php:                        $result = base64_decode($result);
/var/www/html/program/lib/Roundcube/rcube_imap_generic.php:                        $line = base64_decode($line);
/var/www/html/program/lib/Roundcube/rcube_session.php:            $this->vars      = base64_decode($sql_arr['vars']);
/var/www/html/program/lib/Roundcube/rcube_ldap.php:        return base64_decode($str);
/var/www/html/program/lib/Roundcube/rcube.php:        $cipher = $base64 ? base64_decode($cipher) : $cipher;
/var/www/html/program/lib/Roundcube/rcube_mime.php:                        $text .= base64_decode($tmp[$i]);
/var/www/html/program/lib/Roundcube/rcube_mime.php:            return base64_decode($input);
/var/www/html/program/lib/Roundcube/rcube_vcard.php:            return base64_decode($value);
/var/www/html/program/lib/Roundcube/rcube_db.php:            return @unserialize(base64_decode($input));
```

Another good way of detecting webshells is to look at the timestamps of all files in the web directory. Chances are the web shell was uploaded after everything else so it should have a later time than most files in the web directory. First find the timestamp which the files in the web directory were written to:
```
#	ls -ltcr /var/www/html/
total 232
-rw-r--r--  1 www-data www-data   2974 Mar 28 11:41 UPGRADING
-rw-r--r--  1 www-data www-data     26 Mar 28 11:41 robots.txt
-rw-r--r--  1 www-data www-data   4031 Mar 28 11:41 README.md
-rw-r--r--  1 www-data www-data  35147 Mar 28 11:41 LICENSE
drwxr-xr-x  3 www-data www-data   4096 Mar 28 11:41 installer
-rw-r--r--  1 www-data www-data   9246 Mar 28 11:41 INSTALL
-rw-r--r--  1 www-data www-data   1556 Mar 28 11:41 composer.json-dist
drwxr-xr-x  2 www-data www-data   4096 Mar 28 11:41 temp
drwxr-xr-x  4 www-data www-data   4096 Mar 28 11:41 skins
drwxr-xr-x  8 www-data www-data   4096 Mar 28 11:41 program
drwxr-xr-x 34 www-data www-data   4096 Mar 28 11:41 plugins
-rw-r--r--  1 www-data www-data  13371 Mar 28 11:41 index.php
-rw-r--r--  1 www-data www-data 118726 Mar 28 11:41 CHANGELOG
drwxr-xr-x  2 www-data www-data   4096 Mar 28 11:41 bin
drwxr-xr-x  6 www-data www-data   4096 Mar 28 11:41 SQL
drwxr-xr-x  2 www-data www-data   4096 Mar 28 12:22 logs
drwxr-xr-x  2 www-data www-data   4096 Mar 31 04:21 config
```

Here we can see that the average time stamp for all of the files in the web directory is `11:41`. We can do a grep on the entire web directory for all files that don't have that timestamp:

```
#	ls -ltcrR | grep -v "11:41"
./logs:
total 80
-rw-r--r-- 1 www-data www-data 65899 Mar 31 04:33 errors
-rw-r--r-- 1 www-data www-data  4343 Mar 31 04:35 sendmail

./config:
total 60
-rw-r--r-- 1 www-data www-data 46727 Mar 31 04:21 defaults.inc.php
-rw-r--r-- 1 www-data www-data  2059 Mar 31 04:21 config.inc.php
```

the last thing that should be effective at finding webshells is tailing the apache access logs. Essentially you keep an eye on what clients are accessing, and if there is anything suspicious you invistigate:
```
#	tail -f /var/log/apache2/access.log 
```

user logging in:
```
192.168.234.1 - - [31/Mar/2018:17:05:21 -0400] "POST /?_task=login HTTP/1.1" 302 758 "http://192.168.234.186/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:05:21 -0400] "GET /?_task=mail HTTP/1.1" 200 8700 "http://192.168.234.186/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:05:21 -0400] "GET /?_task=mail&_action=getunread&_remote=1&_unlock=0&_=1522519522246 HTTP/1.1" 200 622 "http://192.168.234.186/?_task=mail&_mbox=INBOX" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:05:21 -0400] "GET /?_task=mail&_refresh=1&_mbox=INBOX&_action=list&_remote=1&_unlock=loading1522519522347&_=1522519522245 HTTP/1.1" 200 1664 "http://192.168.234.186/?_task=mail" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

user sending email:
```
192.168.234.1 - - [31/Mar/2018:17:36:07 -0400] "GET /?_task=mail&_mbox=INBOX&_action=compose HTTP/1.1" 302 527 "http://192.168.234.186/?_task=mail&_mbox=INBOX" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:36:07 -0400] "GET /?_task=mail&_action=compose&_id=13137163815abfff476f654 HTTP/1.1" 200 8310 "http://192.168.234.186/?_task=mail&_mbox=INBOX" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:36:13 -0400] "POST /?_task=mail&_action=autocomplete HTTP/1.1" 200 640 "http://192.168.234.186/?_task=mail&_action=compose&_id=13137163815abfff476f654" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:36:22 -0400] "POST /?_task=mail&_unlock=loading1522521382884&_lang=en_US&_framed=1 HTTP/1.1" 200 884 "http://192.168.234.186/?_task=mail&_action=compose&_id=13137163815abfff476f654" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:36:23 -0400] "GET /?_task=mail&_refresh=1&_mbox=INBOX HTTP/1.1" 200 8725 "http://192.168.234.186/?_task=mail&_action=compose&_id=13137163815abfff476f654" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:36:23 -0400] "GET /?_task=mail&_action=getunread&_remote=1&_unlock=0&_=1522521383643 HTTP/1.1" 200 609 "http://192.168.234.186/?_task=mail&_mbox=INBOX" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:36:23 -0400] "GET /?_task=mail&_refresh=1&_mbox=INBOX&_action=list&_remote=1&_unlock=loading1522521383726&_=1522521383642 HTTP/1.1" 200 1702 "http://192.168.234.186/?_task=mail&_refresh=1&_mbox=INBOX" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

user reading an email:
```
192.168.234.1 - - [31/Mar/2018:17:37:06 -0400] "GET /?_task=mail&_uid=33&_mbox=INBOX&_action=pagenav&_remote=1&_unlock=loading1522521426878&_=1522521426817 HTTP/1.1" 200 694 "http://192.168.234.186/?_task=mail&_action=show&_uid=33&_mbox=INBOX&_caps=pdf%3D0%2Cflash%3D1%2Ctif%3D0" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:17:37:06 -0400] "GET /?_task=mail&_action=getunread&_remote=1&_unlock=0&_=1522521426818 HTTP/1.1" 200 609 "http://192.168.234.186/?_task=mail&_action=show&_uid=33&_mbox=INBOX&_caps=pdf%3D0%2Cflash%3D1%2Ctif%3D0" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

a user logging off:
```
192.168.234.1 - - [31/Mar/2018:17:37:32 -0400] "GET /?_task=logout HTTP/1.1" 200 2969 "http://192.168.234.186/?_task=mail&_action=show&_uid=33&_mbox=INBOX&_caps=pdf%3D0%2Cflash%3D1%2Ctif%3D0" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

#### IPTables

These are the things that we will be allowing inbound/outbound:
*	`22` For SSH
*	`80`	For Roundcube Webapp
*	`3306`-cleint	For MySQL db at `192.168.234.185`
*	`143`-server/client	So Roundcube can use IMAP
*	`25`-server/clinet	So the mail server can send/receiev email, and can talk to itself
*	`53`-client			So the mail server can query DNS
*	`110`-server/client	So the server can use POP3, as both a server and a client
*	all traffic to-from domain controller : Winbind uses random ports (in addition to port `445`) to talk to the domain controller 
*	all traffic to-from the mail server : The way the apps on the mail server talk to each other might be different than our practice enviornment, so this should account for those:

IP's:
*	`192.168.234.185`:	DB
*	`192.168.234.185`:	DNS
*	`192.168.234.186`:	MAIL
*	`192.168.234.187`:	AD

Rules for ingress filtering:
```
#	iptables -A INPUT -p tcp --dport 22 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 3306 -s 192.168.234.185 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 143 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 143 -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 143 -s 192.168.234.186 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 25 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 25 -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 25 -s 192.168.234.186 -j ACCEPT
#	iptables -A INPUT -p tcp --dport 110 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 110 -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -p tcp --sport 110 -s 192.168.234.186 -j ACCEPT
#	iptables -A INPUT -p udp --sport 53 -s 192.168.234.185 -j ACCEPT
#	iptables -A INPUT -s 192.168.234.187 -j ACCEPT
#	iptables -A INPUT -s 127.0.0.1 -j ACCEPT
#	iptables -A INPUT -s 192.168.234.186 -j ACCEPT
```

Rules for egress filtering:
```
#	iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 3306 -d 192.168.234.185 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 143 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 143 -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 143 -d 192.168.234.186 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 25 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 25 -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 25 -d 192.168.234.186 -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 110 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 110 -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 110 -d 192.168.234.186 -j ACCEPT
#	iptables -A OUTPUT -d 192.168.234.187 -j ACCEPT
#	iptables -A OUTPUT -p udp --dport 53 -d 192.168.234.185 -j ACCEPT
#	iptables -A OUTPUT -d 127.0.0.1 -j ACCEPT
#	iptables -A OUTPUT -d 192.168.234.186 -j ACCEPT
```

For ipv6, you will want to allow everything in/out of the loopback address (just in case the network config is different than what is expected):
```
#	ip6tables -A INPUT -s ::1/128 -j ACCEPT
#	ip6tables -A OUTPUT -d ::1/128 -j ACCEPT
```

For the rest you can just block all of it:

```
#	iptables -P INPUT DROP
#	iptables -P OUTPUT DROP
#	iptables -P FORWARD DROP
#	ip6tables -P INPUT DROP
#	ip6tables -P OUTPUT DROP
#	ip6tables -P FORWARD DROP
```
after that to make your iptables rules persistent across reboots you will need to save your iptables rules for both IPv4 and IPv6 to a conf file:

```
#	iptables-save > /etc/iptables.conf
#	ip6tables-save > /etc/ip6tables.conf
```


Next we just need to add a command to the rc.local file which will essentially import the two conf files we just made whenever the box starts up:
```
#	vim /etc/rc.local
```
Here are the commands that you need to input:
```
#	iptables-restore < /etc/iptables.conf
#	ip6tables-restore < /etc/ip6tables.conf
```

#### Active Directory Users

To see all users which have been added through AD:
```
#	wbinfo -u
```

To see all groups which have been added through AD:
```
#	wbinfo -g
```

you can change the password of the user `wrath` (domain is nicknamed `unity`) a user using `passwd`:
```
#	passwd unity+wrath
```

## Logs

#### Postfix

Most of your postfix logs can be found under `syslog`:
```
#	cat /var/log/syslog | grep postfix
```

Here is an example of a sent email:
```
Mar 31 04:34:23 charisma postfix/pickup[1729]: 8337B1096: uid=33 from=<guyinatuxedo@charisma.hopes.end>
Mar 31 04:34:23 charisma postfix/cleanup[2025]: 8337B1096: message-id=<5c7302ad99461ac4b617d99ee122ffd4@charisma.hopes.end>
Mar 31 04:34:23 charisma postfix/qmgr[1233]: 8337B1096: from=<guyinatuxedo@charisma.hopes.end>, size=595, nrcpt=1 (queue active)
Mar 31 04:34:23 charisma postfix/local[2028]: 8337B1096: to=<guy@hopes.end>, relay=local, delay=0.03, delays=0.01/0.01/0/0.01, dsn=2.0.0, status=sent (delivered to mailbox)
Mar 31 04:34:23 charisma postfix/qmgr[1233]: 8337B1096: removed
```

Here is an example of a bounced email:
```
Mar 31 04:35:11 charisma postfix/pickup[1729]: A910F1096: uid=33 from=<guyinatuxedo@charisma.hopes.end>
Mar 31 04:35:11 charisma postfix/cleanup[2025]: A910F1096: message-id=<0c77e6e436dcc57866442923a7f2f9f0@charisma.hopes.end>
Mar 31 04:35:11 charisma postfix/qmgr[1233]: A910F1096: from=<guyinatuxedo@charisma.hopes.end>, size=601, nrcpt=1 (queue active)
Mar 31 04:35:11 charisma postfix/local[2028]: A910F1096: to=<tux@hopes.end>, relay=local, delay=0.01, delays=0/0/0/0.01, dsn=5.1.1, status=bounced (unknown user: "tux")
Mar 31 04:35:11 charisma postfix/cleanup[2025]: ABAF01098: message-id=<20180331083511.ABAF01098@charisma>
Mar 31 04:35:11 charisma postfix/qmgr[1233]: ABAF01098: from=<>, size=2265, nrcpt=1 (queue active)
Mar 31 04:35:11 charisma postfix/bounce[2044]: A910F1096: sender non-delivery notification: ABAF01098
Mar 31 04:35:11 charisma postfix/qmgr[1233]: A910F1096: removed
Mar 31 04:35:11 charisma postfix/local[2028]: ABAF01098: to=<guyinatuxedo@charisma.hopes.end>, relay=local, delay=0.01, delays=0/0/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
Mar 31 04:35:11 charisma postfix/qmgr[1233]: ABAF01098: removed
```

#### Dovecot

Dovecot logs in two main places by default. To view logs related to Dovecot running (things like authenticating users):
```
#	cat /var/log/syslog | grep dovecot
```

example of a successful Dovecot login:
```
Mar 31 04:25:19 charisma dovecot: imap-login: Login: user=<guyinatuxedo>, method=PLAIN, rip=192.168.234.186, lip=192.168.234.186, mpid=1882, secured, session=<pijcGLFomgDAqOq6>
Mar 31 04:25:19 charisma dovecot: imap(guyinatuxedo): Disconnected: Logged out in=82 out=550
Mar 31 04:25:19 charisma dovecot: imap-login: Login: user=<guyinatuxedo>, method=PLAIN, rip=192.168.234.186, lip=192.168.234.186, mpid=1884, secured, session=<qdXcGLFongDAqOq6>
Mar 31 04:25:19 charisma dovecot: imap(guyinatuxedo): Disconnected: Logged out in=44 out=481
Mar 31 04:25:19 charisma dovecot: imap-login: Login: user=<guyinatuxedo>, method=PLAIN, rip=192.168.234.186, lip=192.168.234.186, mpid=1888, secured, session=<GyvhGLFopADAqOq6>
Mar 31 04:25:19 charisma dovecot: imap(guyinatuxedo): Disconnected: Logged out in=119 out=615
Mar 31 04:25:19 charisma dovecot: imap-login: Login: user=<guyinatuxedo>, method=PLAIN, rip=192.168.234.186, lip=192.168.234.186, mpid=1889, secured, session=<pG/hGLFopgDAqOq6>
Mar 31 04:25:19 charisma dovecot: imap(guyinatuxedo): Disconnected: Logged out in=294 out=15948
```

Dovecot also has a log file dealing with startup errors for Dovecot. To view that log file:
```
#	cat upstart/dovecot.log
```

example of an error where ldap settings are trying to be used with a version of Dovecot that doesn't support ldap:
```
doveconf: Fatal: Error in configuration file /etc/dovecot/conf.d/auth-ldap.conf.ext line 35: Unknown setting: hosts
```

#### Apache

With apache there are two logs that you mainly need to worry about, the `error` log and the `access` log.

The `access` logs all requests sent to the webserver:
```
#	cat /var/log/apache2/access.log
```

here is an example of a user logging into Roundcube:
```
192.168.234.1 - - [31/Mar/2018:04:40:51 -0400] "POST /?_task=login HTTP/1.1" 302 758 "http://192.168.234.186/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:04:40:51 -0400] "GET /?_task=mail HTTP/1.1" 200 8702 "http://192.168.234.186/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:04:40:51 -0400] "GET /?_task=mail&_action=getunread&_remote=1&_unlock=0&_=1522515559621 HTTP/1.1" 200 622 "http://192.168.234.186/?_task=mail&_mbox=INBOX" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [31/Mar/2018:04:40:51 -0400] "GET /?_task=mail&_refresh=1&_mbox=INBOX&_action=list&_remote=1&_unlock=loading1522515559704&_=1522515559620 HTTP/1.1" 200 1665 "http://192.168.234.186/?_task=mail" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

The `error` log contains all errors (since php is an Apache module, it includes php errors too):
```
#	cat /var/log/apache2/error.log
```

an example of PHP not being able to access the `PEAR` class due to a misconfigured php base directory:
```
[Fri Mar 30 05:13:48.240604 2018] [:error] [pid 2198] [client 192.168.234.1:49276] PHP Fatal error:  Class 'PEAR' not found in /var/www/html/program/lib/Roundcube/bootstrap.php on line 100
```

#### Rouncube

Rouncube has a debug mode, which will essentially cause Rouncube to display PHP errors. To enable displaying php errors edit this file:
```
#	vim /var/www/html/config/defaults.inc.php
```

add/edit this setting to have the value `4` ('1' will log errors, `4` will display them):
```
$config['debug_level'] = 4;
```

Rouncube keeps some logs that are for things such as failed IMAP logins:
```
#	cat /var/www/html/logs/errors
```

failed login example:
```
[31-Mar-2018 04:13:52 -0400]: <m44lvvaf> IMAP Error: Login failed for guyinatuxedo from 192.168.234.1. AUTHENTICATE PLAIN: Authentication failed. in /var/www/html/program/lib/Roundcube/rcube_imap.php on line 198 (POST /?_task=login?_task=login&_action=login)
```

#### Auth

Logs dealing with who tried to (and whether they succeeded or failed) at logging in can be found here:
```
#	cat /var/log/auth.log
```

Here is an example of the AD user `UNITY+unity` successfully authenticating via SSH:
```
Mar 31 16:27:33 charisma sshd[1603]: pam_winbind(sshd:auth): getting password (0x00000388)
Mar 31 16:27:33 charisma sshd[1603]: pam_winbind(sshd:auth): pam_get_item returned a password
Mar 31 16:27:33 charisma sshd[1603]: pam_winbind(sshd:auth): user 'UNITY+unity' granted access
Mar 31 16:27:33 charisma sshd[1603]: pam_winbind(sshd:auth): getting password (0x00000000)
Mar 31 16:27:33 charisma sshd[1603]: pam_winbind(sshd:auth): user 'UNITY+unity' granted access
Mar 31 16:27:33 charisma sshd[1603]: pam_winbind(sshd:account): user 'UNITY+unity' granted access
Mar 31 16:27:33 charisma sshd[1603]: Accepted password for UNITY+unity from 192.168.234.1 port 57916 ssh2
Mar 31 16:27:33 charisma sshd[1603]: pam_unix(sshd:session): session opened for user UNITY+unity by (uid=0)
```

Here is an example of the AD user `UNITY+wrath` failling at authenticating via SSH:
```
Mar 31 16:39:10 charisma sshd[1661]: pam_winbind(sshd:auth): getting password (0x00000388)
Mar 31 16:39:10 charisma sshd[1661]: pam_winbind(sshd:auth): pam_get_item returned a password
Mar 31 16:39:11 charisma sshd[1661]: pam_winbind(sshd:auth): request wbcLogonUser failed: WBC_ERR_AUTH_ERROR, PAM error: PAM_AUTH_ERR (7), NTSTATUS: NT_STATUS_LOGON_FAILURE, Error message was: Logon failure
Mar 31 16:39:11 charisma sshd[1661]: pam_winbind(sshd:auth): user 'UNITY+wrath' denied access (incorrect password or invalid membership)
Mar 31 16:39:12 charisma sshd[1661]: Failed password for UNITY+wrath from 192.168.234.1 port 57970 ssh2
```
