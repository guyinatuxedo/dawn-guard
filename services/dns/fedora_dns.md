# BindDNS_Fedora-25
For this DNS setup, I will be using the following information. Make sure to substitute it in for your own information.

```
Primary DNS Server: 0ns	172.16.103.157
Secondary DNS Server: 1ns	172.16.103.131
DNS Client: client			172.16.103.125
```


## Primary DNS Server Setup
First you will need to install BIND

```
#	sudo yum install bind bind-utils
```

Next you need to edit the config file for Bind, which is also called named.

```
#	sudo vi /etc/named.conf
```

First you will need to add the trusted ip addresses, which include all of the DNS servers, and clients. To do that add the following text, however subsitute in your own ip addresses and names.

```
acl "dns_ips"	{
	172.16.103.157;	#Primary DNS Server
	172.16.103.131;	#Secondary DNS Server
	172.16.103.125;	#DNS Client
	127.0.0.1;		#loopback
};
```

Next we will need to add the ip that the DNS sevrer will be listening on (by default it's port 53). In addition to that, if you do not plan on using IPv6 you should comment out the setting to listen on it. So change this...

```
options	{
		 listen-on port 53 { 127.0.0.1; };
                 listen-on-v6 port 53 { ::1; };
```

To this (make sure to substitue the ip address with that of your primary DNS server).

```
options	{
		 listen-on port 53 { 127.0.0.1; 172.16.103.157; };
#           listen-on-v6 port 53 { ::1; };
```

Next you will need to specify what ip addresses are allowed to do a zone transfer (for the secondary DNS server), and what ip addresses are allowed to query DNS. So edit the following lines

```
allow-query { dns_ips; };				#Allow DNS Queries
```

Lastly with this file, we will need to include a seperate config file which we will store additional configurational information, so just add this line to the bottom.

```
include "/etc/named/dns.conf"
```

Now we will need to create the config file we just included

```
#	vi /etc/named/dns.conf
```

Now here we will specify the two zones we will be using, and that they will both be masters. One will be a Forward zone and used to get an ip address from a DNS name, and the other will be a Reverse zone which is used to get a DNS name from and ip address. For the reverse zone file, we will name it the network portion of the ip address (this subnet mask is a /24 so the first three octets) backwards followed by "103.16.172.in-addr.arpa". So just add this text (change the zone and file names to match your config).

```
zone "nightfall.bz" {
		type master;
		file "/etc/named/dns_zones/db.nightfall.bz";
};

zone "103.16.172.in-addr.arpa" {
		type master;
		file "/etc/named/dns_zones/db.reverse";
};
```

Now we will need to create the two zone files our zones point to. Let's start with the forwarding zone. But first we will need to create the directory which they are stored.

```
#	mkdir /etc/named/dns_zones/
#	vi /etc/named/dns_zones/db.nightfall.bz
```

Now we will need to add DNS records for the forward zone. The first record we will need to add is a SOA (start of authority), which specifies information such as refresh timers, the domains serial numeber (which is used as a version number), and some other information such as the administrators email address (in this case it's root.nightfall.bz). Btw ";" signifies a comment

```
; SOA record
@	IN	SOA		0ns.nightfall.bz.	root.nightfall.bz.	(
		3		;Serial Number
		604800	;Amount of seconds before slave DNS will refresh
		86400	;# of second before slave DNS will try to contact master after it failed to connect
		2419200	;# of seconds the slave DNS will keep a cached zone file after it fails to reach the primary DNS server
		604800	;# of seconds by default the slave DNS will keep a Zone File cache
)
```

Now onto te NS records, which are DNS records that signify the DNS servers. 

```
; DNS Servers
nightfall.bz.	IN	NS	0ns.nightfall.bz.
nightfall.bz.	IN	NS	1ns.nightfall.bz.
```

Now that we have those, we can move onto the A records which just map a domain name to an ip address.

```
; Our DNS Server A records
0ns.nightfall.bz.		IN	A	172.16.103.157
1ns.nightfall.bz.		IN	A	172.16.103.149

; Our DNS Client
client.nightfall.bz.		IN	A	172.16.103.125
```

Lastly we need to make the CNAME record, which will associate the domain name with an ip address (that of 0ns.nightfall.bz.)

```
; CNAME record
www	IN	CNAME	0ns.nightfall.bz.
```

In the end, our Forward zone file should look like this.

```
; SOA record
@	IN	SOA		0ns.nightfall.bz.	root.nightfall.bz.	(
		3		;Serial Number
		604800	;Amount of seconds before slave DNS will refresh
		86400	;# of second before slave DNS will try to contact master after it failed to connect
		2419200	;# of seconds the slave DNS will keep a cached zone file after it fails to reach the primary DNS server
		604800	;# of seconds by default the slave DNS will keep a Zone File cache
)

; DNS Servers
	IN	NS	0ns.nightfall.bz.
	IN	NS	1ns.nightfall.bz.
	

; Our DNS Server A records
0ns.nightfall.bz.		IN	A	172.16.103.157
1ns.nightfall.bz.		IN	A	172.16.103.149

; Our DNS Client
client.nightfall.bz.		IN	A	172.16.103.125

; CNAME record
www	IN	CNAME	0ns.nightfall.bz.
```

Now onto the Reverse Zone File

```
#	vi /etc/named/dns_zones/db.reverse
```

Firstly, just like with the Forward Zone File we need to specify the SOA record.

```
; SOA record
@	IN	SOA		0ns.nightfall.bz.	root.nightfall.bz.	(
		3		;Serial Number
		604800	;Amount of seconds before slave DNS will refresh
		86400	;# of second before slave DNS will try to contact master after it failed to connect
		2419200	;# of seconds the slave DNS will keep a cached zone file after it fails to reach the primary DNS server
		604800	;# of seconds by default the slave DNS will keep a Zone File cache
)
```

Next just like the Forward Zone File we need to specify the NS records. Btw if you don't have the indentation in the front, named won't recognize it (at least it didn't for me).

```
; DNS Servers
	IN	NS	0ns.nightfall.bz.
	IN	NS	1ns.nightfall.bz.
```

Then we need to specify the PTR records, which maps an ip address to a DNS name (opposite of an A record). Now for the ip address, we already specified the network portion (172.16.103) with the zone name in "/etc/named/dns.conf" so we only need to specify the client portion

```
; PTR Records
157	IN	PTR	0ns.nightfall.bz.
149	IN	PTR	1ns.nightfall.bz.
125	IN	PTR	client.nightfall.bz.
```

Lastly we need to make the CNAME record, which will associate the domain name with an ip address (that of 0ns.nightfall.bz.)

```
; CNAME record
www	IN	CNAME	0ns.nightfall.bz.
```

So in the end, our Reverse Zone File should look like this

```
; Reverse Zone File

; SOA record
@	IN	SOA		0ns.nightfall.bz.	root.nightfall.bz.	(
		3		;Serial Number
		604800	;Amount of seconds before slave DNS will refresh
		86400	;# of second before slave DNS will try to contact master after it failed to connect
		2419200	;# of seconds the slave DNS will keep a cached zone file after it fails to reach the primary DNS server
		604800	;# of seconds by default the slave DNS will keep a Zone File cache
)

; DNS Servers
	IN	NS	0ns.nightfall.bz.
	IN	NS	1ns.nightfall.bz.
	
; PTR Records
157	IN	PTR	0ns.nightfall.bz.
149	IN	PTR	1ns.nightfall.bz.
125	IN	PTR	client.nightfall.bz.

; CNAME record
www	IN	CNAME	0ns.nightfall.bz.
```

## Primary DNS Sever Check

So now that we are done with that, we can check the config files we made using named.

```
#	named-checkconf
```

If there is no output, then it means everything is ok (otherwise just fix what it complains about). Next we can actually check the zone files we created.

```
#	named-checkzone nightfall.bz /etc/named/dns_zones/db.nightfall.bz
/etc/named/dns_zones/db.nightfall.bz:1: no TTL specified; using SOA MINTTL instead
zone nightfall.bz/IN loaded serial 3
OK
#	named-checkzone 103.16.172.in-adr.arpa /etc/named/dns_zones/db.reverse
/etc/named/dns_zones/db.reverse:1: no TTL specified; using SOA MINTTL instead
zone 103.16.172.in-adr.arpa/IN loaded serial 3
OK
```

As you can see, named has no problems with our zone files so far. Now let's start and enable (enabling makes it so named is started upon bootup) the named daemon.

```
#	systemctl enable named
#	systemctl start named
``` 

## DNS Server Functionallity Check

So now that we have everything setup, we should be able to test our primary DNS server using dig (keep in mind you have o add the DNS server as one of your own to test it, or specify the IP address). Let's see if we can resolve a DNS name into an ip address, and vice versa.

```
$	dig 0ns.nightfall.bz

; <<>> DiG 9.10.4-P8-RedHat-9.10.4-5.P8.fc25 <<>> 0ns.nightfall.bz
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37018
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;0ns.nightfall.bz.		IN	A

;; ANSWER SECTION:
0ns.nightfall.bz.	604800	IN	A	172.16.103.157

;; AUTHORITY SECTION:
nightfall.bz.		604800	IN	NS	0ns.nightfall.bz.
nightfall.bz.		604800	IN	NS	1ns.nightfall.bz.

;; ADDITIONAL SECTION:
1ns.nightfall.bz.	604800	IN	A	172.16.103.149

;; Query time: 0 msec
;; SERVER: 172.16.103.157#53(172.16.103.157)
;; WHEN: Mon May 22 20:47:36 EDT 2017
;; MSG SIZE  rcvd: 109

$	dig -x 172.16.103.125

; <<>> DiG 9.10.4-P8-RedHat-9.10.4-5.P8.fc25 <<>> -x 172.16.103.125
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9551
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;125.103.16.172.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
125.103.16.172.in-addr.arpa. 604800 IN	PTR	client.nightfall.bz.

;; AUTHORITY SECTION:
103.16.172.in-addr.arpa. 604800	IN	NS	0ns.nightfall.bz.
103.16.172.in-addr.arpa. 604800	IN	NS	1ns.nightfall.bz.

;; ADDITIONAL SECTION:
0ns.nightfall.bz.	604800	IN	A	172.16.103.157
1ns.nightfall.bz.	604800	IN	A	172.16.103.149

;; Query time: 0 msec
;; SERVER: 172.16.103.157#53(172.16.103.157)
;; WHEN: Mon May 22 20:51:11 EDT 2017
;; MSG SIZE  rcvd: 157
```

As you can see, we are able to succesfully pull the DNS records, so we know that the primary DNS Server is Working.


## Slave DNS Server Setup

First install bind on the secondary sever.

```
#	sudo yum install bind bind-utils
```

Next you need to edit the named config file

```
#	vi /etc/named.conf
```

Next you need to add the trusted ip addresses, just like the Master Server

```
acl "dns_ips"   {
    172.16.103.157; #Primary DNS Server
    172.16.103.131; #Secondary DNS Server
    172.16.103.125; #DNS Client
    127.0.0.1;		#loopback
};
```

Then you need to add the ip address which DNS will be listening on (and comment out the IPv6 listening)

```
options {
         listen-on port 53 { 127.0.0.1; 172.16.103.131; };
#     listen-on-v6 port 53 { ::1; };
```

Next we have to allow queries from the trusted ip addresses.

```
allow-query { dns_ips };
```

Lastly with this file, we have to include an additional config file at the end (in addition to the comment out earlier)

```
include "/etc/named/dns.conf"
```

Now we need to create the config file we just included, and make sure it has the right permissions

```
#	chmod 755 /etc/named
#	vi /etc/named/dns.conf
```

Then we need to add this configuration (make sure to adjust it to match your own)

```
zone "nightfall.bz" {
	type slave;
	file "slaves/db.nightfall.bz";
	masters { 172.16.103.157; };
};

zone "103.16.172.in-addr.arpa" {
	type slave;
	file "slaves/db.reverse";
	masters { 172.16.103.157; };
};
```

With this configuration we should be done. We should be able to turn on named, and have it work. Before we do that, let's verify that we can do a zone transfer from the Master DNS Server using dig.

```
#	dig @172.16.103.157 nightfall.bz

; <<>> DiG 9.10.4-P8-RedHat-9.10.4-5.P8.fc25 <<>> @172.16.103.157 nightfall.bz
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3702
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;nightfall.bz.			IN	A

;; AUTHORITY SECTION:
nightfall.bz.		604800	IN	SOA	0ns.nightfall.bz. root.nightfall.bz. 4 604800 86400 2419200 604800

;; Query time: 1 msec
;; SERVER: 172.16.103.157#53(172.16.103.157)
;; WHEN: Tue May 23 21:54:56 EDT 2017
;; MSG SIZE  rcvd: 86

```

As you can see, it works. Let's try turning it on.

```
#	systemctl enable named
#	systemctl start named
```

at that point, it should work. Let's try it!

```
#	dig @172.16.103.133 www.nightfall.bz

; <<>> DiG 9.10.4-P8-RedHat-9.10.4-5.P8.fc25 <<>> @172.16.103.133 www.nightfall.bz
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2038
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.nightfall.bz.		IN	A

;; ANSWER SECTION:
www.nightfall.bz.	604800	IN	CNAME	0ns.nightfall.bz.
0ns.nightfall.bz.	604800	IN	A	172.16.103.157

;; AUTHORITY SECTION:
nightfall.bz.		604800	IN	NS	0ns.nightfall.bz.
nightfall.bz.		604800	IN	NS	1ns.nightfall.bz.

;; ADDITIONAL SECTION:
1ns.nightfall.bz.	604800	IN	A	172.16.103.131

;; Query time: 1 msec
;; SERVER: 172.16.103.133#53(172.16.103.133)
;; WHEN: Tue May 23 21:58:42 EDT 2017
;; MSG SIZE  rcvd: 127


```

As you can see, it works. If it doesn't, try this. Next we will want to change a sysconfig setting to disable IPv6 addresses.

```
#	vi /etc/sysconfig/named
```

and then change the options value to this

```
OPTIONS="-4"
```

## 

I used this Digital Ocean Writeup to make this writeup.

https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-centos-7