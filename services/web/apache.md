# Apache

This is a guide on apache version 2.4.5 on Debian 9. If you are using a distro such as CentOS try replacing `apache2` with `httpd`.

## Installation

To install apache:
```
$	sudo apt-get install apache2
```

#### Setup SSL
First enable the `ssl` and `headers` mods:
```
#	a2enmod ssl
#	a2enmod headers
#	systemctl restart apache2
```

If you are on centos, you may need to install `mod_ssl`:
```
#	sudo yum install mod_ssl
```

Next we are going to need to generate a public certifcate (which is sent to the client), and a private key (which stays on the server) for our website. 
	
```
#	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```

Next you should create a strong Diffie-Hellman group, which will be used by SSL:

```
#	openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Next we can create a conf file for our new Virtual Host file:
```
#	vim /etc/apache2/sites-available/hell-vault.conf
```

should have the following content:
```
<VirtualHost 192.168.234.154:443>
	SSLEngine on
	SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
	SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

	ServerName hellvault.nz
	ServerAdmin silent-horror@endless.night.nz
	DocumentRoot /var/www/sh
	ServerName	silent-horror.nz
	ServerAlias	www.silent-horror.nz	
	
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	BrowserMatch "MSIE [2-6]" \
		nokeepalive ssl-unclean-shutdown \
		downgrade-1.0 force-response-1.0

</VirtualHost>
```

We can check the config with the following command:

```
#	apache2ctl configtest
```

Then we can enable the site, restart apache, and we will have our web server with SSL at `192.168.234.154` on port `443`:

```
#	a2ensite hell-vault.conf
#	systemctl reload apache2
```

#### Setup a password
Apache can password protect directories, so when you visit them in a web browser, you will have to authenticate to reach it. To set this up

First create a file which stores the password for the user `guy` (try to put it in Apache's running directory):
```
#	htpasswd -c /var/run/apache2/pwds guy
```

next create a `.htaccess` file in the web directory (or you could just include this in the `apache.conf` for the section pertainning to where your web root is):
```
#	vim /var/www/html/.htaccess
```

it should have the following values:
```
AuthType Basic
AuthName "Unspeakable Horrors"
AuthBasicProvider file
AuthUserFile "/var/run/apache2/pwds"
Require user guy
```

Now password authentication should be enabled for that directory. If this works, make sure `AllowOverride all` is enabled in the `apache.conf` portion pertainning to where the web directory is:


## Config

##### sites conf

The configuration for the websites which apache can host are typically stored in `/etc/apache2/sites-available` (these sites are not ). The conf files for the sites which are currently enabled are stored in `/etc/apache2/sites-enable`. Here is an example config for a site (known as VirtualHost):
```
<VirtualHost *:80>
	ServerAdmin silent-horror@endless.night.nz
	DocumentRoot /var/www/html
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

*	`</VirtualHost>`:	Specifies that this site is going to listen on all ip addresses, on port 80
*	`DocumentRoot`:	Specifies that the root directory for this site is `/var/www/html`
*	`ErrorLog`:		Specifies the log file which will store errors, which is `Apache_Log_Dir/error.log`
*	`CustomLog`:	Specifies the log file which will store requests, which is `Apache_Log_Dir/access.log`

```
<VirtualHost 192.168.234.153:8080>
	ServerAdmin silent-horror@endless.night.nz
	DocumentRoot /var/www/sh
	DirectoryIndex horror.php
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
	ServerName	silent-horror.nz
	ServerAlias	www.silent-horror.nz	
</VirtualHost>
```
*	`<VirtualHost 192.168.234.153:8080>`:	Specifies that this site is to listen on the ip address `192.168.234.153` and port `8080` (since `8080` is not standard, the line `Listen 8080` will need to be appended to `/etc/apache2/ports`)
*	`ServerName	silent-horror.nz`:		Specifies that the name of this host is `silent-horror.nz`, which will be used to route requests to this site
*	`ServerAlias	www.silent-horror.nz`:	Specifies that an alias (another name for this server) is `www.silent-horror.nz`
*	`DirectoryIndex horror.php`:	Specifies that the defaault document which is served to the user is `horror.php`


for the next config, we will need to enable some mods found in `/etc/apache2/mods-available/` with the following config:

