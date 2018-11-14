# PrestaShop Setup

This will be covering PrestaShop 0.9.7 on Rhel 6.

### Installation:

##### Download:

Donload Prestashop from hwere:
```
https://www.prestashop.com/en/previous-versions
```

##### LAMP:

Install LAMP:
```
$	sudo yum install httpd php php-mysql mysql mysql-server php-gd
```
**
Start the daemons:
```
$	sudo service httpd start
$	sudo service mysqld start
```

Then enable the daemons:
```
$	sudo chkconfig mysqld on	
$	sudo chkconfig httpd on	
```

##### MySQL:

At the moment, MySQL does not have a password. We will want to fix that.
```
$	mysql -u root
```

you can do that by just updating the password field in the mysql.user table. Like this, if you want the password to be `m3m3z4d@yz`:
```
mysql> update mysql.user set password = password('m3m3z4d@yz');
```

After that we will need to create the database and user that Prestashop will be using:
```
mysql> create database meme_machine;
mysql> create user 'bmeme'@'localhost' identified by 'memes_f0r_d@yz';
mysql> grant all on meme_machine.* to 'bmeme'@'localhost';
mysql> flush privileges;
mysql> exit;
```

##### IPTables:

Clear the default iptables rules (if you have already configured iptables for prestashop do not do this):
```
$	sudo iptables -F
```



##### FINISH HIM!!!!

Unzip it:
```
$	cd /tmp
$	unzip -x prestashop_0.9.7.zip
$	sudo cp -avr prestashop/* /var/www/html
$	sudo mv /var/www/html/index.html /var/www/html/htmlindex
```

Make sure to change the owner of the web directory to apache so it will be able to edit it:
```
$	sudo chown -R apache:apache /var/www/html/
```

The next step of the installation should be able to be completed by going to the website with a broswer. After the installation you will need to remove the `instal` directory.

After that, you will need to rename the admin directory to something else in order to access the admin page (as an admin you can't log in as a customer):
```
$	cd /var/www/html
$	sudo mkdir super_secret
$	sudo cp -avr admin/* super_secret/
$	sudo chown -R apache:apache super_secret/
$	sudo rm -rf admin/
```

So to access the admin page if your ip address is `172.16.103.15` go to `http://172.16.103.15/super_secret`.

### Security:

##### Looking for PHP webshells:

To look for web shells, first check in the web directory if any of the files contain the following string:
```
#	grep -r "shell_exec" /var/www/html/*
#	grep -r "passthru" /var/www/html/*
#	grep -r "phpinfo" /var/www/html/*
#	grep -r "chown" /var/www/html/*
```

```
#	grep -r "chmod" /var/www/html/*
#	grep -r "readfile" /var/www/html/*
```

```
#	grep -r "system" /var/www/html/*
#	grep -r "base64_decode"
#	grep -r "fopen" /var/www/html/*
#	grep -r "fclose" /var/www/html/*
```
Next I rcogmend browsing through all of the directories, looking for in each directory files that contain "exec":
```
#	grep -r "exec" /var/www/html/img/*
#	grep "exec" /var/www/html/*
```

##### PHP Webshell logs:
Logs that would detect a PHP Shell in action would be in `/var/log/httpd/access_log`. Here is an example of an attacker from `10.10.10.1` using the `shell.php` to run the `ls` comand:
```
10.10.10.1 - - [07/Jan/2013:06:31:48 -0400] "GET /img/shell.php?sh=ls HTTP/1.1" 200 92 "-" "Mozilla/4.0 (Windows NT 8.0; WOW64; Trident/7.0; rv:11.0) like Gecko"
```

##### IPTables Rules:
```
$	sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
$	sudo iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
$	sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
$	sudo iptables -A OUTPUT -p tcp --sport 80 -m conntrack --ctstate ESTABLISHED -j ACCEPT
$	sudo iptables -P INPUT DROP
$	sudo iptables -P OUTPUT DROP
$	sudo iptables -P FORWARD DROP
$	service iptables save
```

##### Admin Password Reset:

If you want to reset the password for the admin account, just create a new user, then in mysql select the user's password from the `prestashop.customer` table, then update the `prestashop.employee` tables with the same hash (assuming the database's name is prestashop and there is no table prefix).

