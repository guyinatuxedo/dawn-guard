# BindDNS Master/Slave Configuration

Bind and Named are two names for essentially the same thing. I will be using the two names interchangebly. 

## Installation

This will be across two different servers:
```
blood.moon.endless.night	192.168.234.147	Ubuntu 16.04
dawnbringer.endless.night	192.168.234.148	CentOS 7
```

#### Ubuntu Primary DNS Server Installation & Setup

First install bind
```
$	sudo apt-get update
$	sudo apt-get install bind9 bind9utils bind9-doc
```

Next we will want to set BindDNS to IPv4 mode. To do this, we can edit this file:

```
$	sudo vim /etc/default/bind9
```

and add/edit this line:
```
OPTIONS="-4 -u bind"
```

Next we are going to need to add the configurations to edit the named options config file, to specify settings for our named server:
```
$	sudo vim /etc/bind/named.conf.options
```

First we will need to specify an Access Contol List (ACL), so we establish who can use this server. We can just add this to the top of the file:
```
acl "servers" {
	192.168.234.147	#	Our Primary Name Server
	192.168.234.148	#	Our Secondary Name Server
};
```

Proceeding that, in the options block we will need to add/edit in these settings for the `options` block:
```
options {
		directory "/var/cache/bind"; # This is the DNS server's working location, which will store most of it's output
		recursion yes; # Enables Recursive Lookups
		allow-recursion { servers; }; # This defines who is allowed to do recursive lookups, which are all ips in the previous ACL
		listen-on { 192.168.234.147; }; # This defines what IP address the server will listen on
		allow-transfer { servers; }; # This specifies who is allowed to do a zone transfer, which essentially will pull all DNS records from the zone
		forwarders { # This specifies what DNS servers our server will forward to, if it doesn't have a DNS record for the request
			8.8.8.8;	# These two DNS servers are Google's public DNS servers
			8.8.4.4;
		};
		dnssec-validation auto;  # dnssec-validation is essentially a method of validating dns requests by using trusted keys. The options are no (disable), yes (required), or auto (enabled, but default trust anchor is used)
		listen-on-v6 { none; }; #Specify that the server should not be listening on any IPv6 addresses
		auth-nxdomain no; # Specifies that the server says that it isn't an authoritative server when it gets a DNS request for something it doesn't have (nxdomain)
};
``` 

Next we are going to edit the conf file, to specify the settings for our zone. We will be making two zones, a forward zone and a reverse zone. The forward zone will be used to request an IP address from a DNS name. The reverse zone will be used to request a DNS name from an IP address :
```
$	sudo vim /etc/bind/named.conf.local
```

add/edit the folowing settings to match your setup:
```

# Specify our primary zone
zone "endless.night.nz" {
	type master; # Specify that this is a master Server for this zone
	file "/etc/bind/zones/night.nz"; # Specify the file which stores the DNS record information
	allow-transfer { servers; }; # Specify what servers are allowed to do a zone transfer, which we reuse the ACL from the previous conf file
};

# The name for the reverse zone should be the network portion of the IP address reversed. The IP addrss for the server is 192.168.234.141/24

# Specify the reverse zone
zone "234.168.192.in-addr.arpa" { 
	type master; # Specify that this is a master Server for ths zone
	file "/etc/bind/zones/rev-endless.night.nz"; # Specify the file which stores the DNS record information
	allow-transfer { servers; }; # Specify what servers are allowed to do a zone transfer, which we reuse the ACL from the previous conf file
};
```

Next create the directory where the zone files will be stored:
```
$	sudo mkdir /etc/bind/zones/
```

Now create the first zone file:

```
$	sudo vim /etc/bind/zones/endless.night.nz
```

The forward lookup zone file will have these records:

```
$TTL 86400 ; Defines the duration in seconds that a record can be chached
; Start of Authority (SOA) record, specifies information such as Authoritative server, and chaching information
@	IN	SOA		blood.moon.endless.night.nz.	root.endless.night.nz. ( ; Beginning of the SOA record, which specifies the Authoritative server, and the email of the Admin
		2		;	sn = Number whis is incremented each time zone updates, used as version number for zone
		3600	;	refresh = Number of seconds which the slave will try to refresh from master
		1800	;	update retry = Number of seconds which the slave will wait before contacting the master again while refreshing if failed
		604800	;	Expiry = Number of seconds which if passed with no contact with master, the records are no longer valid
		86400	;	nxdomain ttl = Number of seconds which a negative response is cached
)

; This specifies the Name Servers for this zone (NS records)
	IN	NS	blood.moon.endless.night.nz.
	IN	NS	dawnbringer.endless.night.nz.
	
; This specifies the DNS Names, and the corresponding IP address (A records)
blood.moon	IN	A	192.168.234.147
dawnbringer	IN	A	192.168.234.148
```

