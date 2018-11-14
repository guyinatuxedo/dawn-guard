# BindDNS_Solaris

First install bind:
```
$	sudo pkg install service/network/dns/bind
```

Verify that it is installed:
```
$	sudo pkg list | grep -i bind
```

Next create the Solairs DNS config file at `/etc/named.conf`. It will contain the options fo rthe forward and reverse zones:
```
$	sudo nano /etc/named.conf
$	cat /etc/named.conf
options {
		directory "/etc/namedb/working";
		pid-file "/var/run/namedb/pid";
		version "unkown";
		allow-transfer { 172.16.103.141; };
		allow-query { 172.16.103.0/24; };
		forwarders { 8.8.8.8; };
};

zone "blood.lan" {
		type master;
		file "/etc/namedb/master/blood";
};

zone "103.16.172.in-addr" {
		type master;
		file "/etc/namedb/master/103.16.172";
};
```

Next create all of the necissary directories/files that the options require:
```
$	sudo mkdir -p /var/run/namedb
$	sudo mkdir /etc/namedb
$	sudo mkdir -p /etc/namedb/master
$	sudo mkdir -p /etc/namedb/working
$	sudo touch /var/run/namedb/pid
```

Next create the forward zone file:
```
$	cat /etc/namedb/master/blood
$TTL 3h
@		IN	SOA	drop.blood.lan.	root.blood.lan.	(
			2013022744
			3600
			3600
			604800
			38400
)

blood.lan.	IN	NS	drop.blood.
drop	IN	A	172.16.103.141
```

Then create the reverse zone file:
```
$	cat /etc/namedb/master/103.16.172
$TTL 3h
@	IN	SOA	db.blood.	root.db.blood.(
		2013022744
		28800
		3600
		604800
		38400
)
	IN	NS	db.blood.
141	IN	PTR	server.blood
```

enable the service, and confirm it's online:
```
$	sudo svcadm clear dns/server
$	sudo svcadm enable dns/server
$	sudo svcs dns/server
```

lastly, you can test if it is working with Dig.


### Networking

To list all interfaces:
```
#	netadm list
```

To set the `ncp` interface to Static IP addressing:
```
#	netadm enable -p ncp DefaultFixed
```

Next we will need to create a new network interface for the static ip:
```
#	ipadm create-ip net0
```

After that, you can check to see if the interface has been made. It's state should be shown as down:
```
#	ipadm show-if
```

Next we are going to assign the interface to a static ip address:
```
#	ipadm create-addr -T static -a 172.16.103.141/24 net0/acme
```

Next we can check to see if the interface is up, and that it has the correct ip address:
```
#	ipadm show-addr
```

Next we will need to add a static default route:
```
#	route -p add default 172.16.103.2
```

Check to see that the route has been added:
```
#	route -p show
```

Lastly add/edit in your DNS servers:
```
$	cat /etc/resolv.conf
nameserver	172.16.1041
```

### Firewall

First check to see if the firewall service is running:
```
#	svcs -a | grep pfil
```

To enable/start the firewall service:
```
#	svcadm enable svc:/network/ipfilter:default
```

View all current firewall rules:
```
#	ipfstat -io
```

To add/remove firewall rules edit the `ipf.conf` file:
```
#	vim /etc/ipf/ipf.conf
```

To refresh the ipfilter rules:
```
#	ipf -Fa -f /etc/ipf/ipf.conf
```

##### Firewall Rules Configurations
These are all settings stored in the `/etc/ipf/ipf,conf` file.

If you want to allow all traffic through on the `net0` interface:
```
pass in on net0 all
pass out on net0 all
```

This will block packets to small to not be an attack:
```
block in log quick all with short
```
This will by default block all incoming network traffic:
```
block in log on net0 from any to any head 100
b
```
This will allow in 22/tcp:
```
pass in quick on net0 proto tcp from any \
to net0/32 port = 22 flag S keep state group 100
```

This will allow in 53/udp:

```
pass in log quick on net0 proto tcp/udp from any to net0/32 port = 53 keep state
```