```
$	sudo a2enmod proxy
$	sudo a2enmod proxy_html
$	sudo systemctl restart apache2
```

Here is the conf:
```
<VirtualHost 192.168.234.153:880>
	ProxyPreserveHost On
	ProxyPass	"/"	"http://192.168.234.153"
	ProxyPassReverse "/"	"http://192.168.234.153"

	ServerName lost-soul.nz
	ServerAdmin silent-horror@endless.night.nz
	ServerName	lost-soul.nz
	ServerAlias	www.lost-soul.nz	
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
*	`<VirtualHost 192.168.234.153:880>`:	Specifies that this site is to listen on the ip address `192.168.234.153` and port `880` (since `8080` is not standard, the line `Listen 8080` will need to be appended to `/etc/apache2/ports`)
*	`ProxyPreserveHost On`:	This setting is to perserve the original client header, while it is forging the new header to be sent to the web server when proxied
*	`ProxyPass	"/"	"http://192.168.234.153"`:	This maps the root directory of this host to the url `"http://192.168.234.153`
*	`ProxyPassReverse "/"	"http://192.168.234.153"`:	This lets Apache adjust the URL in the Location, Content-Location and URI headers on HTTP redirect responsed. This is important in making sure it doesn't bypass the reverse proxy

```
<VirtualHost 192.168.234.154:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

	SSLProtocol -ALL +TLSv1.2
	SSLHonorCipherOrder on
	SSLCipherSuite ALL:!aNULL:!eNULL:!EXP:!LOW:!MD5:!RC4:!SHA1:!DH

    ServerName hellvault.nz
    ServerAdmin silent-horror@endless.night.nz
    DocumentRoot /var/www/sh
    ServerName  silent-horror.nz
    ServerAlias www.silent-horror.nz    

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    BrowserMatch "MSIE [2-6]" \
        nokeepalive ssl-unclean-shutdown \
        downgrade-1.0 force-response-1.0

</VirtualHost>
```
*	`SSLEngine on`:	Specifies that SSL will be running on this virtual host
*	`SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt`:	Specifies that the certificate file (which is sent to the client as part of SSL) is `/etc/ssl/certs/apache-selfsigned.crt`
*	`SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key` Specifies that the corresponding private key for the certificate specified above is `/etc/ssl/private/apache-selfsigned.key`
*	`SSLProtocol -ALL +TLSv1.2`:	Specifies that the server is only to use TLS v1.2, which is currently the latest version of TLS
*	`SSLHonorCipherOrder on`:	Specifies that the server will honor the ciphers in this config file, which are speicifed in the next line
*	`SSLCipherSuite ALL:!aNULL:!eNULL:!EXP:!LOW:!MD5:!RC4:!SHA1:!DH`:	Specifies that this server can use all cipher algorithms, except `NULL`, `EXP`, `MD5`, and the other Ciphers mentioned (essentially it's a blacklist)
*	`BrowserMatch "MSIE [2-6]" \`: Specifies that if the client Browser is Internet Explorer versions 2 - 6, the following two lines apply (this is done for compatibillity purposes)
*	`nokeepalive ssl-unclean-shutdown \`:	This specifies that `KeepAlive` is disabled, and ssl is just to force a shutdown
*	`downgrade-1.0 force-response-1.0`:	This forces the client to downgrade to HTTP 1.0 

To enable the site with the conf file `/etc/apache2/sites-available/silent-horror.conf`:
```
$	sudo a2ensite silent-horror.conf
$	sudo systemctl reload apache2
```

To disable a site with the conf file `/etc/apache2/sites-enabled/lost-soul.conf`:
```
$	sudo a2dissite lost-soul.conf
$	sudo systemctl reload apache2
```

to check the apache configs for syntax errors:
```
#   apache2ctl configtest
```

##### mods

To enable the mod `data` for apache, located in `/etc/apache2/mods-avaliable`:
```
$	sudo a2enmod data
$	sudo systemctl restart apache2
```

To disable the mod `data` for apache:
```
$	sudo a2dismod data
$	sudo systemctl restart apache2
```

##### confs

To enable the conf `security` for apache, located in `/etc/apache2/conf-avaliable`:
```
$	sudo a2enconf security
$	sudo systemctl reload apache2
```

To disable the conf `security` for apache:
```
$	sudo a2disconf security
$	sudo systemctl reload apache2
```

##### apache.conf

This specifies the directory which apache will look for it's conf files (default `/etc/apache2`):

```
#ServerRoot "/etc/apache2"
```

To specify where to store Mutex files, which are used by Apache when it locks a file:
```
#Mutex file:${APACHE_LOCK_DIR} default
```

This option specifies that the Runtime Directory (which by default is set equal to the envvar `APACHE_RUN_DIR`). This is the directory which apache should interface with, if it needs to do things like write to the filesystem outside of the web directory, SELinux will block apache from writing to files outside of this directory:

```
DefaultRuntimeDir ${APACHE_RUN_DIR}
```

This option specifies the file which will hold the PID (Process ID) which Bind will use, which this value is stored in the `APACHE_PID_FILE` envvar:
```
PidFile ${APACHE_PID_FILE}
```

This option specifies the amount of seconds apache will wait for input/output before it kills the connection:
```
Timeout 300
```
The `KeepAlive` utillity allows for multiple requests to be sent over the same TCP connection:
```
KeepAlive On
```
This option specifies the maximum amount of requests per TCP connection for KeepAlive (if the value is 0, there is no limit):
```
MaxKeepAliveRequests 100
```

This specifies the amount of seconds Apache will wait on requests using KeepAlive before killing the conntection:
```
KeepAliveTimeout 5
```

This specifies the user and group which Apache will runs as, which is stored in the `APACHE_RUN_USER` and `APACHE_RUN_GROUP` envvars:
```
User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}
```
This setting is to perform a reverse DNS resolution (resolves for a PTR record) for every client's IP address:
```
HostnameLookups Off
```

This specifies the file for Apache Error Logs, which the Apache Log Directory is stored in the envvar `APACHE_LOG_DIR`:
```
ErrorLog ${APACHE_LOG_DIR}/error.log
```

This option specifies how verbose logs are (deug is most info, emerg is least, trace1-8 gets into logging for actual network communication with 8 being most verbose):
```
LogLevel warn
```
 
`AuthType` specifies the authentication type to be used, `Basic` simply just sends the password from the client to the server unencrypted
```
AuthType Basic
```

`AuthName` specifies the name of what you are authenticating to, when you authenticate to it
```
AuthName "Unspeakable Horrors"
```

`AuthBasicProvider` specifies how the password is stored, which in this case is a password:
```
AuthBasicProvider file
```

`AuthUserFile` specifies the file which stores the password for authentication
```
AuthUserFile "/var/run/apache2/pwds"
```

This specifies that the user `guy` can authenticate
```
Require user guy
```

These lines are to include all mods, which will load all files from `apache_dir/mods-enabled/` with the file extensions `.load` or `.conf`, will not throw an error if files are not found:
```
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf
```

This option specifes to include `ports.conf`:
```
Include ports.conf
```

sample ports.conf file:
```
#Specify main port
Listen 80