Next create the reverse zone file:
```
$	sudo vim /etc/bind/zones/rev-night.nz
```

The reverse zone file will have these records:

```
$TTL 86400 ; Defines the duration in seconds that a record can be chached
; Start of Authority (SOA) record, specifies information such as Authoritative server, and chaching information
@	IN	SOA		blood.moon.endless.night.nz.	root.endless.night.nz. ( ; Beginning of the SOA record, which specifies the Authoritative server, and the email of the Admin
		2		;	sn = Number whis is incremented each time zone updates, used as version number for zone
		3600	;	refresh = Number of seconds which the slave will try to refresh from master
		1800	;	update retry = Number of seconds which the slave will wait before contacting the master again while refreshing if failed
		604800	;	Expiry = Number of seconds which if passed with no contact with master, the records are no longer valid
		86400	;	nxdomain ttl = Number of seconds which a negative response is cached
)

; This specifies the Name Servers for this zone (NS records)
	IN	NS	blood.moon.endless.night.nz.
	IN	NS	dawnbringer.endless.night.nz.
	
; This specifies the ip addresses, which are associated with the DNS names.
147	IN	PTR	blood.moon.endless.night.nz.
148	In	PTR	dawnbringer.endless.night.nz
```

Now that all of our Bind config files have been made/edited to our needs, we can check the config files:
```
$	named-checkconf
```

then we can start/enable the service:

```
$	sudo systemctl start bind9
$	sudo systemctl enable bind9
```

Next we just have to add our DNS server to the list of DNS server's we will be using. To do that, edit this file:

```
$	sudo vim /etc/resolvconf/resolv.conf.d/head
```

then you can just add/edit the folloing configurations:
```
search night.nz $ Our domains which DNS servers we are using
nameserver 192.168.234.141 # The IP address of our name server
```

Next we can just refresh the DNS settings for our client:

```
$	sudo resolvconf -u
```

Now we can test the forward and reverse zones. First we will test the forward zone by tring to resolve an IP address from a DNS name using `dig`:
```
$	dig ns0.night.nz

; <<>> DiG 9.10.3-P4-Ubuntu <<>> ns0.night.nz
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47994
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;ns0.night.nz.			IN	A

;; ANSWER SECTION:
ns0.night.nz.		86400	IN	A	192.168.234.141

;; AUTHORITY SECTION:
night.nz.		86400	IN	NS	ns0.night.nz.

;; Query time: 0 msec
;; SERVER: 192.168.234.141#53(192.168.234.141)
;; WHEN: Tue Feb 06 00:31:53 EST 2018
;; MSG SIZE  rcvd: 71
```

next we will test resolving a DNS name from an IP address to test the reverse zone:
```
$	dig -x 192.168.234.141

; <<>> DiG 9.10.3-P4-Ubuntu <<>> -x 192.168.234.141
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21021
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;141.234.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
141.234.168.192.in-addr.arpa. 604800 IN	PTR	nso.night.nz.

;; AUTHORITY SECTION:
234.168.192.in-addr.arpa. 604800 IN	NS	ns0.night.nz.

;; ADDITIONAL SECTION:
ns0.night.nz.		86400	IN	A	192.168.234.141

;; Query time: 0 msec
;; SERVER: 192.168.234.141#53(192.168.234.141)
;; WHEN: Tue Feb 06 00:33:31 EST 2018
;; MSG SIZE  rcvd: 117
```

and just like that, we can see that both zones are working!

#### Centos Secondary DNS Server 

Fist install the bind server:

```
$	sudo yum -y update
$	sudo yum -y install bind bind-utils
```

Next edit the named conf file:

```
$	sudo vim /etc/named.conf
```

this is the acl which we will be using (only IP addresses in this ACL can query the DNS Server):
```
acl "clients" { // Our acl will be named clients
	192.168.234.147 // Our First client, which is the authoritative DNS server
	192.168.234.148 // Our Second client, which is this server
};
```

the options block will look like this:
```
options {
		listen-on port 53 { 192.168.234.148; };
		listen-on-v6 { none; };
		directory	"/var/named";
		dump-file "/var/named/data/cache_dump.db";
		statistics-file "/var/named/data/named_stats.txt";
		memstatistics-file "/var/named/data/named_me_";
		allow-query	{ clients; };

		recursion yes;
		allow-transfer{ none; };
		
		dnssec-enable yes;
		dnssec-validation yes;
		bindkeys-file "/etc/named.iscdlv.key";
		managed-keys-directory "/var/named/dynamic";
		pid-file "/run/named/named.pid";
		session-keyfile "/run/named/session.key";
};
```

