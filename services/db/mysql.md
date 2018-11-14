# MySQL 1337 5h337
This is a reference on how to use mysql.

MySQL runs on port 3306

#### How to log in:

To log in to mysql as root using a password:

```
$	mysql -u root -p
```

To log in to mysql as root without a password:

```
$	mysql -u root
```

To log in to mysql on a remote host as the "joomla" user with a password (remote host ip is 172.16.103.141):

```
mysql -u joomla -p -h 172.16.103.141
```

#### MySQL General Commands:

To list all databases:

```
mysql> show databases;
```

To select a database to do stuff with named "guy" (in order to actually do stuff with the contents of a database you have to select it)

```
mysql> use tux;
```

To list all tables in a db

```
mysql> show tables;
```

to describe a table named tux (describe shows you what data it stores, and it;s names):

```
mysql> describe tux;
```

To select all data from a table named tux;

```
mysql> select * from tux;
```

To select all data from the user table in the mysql db without having it selected:

```
mysql> select * from mysql.user;
```

To select only the "exploit" collumn from the tux table;

```
mysql> select exploit from tux;
```

To change value of hax to 24 in the entry where id="1":

```
mysql> update tux set hac='24' where id='1';
```

To delete the entry where id = "1";

```
mysql> delete from tux where id = '1`
```

You can also include Ands in your commands, and this is to select all from the tux table where id = "2" and hax = "33";

```
mysql> select * from tux where id = '2' and hax = '33';
```

You can also include Ors in your commands, and this is to select all from the tux table where either id = "6" or hax = "33";

```
mysql> select * from tux where is = '6' or hax = '33';
```

You can also include Nots in your statement, which essentiallty reverse the search parameters. Like this to search for where the id = "2" and hax does not equal "34".

```
mysql> select * from tux where id = '2' and not hax = '34';
```

Or here just to select everything from the tux table that id does not equal 2.

```
mysql> select * from tux where not id = '2';
```

#### Managing Users:
Now for MySQL Users, they have a username and a password however they also have a host, which is the ip address at which the log in must be from. Make sure to flush privileges after changes have been made.

To view all users:
```
mysql>	select user, host from mysql.user;
```

To add a user named "guy", from the host "localhost" with the password "memez4dayz":

```
mysql>	create user 'guy'@'localhost' identified by 'memez4dayz';
```

To change the password for the user 'guy'@'localhost' to "memez4life":

```
mysql> update mysql.user set authentication_string = password('memez4life') where user = 'guy' and host = 'localhost';
```

If that doesn't work change `authentication_string` to `password` (probably an older version of MySQL):
```
mysql> update mysql.user set password = password('memez4life') where user = 'guy' and host = 'localhost';
```

To lock the account 'guy'@'localhost' (they won't be able to log in):

```
mysql> alter user 'guy'@'localhost' account lock;
```

To unlock the account 'guy'@'localhost' (they will be able to log in):

```
mysql> alter user 'guy'@'localhost' account unlock;
```

To delete the user 'guy'@'localhost'

```
mysql> remove user 'guy'@'localhost';
```

To flush the privileges so changes take place:

```
mysql> flush privileges
```

To create a user `meme` that can be logged in from anywhere use `%` for the host host):
```
mysql> create user 'meme'@'%' identified by 'memez';
```

#### Managing Permissions:
This section covers permissions, which is what user accounts can do what to what data. Make sure to flush privileges after changes have been made.

To display permssions for the account "guy"@"localhost":

```
mysql> show grants for 'guy'@'localhost';
```

To display permssions for the current user:

```
mysql> show grants;
```

To display permissions for all users:

```
mysql> select * from information_schema.schema_privileges;
```

To grant all access to the user 'guy'@'localhost' for all tables in the "db" database:

```
mysql> grant all on db.* to 'guy'@'localhost';
```

To grant all access to the user 'guy'@'localhost' for the "tux" table in the "db" database:

```
mysql> grant all on db.tux to 'guy'@'localhost';
```

To grant select and update permissions on the "db" database to the user 'guy'@'localhost':

```
mysql> grant select, update on db.* to 'guy'@'localhost';
```

To revoke all permissions to the "db" database to the user 'guy'@'localhost':

```
mysql> revoke all on db.* from 'guy'@'localhost';
```

To revoke select and update on db.* from 'guy'@'localhost';

```
mysql> revoke select, update on db.* from 'guy'@'localhost';
```

To flush the privileges so changes take place:

```
mysql> flush privileges
```

#### Break into MySQL:

In the event that you are on a server that has MySQL, however don't have the password form it, here is how you can break into it.

0.) To start of you will need to then stop the mysql daemon


for ubuntu:
```
#	/etc/init.d/mysql stop
```

other distros:
```
#	/etc/init.d/mysqld stop
```

1.) Then you will want to create the directory for MySQL to store it's socket file, and make sure it can write to it. If it already exits, skip this step.	

```
#	mkdir -p /var/run/mysqld
#	chown mysql:mysql /var/run/mysqld
```

2.) Next you will need to start up mysql in a mode where it doesn't require a password, and networking is disabled so someone can't just remotely access your root account:

```
#	mysqld_safe --skip-grant-tables --skip-networking &
```

3.) Now you should be able to log in to mysql using a different terminal or ttyl session:

```
#	mysql -u root
```

4.) Now that you are logged into mysql, you can update the password and flush the privileges. Here I changed it to "memez4life":

```
mysql> update mysql.user set authentication_string = password('memez4life') where user = 'root' and host = 'localhost';
mysql> flush privileges;
```

####MySQL Creating things:

To create a database named "guy":

```
mysql> create database guy;
```

Now to do anything with the contents of a database, whether it is selecting information, creating or dropping tables, or updating information, you need to select a database. To select a database named "guy":

```
mysql>	use guy;
```

Now to create a database named tux, that holds four values.

*	ID number that auto increments, will store up to 4 decimal places
*	An integer named "hax" that will store up to 5 decimal places
*	a string named "exploit" that will store up to 20 characters
*	the data, named as "day"

```
mysql>	create table tux (id int(4) unsigned auto_increment primary key, hax int(5), exploit varchar(20), day date);
```

Now to create a record for that database we just made, you can insert a row into it.  Keep in mind that the format for the date is (yyyy-mm-dd).

```
mysql>	insert into tux (hax, exploit, day) values (33, "buffer overflow", "1969-07-29");
```

#### MySQL Deleting things:

To delete the entry where id = "1";

```
mysql> delete from tux where id = '1`
```

To delete all data in the db.tux table:

```
mysql> truncate db.tux;
```

To delete a the db.tux table:

```
mysql> drop table db.tux;
```

To delete the db database:

```
mysql> drop database db;
```

### Altering Tables:

To add columns to a tables:
```
mysql>	alter table tux add string varchar(5);
```

to remove columns from a table:
```
mysql>	alter table tux drop columnd string;
```

to change the name and data type of the string column:
```
mysql>	alter table tux change column string new_name varchar(50);
```

#### Copying Database:

##### 0.) To copy a locally stored database to a remote server
The local database is called "local" and the remote database will be called "remote". The remote serber is 172.16.103.141, the remote user is called "guy" and for the local database I will be using root.


First make a database on the remote server to copy the data to:
```
$	mysqladmin -u guy -p -h 172.16.103.141 create remote
```

Next we can use mysqldump to dump the contents of the local database to a file:

```
$	mysqldump -u root -p local > /tmp/db
```

Now we can copy the contents of the file over to the remote server:

```
$      cat /tmp/db | mysql remote -h 172.16.103.141 -u guy -p 
```

##### 1.) To copy a remotely stored database locally

First create the database which will hold the data "local":

```
$	mysqladmin -u root -p create local
```

Copy the data over from the "remote" database on the remote server using the user account guy:

```
$	mysqldump -h 172.16.103.141 --compress remote -u guy -p > /tmp/db
```

throw the copied data into the "local" database:

```
$	cat /tmp/db | mysql local -u root -p 
```

### Security & Uptime

##### Backups
First take a backup of literally everything:
```
#	mysqldump -u root -p --all-databases > /backups/mysqlbackup
```

If you need to import the db backup:
```
#	mysql -u root -p < /backups/mysqlbackup
```

If you want to backup a single database:
```
#	mysqldump -u root -p Joomla > /backup/dbs/Joomla_1
```

and if you want to restore that single database:
```
mysqldump -u root -p Joomla > /backup/dbs/Joomla_1
```
##### Access Control

To view all users, and their hosts (btw % means all hosts):
```
mysql> select user, host from mysql.user;
```

To delete a a user:
```
mysql> drop user 'root'@'::1';
```

To view the permissions of the user `pdns@localhost`:
```
mysql> show grants for 'pdns'@'localhost';
```

To revoke update on all databases from the user `pdns@192.168.6.100`:
```
mysql> revoke update privileges on *.* from 'pdns'@'192.168.6.100';
```

To revoke select on the pdns database from the user 'guy@192.168.6.100'. Keep in mind that the grant must be there in order to revoke it.
```
mysql> revoke select on pdns.* from 'guy'@'192.168.6.100';
```

Now you should go through remove any unnecissary MySQL accounts (the scoring MySQL account is necissary). You should also change the passwords for all of the accounts (except for the scorred account), but before you change the password for an account that is used by another service like wordpress, change it with the guy who is working on it otherwise shit will break. The syntax for how to do this can be found under `Managing Users` earlier in the writeup.

To view what user you are logged in as:
```
mysql> select current_user();
```

To view what user you originally logged in as:
```
mysql> select user();  
```

To view all users that are logged in:
```
mysql> show processlist;
```

To kick a user out, change the password for the account then restart the service.

##### Logging

To enable logging, edit the mysql conf file:
```
#	vim /etc/mysql/my.cnf
```

add/edit the following line:
```
log = /var/log/mysql/mysql.log
```

then restart the mysql service:
```
#	/etc/init.d/mysql restart
```

then you should start seeing mysql logs appearing in there. This is what a successful log on/off looks like:
```
180221 12:25:02       8 Connect     root@localhost on 
                      8 Query       select @@version_comment limit 1
