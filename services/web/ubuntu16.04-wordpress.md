## Ubuntu 16.01 Wordpress Setup

#### LAMP Installation
Lamp stands for Linux Apache Mysql Php, which we will install the last three on this linux distro.

To start off, install the mysql server, php mysql plug in, and php apache plug in. Durring this it will ask you for 

```
#	sudo apt-get update
#	sudo apt-get install mysql-server libapache2-mod-php php-mysql
```

Next install php and apache

```
#	sudo apt-get update
#	sudo apt-get install apache2 php
```

Next we will want to restart apache to ensure it is working fine with the other componets, and enable it so it will started upon boot

```
#	systemctl restart apache2
#	systemctl enable apache2
```

Just to verify that everything is working, we can wget our apache server to ensure that it is functiong (replace the 127.0.0.1 with your lamp server's ip)

```
#	wget 127.0.0.1
--2017-05-25 18:08:14--  http://127.0.0.1/
Connecting to 127.0.0.1:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11321 (11K) [text/html]
Saving to: ‘index.html’

index.html          100%[===================>]  11.06K  --.-KB/s    in 0s      

2017-05-25 18:08:14 (443 MB/s) - ‘index.html’ saved [11321/11321]
#	cat index.html
```

As you can see, we were able to pull an html page from our webserver, and it has html code that we expect. Now to test mysql, we can just log in to it using our password.

```
#	mysql -u root -p
Enter password:
```

When we log in, we see that it's running. So now to verify that php is working, by creating a php file in apache's web directory and see if apache will execute the php code.

```
#	mkdir /var/www/html/test
#	vim /var/www/html/test/test.php
```

Now in it, we will have to input the following php code

```
<?php
phpinfo();
?>
```

Now with that file, we can just navigate to the file through apache. You can do it by navigating to the following address (swap out the ip for that of your own lamp server). If you see a bunch of information about your php installation, it works.

```
http://172.16.103.135/test/test.php
```

#### Wordpress MySQL Setup

Now for Wordpress, we will need to configure a database and user for wordpress. First we will need to long on to the server

```
#	mysql -u root -p
```

First we will want to create the database for wordpress to use (in this case it's called wordpress)

```
mysql> create database wordpress;
Query OK, 1 row affected (0.01 sec)
```

Next we will want to create a user for wordpress, and give it permissions to it's database. Here the username will be "wordpress", and the password will be "wordpass". MySQL requires it's users to also be from a specific host, so in this case I will have it come from localhost. After that, we will just need to flush privileges so what we did will take effect, and exit MySQL

```
mysql>	create user 'wordpress'@'localhost' identified by 'wordpass';
Query OK, 0 rows affected (0.01 sec)

mysql> grant all privileges on wordpress.* to 'wordpress'@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```

#### Wordpress Setup

First we will need to download the latest wordpress version, which we can do using wget.

```
#	cd /tmp
#	wget http://wordpress.org/latest.tar.gz
```

after that is done installing, we will want to extract the tar file and move the contents over to the apache2 directory "/var/www/html". We might also want to clear the apache2 directory before we move anything in there.

```
#	tar xvfz latest.tar.gz
#	rm -rf /var/www/html/*
#	cp -avr wordpress/* /var/www/html/
```

Next we will want to create the wordpress config file. Luckily it comes with a sample one which we can just copy and modify.

```
#	cd /var/www/html
#	cp wp-config-sample.php wp-config.php
#	vim wp-config.php
```

Now in it, you will want to change these lines so Wordpress can authenticate and use the MySQL database.

```
/** The name of the database for WordPress */
define('DB_NAME', 'database_name_here');

/** MySQL database username */
define('DB_USER', 'username_here');

/** MySQL database password */
define('DB_PASSWORD', 'password_here');

/** MySQL hostname */
define('DB_HOST', 'localhost');
```

Now you will want to input the correct information so Wordpress will be able to use the user account and database we made for it. Since the user account I made for it was designated for the localhost, I won't have to change that.

```
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpress');

/** MySQL database password */
define('DB_PASSWORD', 'wordpass');

/** MySQL hostname */
define('DB_HOST', 'localhost');
```

As you can see, the information there reflects the settings we made from the MySQL section.

Next we will need to make the apache web directory owned by the apache service "www-data".

```
#	chown -R www-data /var/www/html/*
```

 Now after that, you should be able to navigate to the Wordpress site in a web browser and finish the installation there. For me I would just go here (substitute in your ip)

```
http://172.16.103.135
```

From there you can finish the installation

#### Setup Insert PHP Plugin

Insert PHP is a plugin that will allow you to insert php code into your posts, and it will actually run. To install it, first you will need to download and unzip it.


```
#	cd /tmp
#	wget http://www.willmaster.com/download/DownloadHandlerLink.php?file=insert-php.zip
#	unzip -D DownloadHandlerLink.php?file=insert-php.zip
```

You should see two new files, a "readme.txt" that describes the plugin and "insert_php.php" which actually contains the plugin. To install the plugin just move the "insert_php.php" into the plugin directory.

```
#	cp insert_php.php /var/www/html/wp-content/plugins/
```

After that the plug in should be installed. You just need to activate it in the Wordpress admin pannel under Plugins>Installed Plugins. After that you will be able to inset php code into your pages, however you will need to change the php tags from "<?php" and "?>" to "[insert_php]" and "[/insert_php]". For instance if you wanted to run this php code

```
<?php
phpinfo();
?>
```

You would put this

```
[insert_php]
phpinfo();
[/insert_php]
```

Once you save it, and view the page in browser you should see the php code run. That's it!

Here is some sample php code

```

/*phpinfo();*/


$user = "cash";
$pass = "1R5sUp3r%3cur1TY";
$db = "cash_n_grab";
$sever = "localhost";

$mysql_connection = new mysqli($server, $user, $pass, $db);

if ($mysql_connection->connect_error) {

die("Connection failed, contact sysamind:  " . $mysql_connection->connect_error);

}

$input = $_POST["Birth"];
$input = htmlspecialchars($input);
$input = $mysql_connection->real_escape_string($input);


$select = "Select * from personal_information where Birthdate = '" . $input . "'";
$query = $mysql_connection->query($select);


if  ($query->num_rows > 0) {

while ($row = $query->fetch_assoc()) {

echo "id: " . $row["id"] . " SSN: " . $row["SSN" ] . " Birthdate: " . $row["Birthdate"] . " Tax Refund Amount: " . $row["tax_refund"] . " Address: " . $row["Address"] . " Credit Card: " . $row["CreditCard"] . " Bank Account Number: " . $row["bank_account"] . " Routing Number: " . $row["routing"] . "<br>";


}
}
else {

echo "We do not have an account associated with that Birth Date.";

}

[/insert_php]
```