Now for the zone configurations, we will be editing the same config file `/etc/named.conf`. First we will created the directory which will store the zone files (zone files will automatically be generated by Bind when it pulls the DNS records from the primary sever):
```
$	sudo mkdir /etc/slave-zones
```

Next we can add the Zone configurations by just appending them to the end of `/etc/named.conf`:
```
# Create the zone configurations for the forward zone
zone "endless.night.nz" {
	type slave;	# Specify that it is a slave server
	file "/var/named/slaves/night/nz";	# Specify the file which will sotre the DNS records
	masters { 192.168.234.147; };	# Specify the master which will server the DNS records to us
	allow-transfer { none; }; # Specify nobody is supposed to do a zone transfer
};

# Create the zone configurations for the reverse zone
zone "234.168.192.in-addr.arpa" {
	type slave;	# Specify that it is a slave server
	file "/var/named/slaves/rev-endless.night.nz";	# Specify the file which will sotre the DNS records
	masters { 192.168.234.147; };	# Specify the master which will server the DNS records to us
	allow-transfer { none; }; # Specify nobody is supposed to do a zone transfer
};
```

also you might want to comment out the following default zone:
```
/*
zone "." IN {
        type hint;
        file "named.ca";
};
*/
```

Check the configurations:
```
$	sudo named-checkconf
```

If there is no output, there were no configuration errors detected. Next you can start and enable the service:

```
$	sudo systemctl start named
$	sudo systemctl enable named
Created symlink from /etc/systemd/system/multi-user.target.wants/named.service to /usr/lib/systemd/system/named.service.
```

Then your dns server will be functional!

## Config

The master config file for Bind on Ubuntu is `/etc/bind/named.conf`, and for CentOS is `/etc/named.conf`.

To include an additional config file:
```
include "/etc/bind/named.conf.options";
```

To Specify an Access Control List, which is essentially a name for a group of IP Addresses:
```
acl "servers" {
        192.168.234.149;
        192.168.234.151;
        192.168.234.1;
};
```

To enable recursion, so the DNS server can actaully answer DNS record resolutions:
```
recursion yes;
```

To specify who the server will resolve DNS queries for:
```
allow-recursion { servers;};
```

also

To specify the directory which bind should work out of, this should be the directory which SELinux will allow Bind to write to:
```
directory "/var/named";
```

To specify what ip address and port Bind should listen on:
```
listen-on port 52 { 192.168.234.147; };
```

To specify what IPv6 addresses Bind will listen on:
```
listen-on-v6 { none; };
```

To specify who can do a zone transfer:
```
allow-transfer { servers; };
```

To specify what DNS servers this server will forward requests to if it can't resolve it (known as forwarders):
```
forwarders {
        8.8.8.8;
        8.8.4.4;
};
```

To specify what it it is a slave/master:
```
type master;
```

To specify what file the zone will use, if it is a master it will read from it, if it is a slave it will store the records there:
```
file "/etc/bind/zones/rev-endless.night.nz.signed";
```

To specify who can update this DNS zone:
```
allow-update {none};
```

To specify ip addresses/ports the master DNS server is listening on:
```
masters port 52 { 192.168.234.147; };
```

To specify a zone:
```
zone "endless.night.nz" {
	/* Zone config here*/
};
```

To specify what file the output is stored when the command `rndc dumpdb` is run:
```
dump-file       "/var/named/data/cache_dump.db";
```

To specify the file that statistics information is stored when the `rndc stats` command is run:
```
statistics-file "/var/named/data/named_stats.txt";
```

To specify the file which Bind will dump memory statistics:
```
memstatistics-file "/var/named/data/named_mem_stats.txt";
```

To sepcify that this server will use dnssec for things such as zone transfers and DNS updates:
```
dnssec-enable yes;
``` 

To specify that this server will try to resolve named against singed zones:
```
dnssec-validation yes;
```

This file holds the default keys used by dnssec.
```
bindkeys-file "/etc/named.iscdlv.key";
```

This directory will hold the keys made by dnssec, if you have it automatically generate keys.
```
managed-keys-directory "/var/named/dynamic";
```

This file holds the Process ID which Bind will use. If this setting is not available, this file should be found in `/var/run/named.pid` (or just search for `named.pid`)
```
pid-file "/run/named/named.pid";
```

This file holds the TSIG session key, which is used when the command `nsupdate -l` (that just sends a DNS update for a dynamic DNS server) is run: 
```
session-keyfile "/run/named/session.key";
```

This setting will specify if Bind will reply say that it is an authoritative server (the AA bit is set) when saying that the domain is not set (known as returning NXDOMAIN):
```
auth-nxdomain no;
```

