## Solaris MySQL & Joomla

This will cover installing Joomla CMS on one box, and using a seperate box for the Database. Both boxes will be Solaris 11.

#### Apache:

First you will want to install the Apache webserver

```
#	pkg install web/server/apache-22
```

After that installs, you will want to enable the service

```
#	svcadm -v enable /network/http:apache22
svc:/network/http:apache22 enabled.
```

Next you will need to find your ip address using the following command

```
#	ifconfig -a
```

Next you should just be able to navigate to the ip address in a web browser (just type in the ip address into the url bar) and you should see a webpage that says it works.

#### PHP:

The easiest way to do this is to install the amp (apache mysql php) package. Or you might be able to get away with not doing it. I'll be frank, at this point I'm not sure.

```
#	pkg install amp
```

#### Mysql:

In order to setup MySQL, you will need to install the Server and the Client. If you want to install the server. At this point, you will need to know the ip address of the ECOM box (the server which hosts the website) since MySQL identifies based upon where the login is coming from. For here the ip address is "172.16.103.139" make sure to change it.

```
#	pkg install mysql-55
```

Next you will need to enable it

```
#	svcadm enable mysql
```

Next you will need to install the client

```
#	pkg install mysql-55/client
```

Now that you have that setup, you will need to log in and change the root password, since right now it's nothing. In here I will be changing my password to "memesfordayz".

```
#	mysql -u root
mysql>	use mysql;
Database changed
mysql> update user set password=password('memesfordayz') where user='root';
Query OK, 4 rows affected (0.00 sec)
Rows matched: 4 Changes: 4 Warnings: 0

mysql>	flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```

Now the root account on your mysql server should be password protected. Also now we will have to create a database and a user for 

```
#	mysql -u root -p
Enter password:
mysql>	create database joomla;
Query OK, 1 row affected (0.70 sec)

mysql>  create user 'joomla'@'172.16.103.139' identified by "memesfordayz";
Query OK, 0 rows affected (0.00 sec)

mysql> grant all on joomla.* to 'joomla'@'172.16.103.139';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```

#### Joomla:

First you will need to download Joomla, which can be done here:

```
https://downloads.joomla.org/
```

Next we will need to extract it

```
#	cd /tmp
#	cp /export/home/guyinatuxedo/Downloads/Joomla_3.7.2-Stable-Package.zip .
#	mkdir Joomla
#	unzip Joomla_3.7.2-Stable-Full_Package.zip -d Joomla/
```

and now throw it into the apache web directory

```
#	rm -rf /var/apache2/2.2/htdocs/index.html
#	cp -r /tmp/Joomla/* /var/apache2/2.2/htdocs/
```

After that, you should be able to install it through the website itself. It will ask you for settings such as what the admin account for the site should be, the MySQL DB (which for the host you will need to put the ip of the DB server) for the site, etc. After that you will need to remove the installation folder.

```
#	rm -rf /var/apache2/2.2/htdocs/installation/
```

It may also ask you to copy paste some settings into a config file "configuration.php". If that happens, create this file.

```
#	vim /var/apache2/2.2/htdocs/configuration.php
```

#### Writeups:

Install apache:
https://docs.oracle.com/cd/E36784_01/html/E49624/gnvhs.html

MySQL Removal:
https://docs.oracle.com/cd/E23824_01/html/E21802/gkoic.html
