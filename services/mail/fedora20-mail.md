# PowerDNS Secondary
This guide is for the setup of a Secondary PowerDNS Server, which uses a master Bind DNS server from the other DNS writeup.

*	Domain Name:	`dinosaur.lan`
*	Primary DNS Server:	`172.16.103.153`
*	Secondary DNS Server: `172.16.103.154`


### Installation:

#### Primary DNS

You will want to edit this file:
```
#	vim /etc/bind/name.conf.options
```

Make sure to add the ip to the access control list `172.16.103.174`:
```
acl "clients" {
	172.16.103.174;
	172.16.103.154;	
};
```

Also add/edit this so the DNS server can do zone transfers:
```
allow-transfer { 172.16.103.154; } ;
```

then restart bind:
```
#	/etc/init.d/bind9 restart
```

then from the Secondary DNS server, you shoul dbe able to do a zone transfer:
```
#	dig axfr @172.16.103.153 dinosaur.lan
```

If you see all of the dns records, you know it works and you can move onto the secondary server.

#### Secondary DNS

First install PowerDNS, and the MySQL backend:
```
$	sudo yum install pdns pdns-backend-mysql
```

Next edit this file:
```
$	sudo vim /etc/pdns/pdns.conf
```

add these settings to the bottom:
```
allow-recursion=0.0.0.0/0
local-port=53
slave=yes
launch=gmysql
gmysql-host=localhost
gmysql-user=pdns
gmysql-password=meme
gmysql-dbname=pdns
```

Next you will want to undergo the mysql secure installatio

```
#	mysql_secure_installation
```

after that, login to mysql and create a database and user for it:

```
$	mysql -u root -p
mysql> create user 'pdns'@'localhost' identified by 'meme';
mysql> create database pdns;
mysql> grant all on pdns.* to 'pdns'@'localhost';
mysql> flush privileges;
```
After that we will want to create the tables which MySQL will use:
```
mysql> CREATE TABLE domains (id INT auto_increment, name VARCHAR(255) NOT NULL, master VARCHAR(128) DEFAULT NULL, last_check INT DEFAULT NULL, type VARCHAR(6) NOT NULL, notified_serial INT DEFAULT NULL, account VARCHAR(40) DEFAULT NULL, primary key (id));
```
```
mysql> CREATE TABLE records (id INT auto_increment, domain_id INT DEFAULT NULL, name VARCHAR(255) DEFAULT NULL, type VARCHAR(6) DEFAULT NULL, content VARCHAR(255) DEFAULT NULL, ttl INT DEFAULT NULL, prio INT DEFAULT NULL, change_date INT DEFAULT NULL, auth TINYINT(1) DEFAULT 1, disabled TINYINT(1) DEFAULT 0, primary key(id));
```
```
CREATE TABLE supermasters (
ip VARCHAR(25) NOT NULL,
nameserver VARCHAR(255) NOT NULL,
account VARCHAR(40) DEFAULT NULL
);
```
```
mysql> create table domainmetadata (id INT AUTO_INCREMENT, domain_id INT NOT NULL, kind VARCHAR(32), content TEXT, PRIMARY KEY (id));
```

Next we will need to create the records needed to signify the master DNS, and do a zone transfer from it:
```
mysql>	insert into domainmetadata (domain_id, kind, content) values (1, 'AXFR-SOURCE', '172.16.103.153';
mysql>	insert into domains (name, master, type) values ('dinosaur.lan', '172.16.103.153', 'SLAVE');
mysql>	insert into supermasters values ('172.16.103.153', 'pte.dinosaur.lan', 'admin');
```

After that you can restart mysqld, and powerDNS, enable both of them, and everything should work (you have to configure `/etc/resolv.conf` to use the dns serrver):
```
#	systemctl enable pdns
#	systemctl enable mariadb
#	systemctl restart pdns
#	systemctl restart mariadb
```

### Extra

##### Networking:
```
$	yum remove firewalld
```

```
$	vim /etc/sysconfig/network-scripts/ifcfg-eno16777736
```
```
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=none
IPADDR=172.16.103.154
GATEWAY=172.16.103.2
```

```
$	sudo vim /etc/yum.repos.d/fedora.repo
```

##### Sources:

Only do this if it your repos don't work, which can be found in `/etc/yum.repos.d/`

```
[fedora-debufinfo]
```
```
baseurl=http://download.fedoraproject.org/pub/fedora/linux/releases/$releaserver/Everything/$basearch/debug
```

```
[fedora-source]
```

```
baseurl=http://download.fedoraproject.org/pub/fedora/linux/releases/$releaserver/Everything/source/SRPMS/
```
### Writeups

This writeup is based off of these writeups

https://doc.powerdns.com/md/authoritative/backend-generic-sql/
http://www.debiantutorials.com/installing-powerdns-as-supermaster-with-slaves/
https://doc.powerdns.com/md/authoritative/domainmetadata/