##### Bind DNS records

To specify a SOA (Start of Authority) record, which just specifies needed admin data about the zone:

```
$TTL 86400 ; Defines the duration in seconds that a record can be chached
; Start of Authority (SOA) record, specifies information such as Authoritative server, and chaching information
@   IN  SOA     blood.moon.endless.night.nz.    root.endless.night.nz. ( ; Beginning of the SOA record, which specifies the Authoritative server, and the email of the Admin
        2       ;   sn = Number whis is incremented each time zone updates, used as version number for zone
        3600    ;   refresh = Number of seconds which the slave will try to refresh from master
        1800    ;   update retry = Number of seconds which the slave will wait before contacting the master again while refreshing if failed
        604800  ;   Expiry = Number of seconds which if passed with no contact with master, the records are no longer valid
        86400   ;   nxdomain ttl = Number of seconds which a negative response is cached
)
```

To specify a NS (Nameserver) record, which just startes a nameserver:
```
	IN	NS	blood.moon.endless.night.nz.
```

To specify an A record (simple name record), which just maps an IP address to a DNS name:
```
blood.moon	IN	A	192.168.234.147
```

To specify a CNAME (Canonical name) record just maps one DNS name to another:
```
bm      IN      CNAME   blood.moon.
; The name bm is mapped to the DNS name blood.moon
```

To specify an MX record (Mail Server), which just specifies the mail server for a domain:
```
nightfall.bz.			IN	MX	10	mail.nightfall.bz.
; This record will route mail for the nightfall.bz domain to mail.nightfall.bz with the preference 10, lower preferences are more preferred
```

To specify a PTR record (reverse A record), which maps a DNS name to an IP address:
```
147	IN	PTR	blood.moon.endless.night.nz.
; The 147 is the client portion of the ip address blood.moon.endless.night.nz is mapped to
```

## Security

For securing a DNS Server, there are mainly four things you need to worry about:

*	Zone Transfers
*	IPTables rules
*	Unpatched Version
*	Faking authoritative/secondary server, and doing things like injecting false DNS records

#### Zone Transfers

This is a pretty simple fix. In the configuration settings for each zone (`named.conf` for Centos, `named.conf.local` for Ubuntu) there is a setting titled `allow-transfer{};`. For authoritative servers, only secondary dns servers should be listed. For secondary dns servers, no ip address should be listed.

for authoritative servers:
```
acl "servers" {
    192.168.234.147 
    192.168.234.148 
};

zone "endless.night.nz" {
	type master; 
	file "/etc/bind/zones/night.nz";
	allow-transfer { servers; }; 
};
```

for secondary servers:
```
zone "endless.night.nz" {
	type slave;	
	file "/var/named/slaves/night/nz";
	masters { 192.168.234.147; };
	allow-transfer { none; };
};
```

#### IPTables

Here are the IPTables rules which will allow bind DNS to work with ingress and egress filtering. Essentially it talks to itself on port `953` (port server talks to bind on), port `53` (port dns server listens on for requests), and `22` (for ssh) along with `icmp` (so ping still works):
```
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 953 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 953 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT 
iptables -A INPUT -p udp --dport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p udp --sport 53 -s <server_ip_here> -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -P INPUT DROP
```

here are the corresponding ouput rules:
```
iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p udp --sport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 953 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 953 -s 127.0.0.1 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p udp --sport 53 -s <server_ip_here>  -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT
iptables -P OUTPUT DROP
```
#### Unpatched Version

For unpatched versions, first check what version you are running:
```
$	named -v
BIND 9.9.4-RedHat-9.9.4-51.el7_4.2 (Extended Support Version)
```

Check what CVEs are currently published for that version, and update it if you see fit. You may need to compile from source to do this.

#### DNSSec

DNSSec is essentially a way for primary/secondary DNS servers to authenticate themsevles and prove the authenticity of DNS records. 

##### DNSSec Master Config

For this, you can have Bind handle it, or you can do it yourself. To have Bind handle it

First make sure these settings are included in your Bind config file (`named.conf` for CentOS, `named.conf.options` for Ubuntu)For this, you can have Bind handle it, or you can do it yourself. To have Bind handle it have these settings (and stop following this guide):
```
dnssec-enable yes;
dnssec-validation auto;
managed-keys-directory "/etc/named/dynamic";
```

TO do it yourself (and continue following this guide) add/edit these configs in the same file:
```
dnssec-enable yes;
dnssec-validation yes;
dnssec-lookaside auto;
```

Next we are going to have to generate keys for the zones. To do this relatively quickly, you should install `haveged`. 

To install it on Ubnuntu:
```
$	sudo apt-get install haveged
```

