# PowerDNS_RHEL
This is the setup for to RHEL servers, running a PowerDNS (pdns) master and BindDNS (also known as named) slave server. The setup for this is doen with only access to a basic RHEL mirror and flashdrives. Here are the DNS servers/clients:

*	DNS Server 1 (pdns):	`192.168.6.100`
*	DNS Server 2 (bind):	`192.168.6.101`
*	DNS Client 1:		`192.168.6.102`
*	DNS Client 2:	`192.168.6.103`


### pdns Installation

If you do have an internet connection, you can skip the next part by just installing this (have not tested):
```
$	sudo yum install epel-release
$	sudo yum install mysql mysql-server
$	sudo yum install pdns pdns-backend-mysql
```
##### Donloading pdns:

First add the user which powerdns will use (pdns). You want to make sure it doesn't have a home directory, and it can't login:

```
$	sudo useradd -M pdns
$	sudo usermod -L pdns
```

Next you will want to download the pdns package and install it:

```
$	cd /tmp
$	wget https://downloads.powerdns.com/releases/rmp/pdns-static-3.4.11-1.x86_64.rpm
$	rpm -i pdns-static-3.4.11-1.x86_64.rpm
```

Next you will want to add the user which pdns-backend-mysql will use: 

```
$	sudo useradd -M builder
$	sudo usermod -L builder
```

Then you want to download and install the pdns-backend-mysql package (got it from pkgs.org):
```
$	cd /tmp
$	wget ftp.altlinux.org/pub/distributions/ALTLinux/Sisyphus/x86_64/SRPMS.classic/pdns-4.0.3-alt2.src.rpm 
$	rpm -i pdns4.0.3-alt2.src.rpm
```

##### MySQL Database Config:

First start and enable the mysqld daemon to star on boot, then log on to mysql:

```
$	sudo /etc/init.d/mysqld start
$	sudo chkconfig mysqld
$	mysql -u root
```