180221 12:26:42       8 Quit 
```

This is what a failed login looks like:
```
180221 12:26:55       9 Connect     Access denied for user 'root'@'localhost' (using password: YES)
```

This is what a select looks like:
```
180221 12:30:44       7 Query       select * from jos_users
```

This is what show databases looks like:
```
                      7 Query       show databases
```

This is what `show tables` looks like (it is followed by a list of all tables it displayed):
```
                      7 Query       show tables
                      7 Field List  jos_banner 
                      7 Field List  jos_bannerclient 
                      7 Field List  jos_bannertrack 
                      7 Field List  jos_categories 
                      7 Field List  jos_components 
                      7 Field List  jos_contact_details 
                      7 Field List  jos_content 
                      7 Field List  jos_content_frontpage 
                      7 Field List  jos_content_rating 
                      7 Field List  jos_core_acl_aro 
                      7 Field List  jos_core_acl_aro_groups 
                      7 Field List  jos_core_acl_aro_map 
                      7 Field List  jos_core_acl_aro_sections 
                      7 Field List  jos_core_acl_groups_aro_map 
                      7 Field List  jos_core_log_items 
                      7 Field List  jos_core_log_searches 
                      7 Field List  jos_groups 
                      7 Field List  jos_menu 
                      7 Field List  jos_menu_types 
                      7 Field List  jos_messages 
                      7 Field List  jos_messages_cfg 
                      7 Field List  jos_migration_backlinks 
                      7 Field List  jos_modules 
                      7 Field List  jos_modules_menu 
                      7 Field List  jos_newsfeeds 
                      7 Field List  jos_plugins 
                      7 Field List  jos_poll_data 
                      7 Field List  jos_poll_date 
                      7 Field List  jos_poll_menu 
                      7 Field List  jos_polls 
                      7 Field List  jos_sections 
                      7 Field List  jos_session 
                      7 Field List  jos_stats_agents 
                      7 Field List  jos_templates_menu 
                      7 Field List  jos_users 
                      7 Field List  jos_weblinks 