#Specify port for ssl/https
<IfModule ssl_module>
	Listen 443
</IfModule>

<IfModule mod_gnutls.c>
	Listen 443
</IfModule>
```

If you want to specify Apache to listen on the ip address `192.168.234.156`, just add/edit it in (by default on newer versions it listens on IPv6):
```
Listen 192.168.234.156:80
```

These options specify the directories which apache can/can't go. By default, Apache can browser the directories `/usr/share` and `/var/www`, however is banned from everything else:
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

<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

*	`<Directory /usr/share>`:	Specifies options for the file directory `/usr/share`
*	`Require all denied`:	Specifies that apache is to deny all access to the directory (`granted` if you want to grant access)
*	`Options Indexes FollowSymLinks`:		These options will allow Apache to follow symbolic links, and `Indexes` will return a listing of all files in a directory if you browse to a file path for a directory
*	`AllowOverride`:	This specifies if an `.htaccess` file will override existing configurations (if it is set to none, the file is not read)

This specifies the configuration file which apache will look for when responding to requests:
```
AccessFileName .htaccess
```

This option specifies that all files with `.ht` are to not be displayed to the end client (to prevent end users from seeing `.htaccess` files):
```
<FilesMatch "^\.ht">
        Require all denied
</FilesMatch>
```

These options just specify how Apache will format logs:
```
LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent
```

This will include all files under the `/apache_conf_dir/conf-enabled/` directory with the `.conf` file extension. In this directory stores conf files for various things, such as characters set, and security settings:
```
IncludeOptional conf-enabled/*.conf
```

This will include all files under the `/apache_conf_dir/sites-enabled/` directory with the `.conf` file extension. This directory is used to store conf files for websites that are enabled to run on your apache instance:
```
IncludeOptional sites-enabled/*.conf
```

##### Enviornment Variables:

The enviornment variables file can be found in the service file for apache, which on debian is /etc/init.d/apache2`. All of these are used in other conf files, and are used to set the value for things:

```
elif [ "${SCRIPTNAME##apache2-}" != "$SCRIPTNAME" ] ; then
	DIR_SUFFIX="-${SCRIPTNAME##apache2-}"
	APACHE_CONFDIR=/etc/apache2$DIR_SUFFIX
else
	DIR_SUFFIX=
	APACHE_CONFDIR=/etc/apache2
fi
if [ -z "$APACHE_ENVVARS" ] ; then
	APACHE_ENVVARS=$APACHE_CONFDIR/envvars
fi
export APACHE_CONFDIR APACHE_ENVVARS
```

So here, we can see that the file which contains apache enviornment variables is `/etc/apache2/envvars`:

This enviornment variable specifies what user Apache should be running as:
```
export APACHE_RUN_USER=www-data
```

This enviornment variable specifies what group Apache should be running as:
```
export APACHE_RUN_GROUP=www-data
``` 

This specifies the file which stores the PID (process ID) which apache will use (`/var/run/apache2/apache2.pid`)
```
export APACHE_PID_FILE=/var/run/apache2$SUFFIX/apache2.pid
```

This is the directory which apache should interface with, if it needs to do things like write to the filesystem outside of the web directory, SELinux will block apache from writing to files outside of this directory:
```
export APACHE_RUN_DIR=/var/run/apache2$SUFFIX
```

To specify the directory which Apache's Lock FIles (files that are used to ensure two processes don't edit the same file) will be stored:
```
export APACHE_LOCK_DIR=/var/lock/apache2$SUFFIX
```

This specifies the directory where apache stores it's logs `/var/log/apache2/`:
```
export APACHE_LOG_DIR=/var/log/apache2$SUFFIX
```

To specify the Locale, which is the language apache uses for output, `C` means use the default language:
```
export LANG=C
export LANG
```

This variable holds the command which is run when you run the `apachectl status` command:
```
#export APACHE_LYNX='www-browser -dump'
```

This variable holds the ammount of file descriptors (one for each file Apache deals with, so things like log files) which apache will use (default is 8192) 
```
#APACHE_ULIMIT_MAX_FILES='ulimit -n 65536'
```

This enviornment variable stores arguments which are passed to apache, when it's run:
```
#export APACHE_ARGUMENTS=''
```

If this is set to one, then apache will provide a verbose set of logs when web server modules or applications are installed:
```
#export APACHE2_MAINTSCRIPT_DEBUG=1
```

To reload the enviornment variables:
```
$	sudo systemctl reload apache2
```

`TraceEnable` is to allow the http `Trace` requests, which allow a client to see what is being received at the end of a request chain, which can be used for debugging. It should be disabled to prevent Cross-Site Tracing attacks.
```
TraceEnable Off
```

The `ServerSignature` option is to include information about the server (that it is Apache, and what version) when something like an error message, or you're browsing a directory
```
ServerSignature Off
```
This specifes how much information is sent back to clients regarding the server OS and Apache servers. Values range from `Prod` (least) to `Full` (most):
```
ServerTokens Prod
```

For the next two lines, you will need to enable the headers mod
```
$	sudo a2enmod headers
$	sudo systemctl restart apache2
```

This setting will prevent MSIE (Microsoft Internet Explorer) from interpreting files as something else from what it is declared as in HTTP headers:
```
Header set X-Content-Type-Options: "nosniff"
```

This option will prevent other sites from embedding pages from this site as frames, which defends against clickjacking attacks.
```
Header set X-Frame-Options: "sameorigin"
```



##### magic file

This should be a file found in the apache config directory titled `magic` (in Debian it is `/etc/apache2/magic`). This stores information about file headers, which apache uses to identify files as part of MIME.

## Security
Here is a breif look at what you can do to lock down apache

*	Server Side Includes
*	Timeout
*	Disable HTTP 1.0
*	ClickJacking
*	XSS
*	Disable HTTP Trace
*	Limit Request Types
*	Server Version
*	Allow Override
*	Options
*	SSL
*	IPTables Rules

##### Server Side Includes 
This attack essentially allows for the injecting of scripts into HTML pages. To defend against this (can increase stress on server) edit the apache config file:
```
#	vim /etc/apache2/apache2.conf
```

and add/edit in the `-Inlcudes` option inbetween the `<Directory>` tabs for the locations you want to defend:
```
<Directory /var/www/>
	Options -Indexes -Includes
	AllowOverride all
	Require all granted
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

##### Disable HTTP 1.0

HTTP 1,0 is an older version, which is vulnerable to session hijacking and should be disabled. To disable it, first enable the `rewrite` mod for apache:
```
#	a2enmod rewrite
#	systemctl restart apache2
```

Then edit your apache main conf:
```
#	vim /etc/apache2/apache2.conf
```

and add in the following lines to allow only in HTTP 1.1:
```
RewriteEngine On
RewriteCond %{THE_RQUEST} !HTTP/1.1$
RewriteRule .* - [F]
```

##### Clickjacking

Clickjacking is essentially when an attacker embeddes pages as frames. To prevent this, you can you first have to enable the `headers` (might be included in `security.conf` under `conf-available`):

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
Cross Site Scripting (XSS) is a type of attack where code is injected into a website, then executed. Apache can help mitigate this threat. First ensure the mod `headers` is enabled:
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

##### Disable HTTP Trace

Http trace can be used in Cross-Site scripting attacks, since the Trace request includes all http headers and cookie contents include authentication authentication data, so this should be disabled. To disable it edit your main apache conf file (this may be included in your `security.conf` file unser `conf-available`):

then edit your main apache conf file:
```
#   vim /etc/apache2/apache2.conf
```

add/edit this in:
```
TraceEnable Off
```

##### Limit Request Types

For most web apps, the only requests you need are `Get`, `Head`, and `Post`. However Http 1.1 supports additional requests such as `Options`, `Put`, `Delete`, `Trace`, and `Connect` which can be helpful to an attacker. We can whitelist the requests which we need. To do this edit the main apache conf file:
```
#   vim /etc/apache2/apache2.conf
```

add/edit in the following lines to in the directory sections  in the directort sections you wish to protect:

The options to whitelist the requests
```
	<LimitExcept GET POST HEAD>
	deny from all
	</LimitExcept>
```

Those configs in the directory sections:
```
<Directory />
	<LimitExcept GET POST HEAD>
	deny from all
	</LimitExcept>
	Options FollowSymLinks
	AllowOverride None
	Require all denied
</Directory>

<Directory /usr/share>
	<LimitExcept GET POST HEAD>
	deny from all
	</LimitExcept>
	AllowOverride None
	Require all granted
</Directory>

<Directory /var/www/>
	<LimitExcept GET POST HEAD>
	deny from all
	</LimitExcept>
	Options -Includes -Indexes
	AllowOverride all
	Require all granted
</Directory>
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

##### Allow Override
An attacker can override the apache configurations if they can upload a `.htaccess` file. To disable this edit the main apache conf file:
```
#   vim /etc/apache2/apache2.conf
```

add/edit in the following lines for each directory section (see above if you don't know what I mean by directory sections):
```
AllowOverride None
```

##### Options

Each directory section in the apache configs has Options such as `Indexes` which allow for directory listings. This would be very helpful for an attacker to see all files within a directory in your website.To disable this edit the main apache conf file:
```
#   vim /etc/apache2/apache2.conf
```

add/edit in the following lines for each directory section (see two above if you don't know what I mean by directory sections):
```
Options -Includes -Indexes
```

##### SSL
SSL is used by https to encrypt web traffic going to and from the web site, which provides a lot of security. To set this up, look at the `installation` section at the start of this writeup.

##### IPTables

To setup a firweall with Ingress and Egress for both HTTP (port 80) and HTTPS (port 443), add these firewall rules and block everything else:
```
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT
```

## Logs

##### System Logs

For logs related to the actual apache process, check the main log file for the os. For Debian, check `syslog`:
```
#	cat /var/log/syslog
```

Here is an example of a log from `syslog` regarding a failure to start because of bad config syntax:
```
Feb 28 00:30:09 Silent-Horror systemd[1]: Starting The Apache HTTP Server...
Feb 28 00:30:09 Silent-Horror apachectl[3312]: apache2: Syntax error on line 171 of /etc/apache2/apache2.conf: </LimitExcept>AllowOverride> directive missing closing '>'
Feb 28 00:30:09 Silent-Horror apachectl[3312]: Action 'start' failed.
Feb 28 00:30:09 Silent-Horror apachectl[3312]: The Apache error log may have more information.
Feb 28 00:30:09 Silent-Horror systemd[1]: apache2.service: Control process exited, code=exited status=1
Feb 28 00:30:09 Silent-Horror systemd[1]: Failed to start The Apache HTTP Server.
```

##### Access Logs

Access logs will tell you who submitted what requests where on the web server. This is typically controlled with the following config line in the Virtual Hosts' config file:
```
CustomLog ${APACHE_LOG_DIR}/access.log combined
```

By default the log location is `/var/www/apache2/access.log`. Here is an example log:
```
192.168.234.1 - - [28/Feb/2018:00:55:43 -0500] "GET / HTTP/1.1" 200 3459 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [28/Feb/2018:00:55:43 -0500] "GET /icons/openlogo-75.png HTTP/1.1" 200 6119 "http://192.168.234.154/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
192.168.234.1 - - [28/Feb/2018:00:55:43 -0500] "GET /favicon.ico HTTP/1.1" 404 409 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

##### Error Logs

Error Logs report errors exprerienced by apache. Like the access log it is typically controlled with the following config line in the Virtual Hosts' config file:

```
ErrorLog ${APACHE_LOG_DIR}/error.log
``` 

By default the log location is `/var/www/apache2/error.log`. Here is an example log:
```
[Wed Feb 28 00:07:05.679195 2018] [ssl:warn] [pid 2878:tid 140083323802816] AH01906: silent-horror.nz:443:0 server certificate is a CA certificate (BasicConstraints: CA == TRUE !?)
[Wed Feb 28 00:07:05.679225 2018] [ssl:warn] [pid 2878:tid 140083323802816] AH01909: silent-horror.nz:443:0 server certificate does NOT include an ID which matches the server name
```

## Migrating

To migrate a web site on apache, first just copy all of the contents out of the web directory (default is `/var/www/html`) to the new web directory, then just edit the config files to mirror the ones from the old instance (procided you don't want to change them).

## Extra

To get the version number for apache, along with information regarding apache:
```
/usr/sbin/apachectl -V
Server version: Apache/2.4.25 (Debian)
Server built:   2017-09-19T18:58:57
Server's Module Magic Number: 20120211:68
Server loaded:  APR 1.5.2, APR-UTIL 1.5.4
Compiled using: APR 1.5.2, APR-UTIL 1.5.4
Architecture:   64-bit
Server MPM:     event
  threaded:     yes (fixed thread count)
    forked:     yes (variable process count)
Server compiled with....
 -D APR_HAS_SENDFILE
 -D APR_HAS_MMAP
 -D APR_HAVE_IPV6 (IPv4-mapped addresses enabled)
 -D APR_USE_SYSVSEM_SERIALIZE
 -D APR_USE_PTHREAD_SERIALIZE
 -D SINGLE_LISTEN_UNSERIALIZED_ACCEPT
 -D APR_HAS_OTHER_CHILD
 -D AP_HAVE_RELIABLE_PIPED_LOGS
 -D DYNAMIC_MODULE_LIMIT=256
 -D HTTPD_ROOT="/etc/apache2"
 -D SUEXEC_BIN="/usr/lib/apache2/suexec"
 -D DEFAULT_PIDLOG="/var/run/apache2.pid"
 -D DEFAULT_SCOREBOARD="logs/apache_runtime_status"
 -D DEFAULT_ERRORLOG="logs/error_log"
 -D AP_TYPES_CONFIG_FILE="mime.types"
 -D SERVER_CONFIG_FILE="apache2.conf"
```
