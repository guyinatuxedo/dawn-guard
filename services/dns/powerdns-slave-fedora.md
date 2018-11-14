# PowerDNS Slave

This is a writeup for a PowerDNS Slave on Fedora 20 that has a bind9 ) named Master. Here is the setup:

*	Domain:	`dinosaur.lan`
*	Master DNS:	`tri.dinosaur.lan` @ 172.16.103.153
*	Slave DNS	:	`pte.dinosaur.lan` @ 172.16.103.154

### Master Setup:

In Bind9, in the `/etc/bind/named.conf.options` file, make sure to have these settings so the Slave DNS server can do the zone transfer to recieve all of the DNS records:
```
allow-transfer { 172.16.103.154; };
also-notify{ 172.16.103.154; };
```

Make sure you put that inbetween the options bracket. Next restart bind:
```
#	/etc/init.d/bind9	restart
```

Now from the Slave DNS server, you should be able to do a zone transfer. You can do a test zone transfer using dig:
```
#	dig axfr @172.16.103.153 dinosaur.lan

; <<>> DiG 9.9.4-RedHat-9.9.4-8.fc20 <<>> axfr @172.16.103.153 dinosaur.lan
; (1 server found)
;; global options: +cmd
dinosaur.lan.		604800	IN	SOA	tri.dinosaur.lan. root.dinosaur.lan. 3 604800 86400 2419200 604800
dinosaur.lan.		604800	IN	NS	tri.dinosaur.lan.
dro.dinosaur.lan.	604800	IN	A	172.16.103.150
pte.dinosaur.lan.	604800	IN	A	172.16.103.154
rap.dinosaur.lan.	604800	IN	A	172.16.103.155
rap.dinosaur.lan.	604800	IN	MX	10 rap.dinosaur.lan.
ter.dinosaur.lan.	604800	IN	A	172.16.103.181
tri.dinosaur.lan.	604800	IN	A	172.16.103.153
tyr.dinosaur.lan.	604800	IN	A	172.16.103.174
dinosaur.lan.		604800	IN	SOA	tri.dinosaur.lan. root.dinosaur.lan. 3 604800 86400 2419200 604800
;; Query time: 5 msec
;; SERVER: 172.16.103.153#53(172.16.103.153)
;; WHEN: Fri Jun 16 03:52:26 PDT 2017
;; XFR size: 10 records (messages 1, bytes 257)
```

If you see that, it means your Master DNS Server is setup to do zone transfers.

### Installation:

First install powerDNS, and the MySQL backend:
```
$	yum install pdns mysql mysql-server pdns-backen-mysql
```

##### PDNS Conf

Next edit the main pdns conf file:
```
#	vi /etc/pdns/pdns.conf
```

You are going to want to add these settings to the bottom:
```
allow-recursion=0.0.0.0/0
slave=yes
launch=gmysql
gmysql-host=localhost
gmysql-user=pdns
gmysql-password=meme
gmysql-dbname=pdns
```

after that, start and enable both pdns and mysql (mariadb):
```
#	systemctl start mariadb
#	systemctl start pdns
#	systemctl enable mariadb
#	systemctl enable pdns
```

##### MySQL

Undergo the MySQL secure installation to improve general security:
```
#	mysql_secure_installation
```

log in to root
```
$	mysql -u root -p
```

create the user, and the db which pdns will use:

```
MariaDB [(none)]> create database pdns;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user 'pdns'@'localhost' identified by 'pdns';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all on pdns.* to 'pdns'@'localhost';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

Next we are going to want to create the tables which pdns will need:

```
MariaDB [(none)]> CREATE TABLE domains (id INT auto_increment, name VARCHAR(255) NOT NULL, master VARCHAR(128) DEFAULT NULL, last_check INT DEFAULT NULL, type VARCHAR(6) NOT NULL, notified_serial INT DEFAULT NULL, account VARCHAR(40) DEFAULT NULL, primary key (id));
```
```
MariaDB [(none)]> CREATE TABLE records (id INT auto_increment, domain_id INT DEFAULT NULL, name VARCHAR(255) DEFAULT NULL, type VARCHAR(6) DEFAULT NULL, content VARCHAR(255) DEFAULT NULL, ttl INT DEFAULT NULL, prio INT DEFAULT NULL, change_date INT DEFAULT NULL, auth TINYINT(1) DEFAULT 1, disabled TINYINT(1) DEFAULT 0, primary key(id));
```
```
MariaDB [(none)]> CREATE TABLE supermasters (ip VARCHAR(25) NOT NULL, nameserver VARCHAR(255) NOT NULL, account VARCHAR(40) DEFAULT NULL);
```
```
MariaDB [(none)]> create table domainmetadata (id INT AUTO_INCREMENT, domain_id INT NOT NULL, kind VARCHAR(32), content TEXT, PRIMARY KEY (id));
```

##### DNS Records:
We will now be generating the DNS records needed in the mysql tables;
```
MariaDB [(none)]> insert into domainmetadata (domain_id, kind, content) values ()1, 'AXFR-SOURCE', '172.16.103.153');
MariaDB [(none)]> insert into domains (name, master, type) values ('dinosaur.lan', '172.16.103.153', 'slave');
MariaDB [(none)]> insert into supermasters ('172.16.103.153', 'pte.dinosaur.lan', 'admin');
```

The DNS records it gets from the zone transfer will automatically be stored in the records table.

##### Testing

Now you should be able to restart pdns, set your DNS server to the slave, and query the DNS server.
```
#	/etc/init.d/pdns restart
#	dig tri.dinosaur.lan
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60019
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 2800
;; QUESTION SECTION:
;tri.dinosaur.lan.		IN	A

;; ANSWER SECTION:
tri.dinosaur.lan.	604800	IN	A	172.16.103.153

;; Query time: 4 msec
;; SERVER: 172.16.103.154#53(172.16.103.154)
;; WHEN: Fri Jun 16 03:55:44 PDT 2017
;; MSG SIZE  rcvd: 61

```



##### Static IP Addressing

Add these lines:
```
IPADDR=172.16.103.155
GATEWAY=172.16.103.2
NETMASK=255.255.255.0
PEERDNS=no
```

Also make sure to add your nameservers:
```
#	cat /etc/resolv.conf
nameserver 172.16.103.154
```

Reboot to make it static.

### Security

Make sure in the `/etc/pdns/pdns.conf` to disable Zone transfer:
```
disable-axfr=yes
```
you should not have this line (ip doesn't matter):
```
allow-axfr-ips=192.168.6.101
```

To view logs:
```
#	systemctl status 922
#	cat /var/log/messages | grep 922
```

##### IPTables rules

For input:
```
#   iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#   iptables -A INPUT -p udp --dport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#   iptables -A INPUT -p udp --sport 53 -s 172.16.103.154 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#   iptables -A INPUT -p tcp --dport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#   iptables -A INPUT -p tcp --sport 3306 -s 127.0.0.1 -d 127.0.0.1 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#   iptables -A INPUT -p tcp --dport 3306 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A INPUT -p tcp --sport 53 -s 172.16.103.153 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#	iptables -A INPUT -p icmp -j ACCEPT
#   iptables -P INPUT DROP 
```

For output:
```
iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p udp --sport 53 -m conntrack --ctstate NEW,ESTABLISHED  -j ACCEPT
iptables -A OUTPUT -p udp --dport 53 -d 172.16.103.154 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
iptables -A OUTPUT -p tcp --dport 3306 -s 127.0.0.1 -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 3306 -d 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 53 -d 172.16.103.153 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT
iptables -P OUTPUT DROP
```

For Forward:
```
iptables -P FORWARD DROP
```

For IPv6:
```
ip6tables -F
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
ip6tables -P FORWARD DROP
```

to save your iptables rules:
```
/sbin/iptables-sabe
/sbin/ip6tables-sabe
```