First you will want to give root a password (in this instance it's m3Me_m@c!hn3):
```
mysql>	update mysql.user set password = password('m3Me_m@c!hn3') where user = 'root';
```

Next create the database and the table the pdns will use:

```
mysql>	create user 'pdns'@'localhost' identified by 'meme';
mysql>	create database pdns;
mysql>	grant select on pdns.* to 'pdns'@'localhost';
mysql>	flush privileges;
```

After that we will want to create the tables which MySQL will use:
```
mysql> CREATE TABLE domains (id INT auto_increment, name VARCHAR(255) NOT NULL, master VARCHAR(128) DEFAULT NULL, last_check INT DEFAULT NULL, type VARCHAR(6) NOT NULL, notified_serial INT DEFAULT NULL, account VARCHAR(40) DEFAULT NULL, primary key (id));
```
```
mysql> CREATE TABLE records (id INT auto_increment, domain_id INT DEFAULT NULL, name VARCHAR(255) DEFAULT NULL, type VARCHAR(6) DEFAULT NULL, content VARCHAR(255) DEFAULT NULL, ttl INT DEFAULT NULL, prio INT DEFAULT NULL, change_date INT DEFAULT NULL, auth TINYINT(1) DEFAULT 1, disabled TINYINT(1) DEFAULT 0, primary key(id));
```
```
mysql> CREATE TABLE supermasters (ip VARCHAR(25) NOT NULL, nameserver VARCHAR(255) NOT NULL, account VARCHAR(40) DEFAULT NULL);
```
```
mysql> create table domainmetadata (id INT AUTO_INCREMENT, domain_id INT NOT NULL, kind VARCHAR(32), content TEXT, PRIMARY KEY (id));
```

Next we will need to create some indexes:
```
mysql>	CREATE INDEX rec_name_index ON records(name);
mysql>	CREATE INDEX nametype_index ON records(name,type);
mysql>	CREATE INDEX domain_id ON records(domain_id);
```

##### pdns config:

Next edit the pdns config 

```
$	sudo vim /etc/powerdns/pdns.conf
```

Add this to the bottom. Change it to ensure that it reflects your correct information:
```
master = yes
slave = no
local-address = 0.0.0.0
local-port = 53
launch=gmysql
gmysql-host=127.0.0.1
gmysql-user=pdns
gmysql-dbname=pdns
gmysql-password=meme
allow-axfr-ips=192.168.6.101
```
##### Adding Records:

Now before we add records, we have to add a domain. The name of this domain will be 'hack.lan', and since it is the first domain made it's id will be 1 since it is an auto_increment value:


```
mysql> use pdns;
mysql> insert into domain (name, type) values ('hack.lan', 'MASTER');
```

Next we will add the dns records. These are the records which we will be adding:

*	An SOA records for hack.lan
*	A Nameserver (NS) record which points to 0ns.hack.lan
*	A Nameserver (NS) record which points to 1ns.hack.lan 
*	An A record which will make 0ns.hack.lan resolve to 192.168.6.100
*	An A record which will make 1ns.hack.lan resolve to 192.168.6.101

```
mysql> insert into records (domain_id, name, content, type, ttl, prio) values (1, 'hack.lan', 'localhost root.hack.lan 1', 'SOA', 86400, NULL);
mysql> insert into records (domain_id, name, content, type, ttl, prio) values (1, 'hack.lan', '0ns.hack.lan', 86400, NULL);
mysql> insert into records (domain_id, name, content, type, ttl, prio) values (1, 'hack.lan', '1ns.hack.lan', 86400, NULL);
mysql> insert into records (domain_id, name, content, type, ttl, prio) values (1, '0ns.hack.lan', '192.168.6.100', 'A', 120, NULL)
mysql> insert into records (domain_id, name, content, type, ttl, prio) values (1, '1ns.hack.lan', '192.168.6.101', 'A', 120, NULL)
```

In addition to that, we will need to add records to the pdns database to allow the Bind Slave to do a zone transfer, and to notfiy it:
```
mysql> insert into domainmetadata (domain_id, kind, content) values (1, 'ALSO-NOTIFY', '192.168.6.101');
mysql> insert into domainmetadate (domain_id, kind, content) values (1, 'ALLOW-AXFR-FROM', '192.168.6.101');
```

### Bind (named) Installation:

First install bind:
```
$	sudo yum install bind
```

Next you are going to want to edit the config file:
```
$	sudo vim /etc/named.conf
```

Add/Edit these settings in. Make sure they reflect your setup:

```
acl "clients" {
	192.168.6.100;
	192.168.6.101;
	192.168.6.102;
	192.168.6.103;
};

options {
	listen-on port 53 { 192.168.6.101; };
	allow-query { clients; };
	recursion yes
```

At the bottom of the file add this:
```
include "/etc/named/named.conf.local"
```

Make sure that these lines are commented out:
```
listen-on-v6	port 53 ( ::1; );
include "/etc/named.rfc1912.zones"
```

Next we will want to create the zone config file `/etc/named/named.conf.local`:
```
$	cat /etc/named/slaves/hack.lan
zone "hack.lan" {
	type slave;
	masters { 192.168.6.100; };
};
```

Now (actually way before this)  you should be at a point where you can do a zone transfer from pdns. To do it, try this:
```
$	dig axfr @192.168.6.100 hack.lan
```

If you see all of the dns records we made, it worked. If not then you should look in `/var/log/messages` to see what went wrong. If it did work, you should be able to start bind and everything should work:
```
$	sudo /etc/init.d/named start
$	sudo chkconfig named
```

### Security PowerDNS

Make sure in the `/etc/powerdns/pdns.conf` to specify who can do a Zone transfer:
```
allow-axfr-ips=192.168.6.101
```

To view logs:
```
#	/etc/init.d/pdns status
25892: Child running on pid 26910
#	cat /var/log/messages | grep 26910
```
##### IPTables Rules:

What's allowed (all IPv4, no IPv6)

*	new and established SSH inbound, established outbound SSH
*	new and estbalished outbound and inbound udp dns
*	DNS server talking to itself
*	DNS Zone transfers to `192.168.6.101`
*	all traffic comming in/out from mysql and to/from `127.0.0.1`
*	all traffic going to/from mysql from `127.0.0.1`
*	all ICMP traffic

For input:
```
#	iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A INPUT -p udp --dport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A INPUT -p udp --sport 53 -s 192.168.6.100 -d 192.168.6.100 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#	iptables -A INPUT -p tcp --dport 53 -s 192.168.6.101 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A INPUT -p tcp --sport 3306 -s 127.0.0.1 -d 127.0.0.1 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#	iptables -A INPUT -p tcp --dport 3306 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -P INPUT DROP 
```

For output:
```
#	iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#	iptables -A OUTPUT -p udp --sport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A OUTPUT -p udp --dport 53 -s 192.168.6.100 -d 192.168.6.100 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 53 -d 192.168.6.101 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A OUTPUT -p tcp --dport 3306 -s 127.0.0.1 -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A OUTPUT -p tcp --sport 3306 -s 127.0.0.1 -d 127.0.0.1 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#	iptables -P OUTPUT DROP
```

also if you want to query `192.168.6.101` for DNS, you will need to add these rules:
```
#	iptables -A INPUT -p udp --sport 53 -s 192.168.6.101 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#	iptables -A OUTPUT -p udp --dort 53 -d 192.168.6.101 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```

for forwarding:
```
#	iptables -P FORWARD DROP
```

save iptables rules:
```
$	sudo /sbin/iptables-save
```

for IPv6
```
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
ip6tables -P FORWARD DROP
/sbin/ip6
```

###Security Bind9

Make sure to specify that nobody can do a zone transfer, in `/etc/named.conf` with this line:
```
allow-transfer { none; };
```

make sure to specify who can query the DNS Server, which you can do with an access control list. List all of the ip addresses you need in the acl like this:
```
acl "clients" {
	192.168.6.100;
	192.168.6.101;
	192.168.6.102;
	192.168.6.103;
};
options	{
	allow-query { clients; };
```

To view logs:
```
#	/etc/init.d/named status
25892: Child running on pid 26910
#	cat /var/log/messages | grep 26910
```

#####IPTables:

For input:
```
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p udp --dport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 53 -s 192.168.6.100 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 53 -s 172.16.103.153  -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -P INPUT DROP
```

For output:
```
iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p udp --sport 53 -m conntrac k--ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 53 -d 192.168.6.100 -m conntrack -ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 53 -s 172.16.103.153  -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT
iptables -P OUTPUT DROP
```

For forward:
```
iptables -P FORWARD DROP
```

to save the iptables rules:
```
#	/sbin/iptables-save
```

for IPv6
```
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
ip6tables -P FORWARD DROP
/sbin/ip6
```

### Sources:

https://www.tecmint.com/install-powerdns-poweradmin-mariadb-in-centos-rhel/