also make sure this conf file has this setting:
```
cat /etc/default/haveged 
# Configuration file for haveged

# Options to pass to haveged:
#   -w sets low entropy watermark (in bits)
DAEMON_ARGS="-w 1024"
guyinatuxedo@ubuntu:/et
```

To install it on CentOS:
```
#	yum install haveged
```

Then you can generate the Zone Signing Keys (ZSK) for the zones:
```
$	cd /etc/bind/zones/
$	sudo dnssec-keygen -a NSEC3RSASHA1 -b 2048 -n ZONE endless.night.nz
Generating key pair.......................................+++ ......................+++ 
Kendless.night.nz.+007+28412
$	sudo dnssec-keygen -a NSEC3RSASHA1 -b 2048 -n ZONE 234.168.192.in-addr.arpa
Generating key pair.......................+++ ............................................................................................................+++ 
K234.168.192.in-addr.arpa.+007+13115
```

Then you will create the Key Signing Keys:
```
$	sudo dnssec-keygen -f KSK -a NSEC3RSASHA1 -b 4096 -n ZONE endless.night.nz
Generating key pair...................................................................................................................................................++ ................................................................................................................................................................................++ 
Kendless.night.nz.+007+20178
$	sudo dnssec-keygen -f KSK -a NSEC3RSASHA1 -b 4096 -n ZONE 234.168.192.in-addr.arpa
Generating key pair.............................................................................................................................................................................................................................................................................++ .....++ 
K234.168.192.in-addr.arpa.+007+00168
```

Now in this directory we have 8 keys, a public and private key for each ZSK (Zone Signing Key) and KSK (Key Signing Key), for each zone. We will need to include the public keys in the zone file, which can be done with a little bash scripting:
```
#	for key in `ls Kendless*.key`
> do
> echo "\$INCLUDE $key" >> endless.night.nz
> done
#	for key in `ls K234*.key`; do echo "\$INCLUDE $key" >> rev-endless.night.nz; done
```

Next we can sign the zones files:
```
#	dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | sha1sum | cut -b 1-16) -N INCREMENT -o endless.night.nz -t endless.night.nz 
Verifying the zone using the following algorithms: NSEC3RSASHA1.
Zone fully signed:
Algorithm: NSEC3RSASHA1: KSKs: 1 active, 0 stand-by, 0 revoked
                         ZSKs: 1 active, 0 stand-by, 0 revoked
endless.night.nz.signed
Signatures generated:                       11
Signatures retained:                         0
Signatures dropped:                          0
Signatures successfully verified:            0
Signatures unsuccessfully verified:          0
Signing time in seconds:                 0.020
Signatures per second:                 526.920
Runtime in seconds:                      0.034
#	dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | sha1sum | cut -b 1-16) -N INCREMENT -o 234.168.192.in-addr.arpa -t rev-endless.night.nz 
Verifying the zone using the following algorithms: NSEC3RSASHA1.
Zone fully signed:
Algorithm: NSEC3RSASHA1: KSKs: 1 active, 0 stand-by, 0 revoked
                         ZSKs: 1 active, 0 stand-by, 0 revoked
rev-endless.night.nz.signed
Signatures generated:                       10
Signatures retained:                         0
Signatures dropped:                          0
Signatures successfully verified:            0
Signatures unsuccessfully verified:          0
Signing time in seconds:                 0.013
Signatures per second:                 743.052
Runtime in seconds:                      0.017
```

The portion dealing with `urandom` and the `sha1sum` acts as a salt with 16 characters.

With that, there should be two new zone files created `endless.night.nz.signed` and `rev-endless.night.nz.signed`. We will need to edit the bind conf files to include the new zone files:
```
zone "endless.night.nz" {
        type master;
        file "/etc/bind/zones/endless.night.nz.signed";
        allow-transfer { servers; };
};

zone "234.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/zones/rev-endless.night.nz.signed";
        allow-transfer { servers; };
};
```

after that, restart Bind:
```
$	sudo systemctl restart bind9
```