```
##### IPTables

For INPUT:
```
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 3306 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -P INPUT DROP
```

For OUTPUT:
```
iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 3306 -m conntrack --ctstate ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT
iptables -P OUTPUT DROP
```

For FORWARD:
```
iptables -A FORWARD -p icmp -j ACCEPT
```

to save iptables rules:
```
#	/sbin/iptables-save
```

for IPv6
```
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
ip6tables -P FORWARD DROP
/sbin/ip6tables-save
```

##### Logs and Daemon

The mysql service is know as mysql. To check the status of mysql

```
#	systemctl status mysql
```

To view logs:
```
#	cat /var/log/mariadb.log
```

or

```
#	cat /var/log/mysql.log
```


#### Extra:

To edit what port mysql listens on, edit this config file:
```
#	vim /etc/mysql/my.cnf
```

add/edit the following lines:
```
bind-address = 192.168.234.149
```

If you get this error when trying to remote into the mysql server:
```
ERROR 2003 (HY000): Can't connect to MySQL server on '<server-ip>' (110)
```

There is a good chance that mysql isn't listening on the right ip. To fix that edit this file.
```
#	vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

There you can find and change this setting, replace "127.0.0.1" with the ip of your server that other machines can reach:
```
bind-address		= 172.16.103.144
```

#### Writeups:
General MySQL:
https://www.w3schools.com/sql/

Copying MySQL Databases:
https://dev.mysql.com/doc/refman/5.7/en/copying-databases.html
