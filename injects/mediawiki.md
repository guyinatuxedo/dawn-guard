# MediaWiki

## Installation:

#### Lamp Installation

First you will need to install LAMP (Linux, Apache, MySQL, PHP):

```
$	sudo apt-get install apache2 mysql-server php libapache2-mod-php php-mysql
```

Next we will need to install some php addons:

```
$	sudo apt-get install php-mbstring php-xml
```

We should restart apache2 to have it recognize the new php plugins:

```
$	sudo systemctl restart apache2
```

#### MySQL config

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
mysql> create user 'snowden'@'localhost' identified by 'mem3z_4_d@yz';
Query OK, 0 rows affected (0.01 sec)

mysql> grant all on wikileaks.* to 'snowden'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> exit
Bye
```

#### Mediawiki Setup

We will need to download the latest MediaWiki version, which at the time of this writeup is `1.30`:

```
$	cd /tmp
$	wget https://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz
$	tar -zxvf mediawiki-1.30.0.tar.gz
$	sudo cp -avr mediawiki-1.30.0/* /var/www/html/
$	sudo chown -R www-data:www-data /var/www/html/
$	sudo mv /var/www/html/index.html /var/www/html/oldindex
```

After that, the rest of the installatin process takes place by naviagting to the website itself. At the end it will have you download a LocalSettings.php file which you need to place in the root directory of the webap:
```
$	sudo vim /var/www/html/LocalSettings.php
$	sudo chown www-data:www-data /var/www/html/LocalSettings.php 
```

## File Uploads Scripted

Full disclosure, I did not come up with this (it pretty much is just a straight copy and paste). 

First install Pandoc:
```
$	sudo apt-get install pandoc
```

next navigate to the directory which stores all of the documents:
```
$	cd /docs
```

next we are going to use `pandoc` to convert all of the microsoft documents to html:
```
$	find . -name "*.docx" -type f -exec sh -c 'pandoc "${0}" > "${0%.docx}.html"' {} \;
```

after that we are going to use `pandoc` to convert all of the html documents to a format to be imported by mediawiki: 
```
$	find . -name "*.html" -type f -exec sh -c 'pandoc -s -t mediawiki --toc "${0}" -o "${0%.html}.wiki"' {} \;
```

after that you can import all of the newly formatted documents using the user `guyinatuxedo` by using the `importTextFiles` php file located in the `maintenance` directory in the web root:
```
$	php /var/www/html/maintenance/importTextFiles.php /docs/*.wiki --user guyinatuxedo
```



## File Uploads Manual

First you will need to edit the `LocalSettings.php` file to configure file uploads:
```
$	sudo vim /var/www/html/LocalSettings.php
```

add/edit in these lines to enable uploading files for `docx`, `xlsx`, `pdf`, and `vsd` files (along with the default file types)

```
$wgEnableUploads = true;
$wgFileExtensions[] = 'docx';
$wgFileExtensions[] = 'xlsx';
$wgFileExtensions[] = 'pdf';
$wgFileExtensions[] = 'vsd';
```

also you might want to check your php config to ensure that it isn't blocking uploading php files:
```
$	sudo vim /etc/php/7.0/apache2/php.ini
```

these are the settings which will determine if php will allow you to upload files, how many at a time, and how large of a file it will upload:
```
file_uploads = On
upload_max_filesize = 2M
max_file_uploads = 20
```


## Security

To list all users go to this website (swap out the ip address with that of your server):

http://172.16.103.148/index.php?title=Special:ListUsers

##### IPTables

$	sudo iptables -P INPUT DROP
$	sudo iptables -P OUTPUT DROP
$	sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --sport 80 -m conntrack --ctstate ESYABLISHED -j ACCEPT

#### LAMP Security
Try looking at one of the other guides for securing a web app


#### Config 
The rest of these are lines of php code which you will need to add to the bottom of the LocalSettings.php file from earlier.

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
## Creating Webpages:

To create a page, first as an admin naviagate to where the page should be. For instance if I wanted to create the page Dragonforce on the ip 172.16.103.148:

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