Proceeding that, you should be able to use dig to verify that DNSSec was implemented:
```
$	dig DNSKEY endless.night.nz.

; <<>> DiG 9.10.3-P4-Ubuntu <<>> DNSKEY endless.night.nz.
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27076
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;endless.night.nz.		IN	DNSKEY

;; ANSWER SECTION:
endless.night.nz.	86400	IN	DNSKEY	256 3 7 AwEAAbvTLlswmNEvIRFOEfFVO/jTBZGzmVr/FVsC86VsWmsSGug/BEGy xrKigngfmbh6RsWYHNb6ZQea9iACasIeCGXoaQu1GuOwFEybHXG3Q8HA DsILIPSm+Bc0G0XfXfAaSfgd/CTNunmZenyRoalvNPhQDNT7JUiiU/dQ ye1c/N5kYEI3WB7NX6hyP4uTZONFiL/P5040ZCw3GHC6VESEiwM4khV3 9tX0Op8NWPNbugJJDm3XLskG83J14pwxkKYRquOfxbT6RExJpvPTt0Pk EjL0Bhbkij03Ta5OgXrYS5mgQZz85PmDfTgNtQ+Nl8oRgD2l/n8yBSS8 5SrlyUjlOjU=
endless.night.nz.	86400	IN	DNSKEY	257 3 7 AwEAAb/LX0w8eMdZOWguZY2xyGNDK5C/86jM0ECPvzbhIy6wgiil684S zA9NBbmxvs+6pyHbRB3/qZZ7CsXLDAR1/7XircNEoy7g3Ha5dYmhvgPb rk6DnsXAAFEvSn049ouDxAjhawcgdHmg3oM4ET0RkVikZ9jCjg4MFKQK O9d3pxty8OvVvCYmHACoPICdtz/2Hi3wSSguGCmYF9m9Jt7zuKXxgUWQ uBSsbWhFXNjIZreChUPR9gbPPWdnv3ak6zIdxa3x+dCTP0WXfsHjdhx0 nSF1NrPM/8ZyitMo43aKH8GpcfaApmIAdWSWWXxl1QTI5gjhNigysepL vEI1r4XLERP234tUfxMRxdKBfBTuDLgri3M2/QOOSPqi/awtRKCxfyaw QHaoo2nqMl1zwTNkKQJp47H0y+MtsttRBGgBxlJ7XRynJ5vJJba6xSMb g+CD1eN6EZgbmyhBEp5UA2urCH8kL3AXI3AjempN1XMOu1BxAqbHRbEF O60E3XkH48eD+gDYxos3zEFop3lEgWuMeyKuZqKcGXvbc2eOniY7mXfx mt3rRPqa9lQ48vhNqP9Dt5QT2jO+PGdhXbVlCJxWplp4XzLxyp9O9c9E qC+LEFCmRZxdK7leDxOBkx8wdmuL8GvBVM8KTaampYnh5OnkelZ2goC2 Ge0vxyZuEFK9j0or

;; Query time: 0 msec
;; SERVER: 192.168.234.147#53(192.168.234.147)
;; WHEN: Sun Feb 25 17:09:36 PST 2018
;; MSG SIZE  rcvd: 853
```

For the slave server, you only have to change the config file to use dnssec (assuming that the master server has had DNSSEC configured). To do that add these configurations to your Bind config file (`/etc/named.conf` for CentOS, `/etc/bind/named.conf.options` for Ubuntu):
```
dnssec-enable yes;
dnssec-validation yes;
dnssec-lookaside auto;
``` 

Proceeding that, just change the file which the records are stored in to the signed version (append `.signed` to the end):
```
zone "endless.night.nz" {
        type slave;
        file "/var/named/slaves/endless.night.nz.signed";
        masters { 192.168.234.147; };
        };

zone "234.168.192.in-addr.arpa" {
        type slave;
        file "/var/named/slaves/rev-endless.night.nz.signed";
        masters { 192.168.234.147; };
        };
```

after that, you should just be able to restart named, and you will hvae DNSSec setup, and you will be ready:
```
#	systemctl restart named
```
## Logs

Bind by default logs some stuff (such as zone transfers). By Default these are the logging locations for various linux versions:
```
Ubuntu 8.04:	/var/log/syslog
Ubuntu 16.04:	/var/log/syslog
```

This is what a successful transfer of the zone `jurassic.world` looks like:
```
Feb 21 08:11:00 guyinatuxedo-desktop named[5273]: client 192.168.234.149#48438: transfer of 'jurassic
.world/IN': AXFR started
Feb 21 08:11:00 guyinatuxedo-desktop named[5273]: client 192.168.234.149#48438: transfer of 'jurassic
.world/IN': AXFR ended
```

This is what a failed transfer of the zone `jurassinc.world` (typo of jurassic.worl) looks like:
```
Feb 21 08:10:55 guyinatuxedo-desktop named[5273]: client 192.168.234.149#37829: bad zone transfer req
uest: 'jurassinc.world/IN': non-authoritative zone (NOTAUTH)
```

Now to log specific queries, you can do so using `rndc`. To toggle rndc querying on/off in Ubuntu 8.04:
```
$	sudo rndc querylog
```

Now with it on, you should be able to see queries in the corresponding file mentioned above, such as the following query for an `A` record for 
```
Feb 21 08:21:53 guyinatuxedo-desktop named[5273]: client 192.168.234.149#44284: query: velociraptor.jurassic.world IN A +
```

