# MediaWiki

### Installation:

##### LAMP
First you will need to install LAMP (Linux, Apache, MySQL, PHP):
```
$	sudo apt-get install apache2 mysql-server php libapache2-mod-php php-mysql
```

Next we will need to install some php addons, the first being `php-mbstring`:
```
$	sudo apt-get install php-mbstring
```

The second one being `php-xml`:
```
$	sudo apt-get install php-xml
```
We should restart apache2 to have it recognize the new php plugins:
```
$	sudo systemctl restart apache2
```

##### MySQL

Next we will setup the database used by MediaWiki:
```
$	mysql -u root -p
```

First we will create a database for MediaWiki to use:
```
mysql> create database wikileaks;
```

Next create user for MediaWiki, and give it permissions to the wikileaks db:
```
mysql> create user 'snowden'@'localhost' identified by 'NSAsuckz';
mysql> grant all on wikileaks.* to 'snowden'@'localhost';
```

Now we can just Flush the permissions so our user changes will take effect and exit:
```
mysql> flush privileges;
mysql> exit
```

##### Apache

We will need t o download the latest MediaWiki version, which at the time of this writeup this is the latest:
```
$	cd /tmp
$	wget https://releases.wikimedia.org/mediawiki/1.28/mediawiki-1.28.2.tar.gz
$	tar -zvf mediawiki-1.28.2.tar.gz
$	sudo cp -avr mediawiki-1.28.2/* /var/www/html
$	sudo mv /var/www/html/index.html /var/www/html/htmlindex
```

After that, the rest of the installatin process takes place by naviagting to the website itself. At the end it will have you download a LocalSettings.php file which you need to place in the root directory of the wiki `/var/www/html` (if you have been following along).

### Security

To list all users go to this website (swap out the ip address with that of your server):
```
http://172.16.103.148/index.php?title=Special:ListUsers
```

##### IPTables

```
$	sudo iptables -P INPUT DROP
$	sudo iptables -P OUTPUT DROP
$	sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --sport 80 -m conntrack --ctstate ESYABLISHED -j ACCEPT
```

##### File Uploads

First we will want to disable file uploads, which we can do it with php. Edit the `php.ini` file:
```
$	sudo vim /etc/php/7.0/apache2/php.ini
```

If you can find it, run this php code:
```
<?php phpinfo(); ?>
```

Then you will need to make sure the `file_uploads` setting is configured like this:
```
file_uploads = Off
```

You will need to restart the apache daemon for it to take effect:
```
$	sudo systemctl restart apache2
```

##### Directory Browsing

To disables directory browsing, edit this file:
```
$	sudo vim /etc/apache2/apache2.conf
```

Change this:
```
<Directory /var/www/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
```

to this (remove `Indexes`):
```
<Directory /var/www/>
	Options FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
```

##### Config
The rest of these are lines of php code which you will need to add to the bottom of the `LocalSettings.php` file from earlier.

To prevent Anonymous users from editing:
```
$wgGrouPermissions['*']['edit'] = false;
```

To prevent users from being able to edit:
```
$wgGroupPermissions['user']['edit'] = false;
```

To prevent admins from being able to edit:
```
$wGroupPermissions['sysop']['edit'] = false;
```

To prevent new account creation, except for admins:
```
$wgGroupPermissions['*']['createaccount'] = false;
```

To prevent anonymous users from viewing pages, and whitelist the login page so they can use that:
```
$wgGroupPermissions['*']['read'] = false;
$wgWhitelistRead = array ("Special:Userlogin");
``` 

### Content

##### Uploading Files:

First make sure that php file uploading is enabled, by editiing `php.ini`:
```
$	sudo vim /etc/php/7.0/apache2/php.ini
```

Make sure that `file_uploads` is turned on:
```
file_uploads = ON
```

Next you will need to enable file upoloads in the `LocalSettings.php` file.

```
$	sudo vim /var/www/html/LocalSettings.php
```

Add this setting to the bottom:

```
$wgEnableUploads = true;
```

Also if the file your trying to upload doesn't have one of the default file extensions, let's say `.md` you will need to add one of these:

```
$wgFileExtensions[] = 'md';
```

Next you will want to change the owner of the web directory to the apache user, so MediaWiki can write to itself:

```
$	sudo chown www-data /var/www/html/*
```

After that you should be able to upload files to Mediawiki. On the sidebar on the left under tools, you should see an option to `upload file`. Click that and upload your file.

If you want to include a link file you uploaded `tripwire.md` in a page, add this text to it:
```
[[File:Tripwire.md]]
```

##### Creating Webpages:

To create a page, first as an admin naviagate to where the page should be. For instance if I wanted to create the page `Dragonforce` on the ip `172.16.103.148`:

```
http://172.16.103.148/index.php/Dragonforce
```

If you want to add a table, paste this text into the edit box:

```
{|
|hack0||0||harml3ss
|-
|hack1||4||ok
|-
|hack2||gg<br />at||surrender<br />20
|}
```

output:
```
hack0 	0 	harml3ss
hack1 	4 	ok
 	        gg surrender
hack2
		    at 20
```