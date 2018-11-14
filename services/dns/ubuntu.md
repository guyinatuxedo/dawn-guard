# Bind9DNS_Ubuntu_8.04

This DNS setup has a Primary DNS server that forwards to `8.8.8.8`, however will only allow the secondary DNS server at `172.16.103.154` to query it. All servers (including this one) will query the DNS forwarder for DNS. The IP address of this server is `172.16.103.153`.

### Installation:

##### Old Repositories:

First you are going to want to update the repository list:

```
$	sudo vim /etc/apt/sources.list
```

Now to have the repo list point to the old repositories, you will just need to replace all instances of `us.archive` and `security` with `old-releases`. This can be done with these vim commands:  

```
:%s/us.archive/old-releases
:%s/security/old-releases
```

##### APT

To install Bind9:

```
$	sudo apt-get update
$	sudo apt-get install bind9
```

##### Config:

First edit the bind9 options conf file:
```
$	sudo vim /etc/bind/named.conf.options
```

You will need to add the access control list of trusted dns clients (in here it's labeled `clients`) in addition to other settings such as what dns servers it should forward to if it can't resolve it:
```
acl "clients" {
	172.16.103.154;
};

options	{
	directory "/var/cache/bind";
	
		recursion yes;
			allow-recursion { clients;};
			listen-on { 172.16.103.153; };
			allow-transfer { 172.16.103.154;};
				
			forwarders {
				8.8.8.8;
			};	
```

Next you will need to edit the local conf file, and specify the zones you will be servimg. a forwards and reverse:

```
$	cat /etc/bind/named.conf.local
zone "dinosaur.lan" {
	type master;
	file "/etc/bind/zones/dinosaur.lan";
#	allow-transfer { 172.16.103.128; };
};

zone "103.16.172.in-addr.arpa" {
	type master;
	file "/etc/bind/zones/103.16.172";
};
```

Next we will need to first create the Forward Zone file, which stores the dns records:

```
$	sudo mkdir /etc/bind/zones
$	sudo vim /etc/bind/zones/dinosaur.lan
$	cat /etc/bind/zones/dinosaur.lan
$TTL	604800
@	IN	SOA	tri.dinosaur.lan. root.dinosaur.lan.	(
	2;			Serial
	604800;		Refresh
	86400;		Retry
	2419200;		Expire
	604800);		Negative Cache TTL
;
;	Name servers - NS Records
	IN	NS	tri.dinosaur.lan.

;	Name servers - A Records
tri.dinosaur.lan.		IN	A	172.16.103.153	

;	DNS Clients
dro.dinosaur.lan.		IN	A	172.16.103.150
tyr.dinosaur.lan.		IN	A	172.16.103.154
ter.dinosaur.lan.		IN	A	172.16.103.181
```

We will also need to configure the reverse zone:
```
$TTL	604800
@	IN	SOA	dinosaur.lan.	root.dinosaur.lan.	(
			3		;	Serial
			604800	;	Refresh
			86400	;	Retry
			2419200	;	Expire
			604800	;
)

;	name servers
	IN	NS	tri.dinosaur.lan.
	IN	NS	tyr.dinosaur.lan.

;	PTR Records
174	IN	PTR	tyr.dinosaur.lan.	;	172.16.103.154
153	IN	PTR	tri.dinosaur.lan.	;	172.16.103.153
150	IN	PTR	dro.dinosaur.lan.	;	172.16.103.150
```
 
##### Check/Test

To check the two conf files:
```
$	sudo named-checkconf
```

Check the Forward zone:
```
$	sudo named-checkzone dinosaur.lan /etc/bind/zones/dinosaur.lan
```

Check the reverse Zone
```
$	sudo named-checkzone 103.16.172.in-addr.arpa /etc/bind/zones/103.16.172
```

After that, you should be able to restart the bind daemon, set the dns server over to the dnsmasq at `172.16.103.154`, and everything should work.

```
$	/etc/init.d/bind9 restart 
```

At this point I'm going to assume that you have set your dns server. Now you should be able to use dns records:
```
$	ping tri.dinosaur.lan
```
### Security

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

### Security:
##### Zone Transfers, Recursions, and Notifications:

Zone transfer are when all DNS records in a server are outputted. You can restrict them with this:o
To only have `172.16.103.155` able to conduct a zone transfer.
```
allow-transfer { 172.16.103.155; };
```

To only allow `172.16.103.155` and `172.16.103.128` conduct a zone transfer:
```
allow-transfer { 172.16.103.155; 172.16.103.128; };
```

To allow nobody to do a zone transfer:
```
allow-transfer { none; };
```

Also there is a setting called `also-notify` whcih signifies which slave DNS servers can request an update of the DNS records which you should look at, syntax is the same for `allow-transfer`.

In addition to that, the `allow-recursion` is another setting which should be monitored, which allows the DNS server to query other DNS servers.

##### IPTables

Here are some functioning IPTables rules, it will allow in and out DNS, ICMP, unlimited traffic to and from 8.8.8.8, and SSH (for SSH outbound is established only):

for input:
```
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 953 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 953 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
iptables -A INPUT -p udp --dport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 53 -s 172.16.103.153 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -p INPUT DROP
```

for output:
```
iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p udp --sport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 953 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 953 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 53 -s 172.16.103.153  -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
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

To save the IPTables rules:
```
$	sudo /sbin/iptables-save
$	sudo /sbin.ip6tables-save
```

### Network

Static Ip Address:
```
$	cat /etc/network/interfaces
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
	address 172.16.103.153
	netmask 255.255.255.0
	gateway 172.16.103.2
	dns-nameservers 172.16.103.174
```

You might have to manually edit this file:
```
$	cat /etc/resolv.conf
nameserver 172.16.103.174
```

### Sources:
https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-16-04