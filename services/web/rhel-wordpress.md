### Wordpress on RHEL 6

### Installation:

##### Yum
First install lamp:
```
$	sudo yum install https mysql mysql-server php php-common php-mysql php-gd php-xml php-mbstring
```

Then start the services

```
$	sudo /etc/init.d/mysqld start
$	sudo /etc/init.d/httpd start
```
##### MySQL

Next initiate the secure installation of mysql:

```
$	mysql_secure_installation
```

With that, you can log in to MySQL:
```
$	mysql -u root -p
```

Then create a new user, and database for wordpress:

```
mysql>	create database irs;
mysql>	create user 'irs'@'localhost' identified by 'meme_machinez';
mysql>	grant all on irs.* to 'irs'@'localhost';
mysql> flush privileges;
mysql> exit
```

##### httpd

Next you are going to want to configure apache. Edit this file:
```
#	vim /etc/httpd/conf/httpd.conf
```

and add this to the bottom, change it to have it match your setup:
```
<VirtualHost *:80>
	ServerAdmin guy@guy.lan
	ServerName please
	DocumentRoot /var/www/html
	ErrorLog /var/log/httpd/wordpress-error-log
	CustomLog /var/log/httpd/wordpress-access-log common
</VirtualHost>
```

then restart Apache:
```
$	sudo /etc/init.d/httpd restart
```
##### Wordpress

You are going to want to download, extract and throw wordpress into the web directory:

```
$	cd /tmp
$	wget http://wordpress.org/latest.tar.gz
$	sudo tar -xvf wordpress-4.8.tar.gz
$	sudo cp -avr wordpress/* /var/ww/html
```



##### SELinux:

Now SELinux should be blocking your connection. To stop this for the install, you can disable it:

```
#	setenforce 0
```

However you can have both work together. First install some utillities to help manage SELinux, you probably will only need the first couple however I installed everything

```
#	yum install -y policycoreutils policycoreutils-python selinux-policy selinux-policy-targeted libselinux0utils setroubleshoot-server setools setools-console mcstrans
```

Next we need to allow apache to access the network and db.

```
#	setsebool -P httpd_can_network_connect 1
#	setsebool -P httpd_can_network_connect_db 1
```

Lastly we need to mark all of the files with the correct SELinux permissions:
```
#	semanage fcontext -a -t httpd_sys_context_t "/data/.sitehome(/.*)>"
#	restorecon -Rv /var/www/html
```

Then should be able to enable SELinux:
```
#	setenforce 1
#	getenforce
Enforcing
```

##### Web
From here on out, you should just be able to complete the installation by visiting the website in a web browser.

### Insert PHP

To download the addon, go here:
```
http://wordpress.org/plugins/insert_php/
```

There yoy can download a zip file, which you can install it from your client machine by navigating to `Plugins>Add New` and clikcing on `Upload Plugin`.