This is what a PTR record lookup looks like:
```
Feb 21 13:24:23 ubuntu named[3092]: client 192.168.234.147#48791 (147.234.168.192.in-addr.arpa): query: 147.234.168.192.in-addr.arpa IN PTR +E (192.168.234.147)
```

`rndc` does not give information as to what the query returns in these log files.

and just block IPv4 Forwarding, and all of IPv6:
```
iptables -P FORWARD DROP
ip6tables -P FORWARD DROP
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
```

## Migrating

To Migrate this service, it is fairly easy. If it is a primary DNS server, just setup a new instance of Bind then transfer the Bind zone files. If it is a secondary DNS server, install bind and just point it to the authoritative server.

## Extra

SELinux prevents Named (atleast on CentOS 7) from wrtiing to files that are not in the following directories:

```
/var/named/slaves
/var/named/data
/var/tmp
``` 

In Ubuntu 8.04, this is a valid path for slave zone files:
```
file "/var/cache/bind/jurassic.world";
```
#### Dig

Dig is a useful tool for testing if your DNS server is functioning. Here are some following commands to do various queries:

To do a simple query to resolve the DNS name `velociraptor.jurassic.world`:
```
$	dig velociraptor.jurassic.world

; <<>> DiG 9.4.2-P2.1 <<>> velociraptor.jurassic.world
;; global options:  printcmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17100
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1

;; QUESTION SECTION:
;velociraptor.jurassic.world.	IN	A

;; ANSWER SECTION:
velociraptor.jurassic.world. 86400 IN	A	192.168.234.149

;; AUTHORITY SECTION:
jurassic.world.		86400	IN	NS	blood.moon.jurassic.world.
jurassic.world.		86400	IN	NS	velociraptor.jurassic.world.

;; ADDITIONAL SECTION:
blood.moon.jurassic.world. 86400 IN	A	192.168.234.147

;; Query time: 0 msec
;; SERVER: 192.168.234.149#53(192.168.234.149)
;; WHEN: Wed Feb 21 07:50:47 2018
;; MSG SIZE  rcvd: 116
```

To do a simple query to resolve the DNS name `` against the DNS server `192.168.234.147`:
```
dig @192.168.234.147 velociraptor.jurassic.world  

; <<>> DiG 9.4.2-P2.1 <<>> @192.168.234.147 velociraptor.jurassic.world
; (1 server found)
;; global options:  printcmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18824
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1

;; QUESTION SECTION:
;velociraptor.jurassic.world.	IN	A

;; ANSWER SECTION:
velociraptor.jurassic.world. 86400 IN	A	192.168.234.149

;; AUTHORITY SECTION:
jurassic.world.		86400	IN	NS	blood.moon.jurassic.world.
jurassic.world.		86400	IN	NS	velociraptor.jurassic.world.

;; ADDITIONAL SECTION:
blood.moon.jurassic.world. 86400 IN	A	192.168.234.147

;; Query time: 1 msec
;; SERVER: 192.168.234.147#53(192.168.234.147)
;; WHEN: Wed Feb 21 07:51:55 2018
;; MSG SIZE  rcvd: 116
```

To do a zone transfer for the domain `jurassic.world` against the DNS server `192.168.234.147`:
```
$	dig axfr jurassic.world @192.168.234.147

; <<>> DiG 9.4.2-P2.1 <<>> axfr jurassic.world @192.168.234.147
;; global options:  printcmd
jurassic.world.		86400	IN	SOA	blood.moon.endless.night.nz. root.endless.night.nz. 2 3600 1800 604800 86400
jurassic.world.		86400	IN	NS	blood.moon.jurassic.world.
jurassic.world.		86400	IN	NS	velociraptor.jurassic.world.
blood.moon.jurassic.world. 86400 IN	A	192.168.234.147
velociraptor.jurassic.world. 86400 IN	A	192.168.234.149
jurassic.world.		86400	IN	SOA	blood.moon.endless.night.nz. root.endless.night.nz. 2 3600 1800 604800 86400
;; Query time: 0 msec
;; SERVER: 192.168.234.147#53(192.168.234.147)
;; WHEN: Wed Feb 21 07:53:52 2018
;; XFR size: 6 records (messages 1, bytes 220)
```

To do a reverse lookup for the IP for the PTR record for the ip address `192.168.234.148`:
```
dig -x 192.168.234.148

; <<>> DiG 9.10.3-P4-Ubuntu <<>> -x 192.168.234.148
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44642
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;148.234.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
148.234.168.192.in-addr.arpa. 86400 IN	PTR	dawnbringer.endless.night.nz.

;; AUTHORITY SECTION:
234.168.192.in-addr.arpa. 86400	IN	NS	dawnbringer.endless.night.nz.
234.168.192.in-addr.arpa. 86400	IN	NS	blood.moon.endless.night.nz.

;; ADDITIONAL SECTION:
blood.moon.endless.night.nz. 86400 IN	A	192.168.234.147
dawnbringer.endless.night.nz. 86400 IN	A	192.168.234.148

;; Query time: 0 msec
;; SERVER: 192.168.234.147#53(192.168.234.147)
;; WHEN: Wed Feb 21 12:56:07 PST 2018
;; MSG SIZE  rcvd: 170
```

To verify if DNSSec is enabled:
```
$	dig DNSKEY endless.night.nz.

; <<>> DiG 9.10.3-P4-Ubuntu <<>> DNSKEY endless.night.nz.
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27076
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;endless.night.nz.		IN	DNSKEY

;; ANSWER SECTION:
endless.night.nz.	86400	IN	DNSKEY	256 3 7 AwEAAbvTLlswmNEvIRFOEfFVO/jTBZGzmVr/FVsC86VsWmsSGug/BEGy xrKigngfmbh6RsWYHNb6ZQea9iACasIeCGXoaQu1GuOwFEybHXG3Q8HA DsILIPSm+Bc0G0XfXfAaSfgd/CTNunmZenyRoalvNPhQDNT7JUiiU/dQ ye1c/N5kYEI3WB7NX6hyP4uTZONFiL/P5040ZCw3GHC6VESEiwM4khV3 9tX0Op8NWPNbugJJDm3XLskG83J14pwxkKYRquOfxbT6RExJpvPTt0Pk EjL0Bhbkij03Ta5OgXrYS5mgQZz85PmDfTgNtQ+Nl8oRgD2l/n8yBSS8 5SrlyUjlOjU=
endless.night.nz.	86400	IN	DNSKEY	257 3 7 AwEAAb/LX0w8eMdZOWguZY2xyGNDK5C/86jM0ECPvzbhIy6wgiil684S zA9NBbmxvs+6pyHbRB3/qZZ7CsXLDAR1/7XircNEoy7g3Ha5dYmhvgPb rk6DnsXAAFEvSn049ouDxAjhawcgdHmg3oM4ET0RkVikZ9jCjg4MFKQK O9d3pxty8OvVvCYmHACoPICdtz/2Hi3wSSguGCmYF9m9Jt7zuKXxgUWQ uBSsbWhFXNjIZreChUPR9gbPPWdnv3ak6zIdxa3x+dCTP0WXfsHjdhx0 nSF1NrPM/8ZyitMo43aKH8GpcfaApmIAdWSWWXxl1QTI5gjhNigysepL vEI1r4XLERP234tUfxMRxdKBfBTuDLgri3M2/QOOSPqi/awtRKCxfyaw QHaoo2nqMl1zwTNkKQJp47H0y+MtsttRBGgBxlJ7XRynJ5vJJba6xSMb g+CD1eN6EZgbmyhBEp5UA2urCH8kL3AXI3AjempN1XMOu1BxAqbHRbEF O60E3XkH48eD+gDYxos3zEFop3lEgWuMeyKuZqKcGXvbc2eOniY7mXfx mt3rRPqa9lQ48vhNqP9Dt5QT2jO+PGdhXbVlCJxWplp4XzLxyp9O9c9E qC+LEFCmRZxdK7leDxOBkx8wdmuL8GvBVM8KTaampYnh5OnkelZ2goC2 Ge0vxyZuEFK9j0or

;; Query time: 0 msec
;; SERVER: 192.168.234.147#53(192.168.234.147)
;; WHEN: Sun Feb 25 17:09:36 PST 2018
;; MSG SIZE  rcvd: 853
```

To dig on a specific port `52`:
```
$	dig dawnbringer.endless.night.nz -p 52 @192.168.234.147

; <<>> DiG 9.10.3-P4-Ubuntu <<>> dawnbringer.endless.night.nz -p 52 @192.168.234.147
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21852
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;dawnbringer.endless.night.nz.	IN	A

;; ANSWER SECTION:
dawnbringer.endless.night.nz. 86400 IN	A	192.168.234.148

;; AUTHORITY SECTION:
endless.night.nz.	86400	IN	NS	blood.moon.endless.night.nz.
endless.night.nz.	86400	IN	NS	dawnbringer.endless.night.nz.

;; ADDITIONAL SECTION:
blood.moon.endless.night.nz. 86400 IN	A	192.168.234.147

;; Query time: 0 msec
;; SERVER: 192.168.234.147#52(192.168.234.147)
;; WHEN: Sun Feb 25 18:39:49 PST 2018
;; MSG SIZE  rcvd: 128


```
