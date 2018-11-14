# IPTables

#### Basic Config:

List IPTables rules:
```
$	sudo iptables -L
```

To list IPTables rules with numbers:
```
$	sudo iptables -L --line-numbers
```

List IPTables rules for a specific chain, in this case ```INPUT```:
```
$	sudo iptables -L INPUT
```

View the default policies:
```
$	sudo iptables -S
```

Change the default policy for a chain, in this case ```INPUT``` to DROP all packets:
```
$	sudo iptables -P INPUT DROP
```

To delete all IPTables rules (doesn't affect default policy):
```
$	sudo iptables -F
```

To delete all IPTables rules for a particular chain (In this case INPUT):
```
$	sudo iptables -F INPUT
```

To delete the ```IN_public_deny``` chain:
```
$	sudo iptables -X IN_public_deny
```

#### Rule Creation:



To add a rule to allow inbound udp traffic on port 53:
```
$	sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT
``` 

To add a rule to reject New outbound tcp traffic on port 22, that has been established:
```
$	sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate NEW -j REJECT
```

To add a rule to drop incoming udp traffic on port 80 for Established and Related connections:
```
$	sudo iptables -A INPUT -p udp --dport 80 -m conntrack --ctstate ESTABLISHED,RELATED -j DROP
```

To add a rule allowing incomming UDP traffic on port 54 (DNS) for the ip address 172.16.103.134:
```
$	sudo iptables -A INPUT -p udp --dport 53 -s 172.16.103.134 -j ACCEPT
```

To add a rule allowing outgoing UDP traffic on port 54 (DNS) to the ip address 172.16.103.134:
```
$	sudo iptables -A OUTPUT -p udp --sport 53 -d 172.16.103.134 -j ACCEPT
```

Legacy rules for new, established, related:
```
$	sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT
```

#### Deleting Rules:

To list all the rules by numbers:
```
$	sudo iptables -L --line-numbers
```

To delete a the 2nd rule from the INPUT chain:
```
$	sudo iptables -D INPUT 2
```

#### Saving Rules:

##### Ubuntu

o make the IPTables rules persistent, first export the rules to a :
```
#	iptables-save > /etc/iptables-rules
```

Next edit the `rc.local` file to have the iptables rules added on startup:
```
$	sudo vim /etc/rc.local
```

Next add the following line, so it will run a command to add the iptables rules on boot:
```
iptables-restore < /etc/iptables-rules
```

next, make sure that the permissions for `rc.local` is `770`:
```
$	sudo chmod 770 /etc/rc.local
```

##### CentOS

```
#	yum install iptables-services
```

```
#	/usr/libexec/iptables.iptables.init save
```

##### rhel

```
$	sudo /sbin/service iptables save
```
#### Logging:
To view all of the logs:
```
$	sudo iptables -L -v
```

To view all of the logs for the INPUT chain:
```
$	sudo iptables -L INPUT -v
```

To view the logs for the 1st rule in the INPUT chain:
```
$	sudo iptables -L INPUT 1 -v
```

To cleare the logs for the 2nd rule in the INPUT chain:
```
$	sudo iptables -Z INPUT 2
```

#### Basic Firewall Server Inbound Rules:

##### Disable IPv6

```
$	sudo ip6tables -P INPUT DROP
$	sudo ip6tables -P OUTPUT DROP
$	sudo ip6tables -P FORWARD DROP
```

##### Client Inbound ICMP, DNS, and Yum
```
$	sudo iptables -A INPUT -p icmp -m conntrack --ctstate ESTABLISHED -j ACCEPT
$	sudo iptables -A INPUT -p udp --sport 53 -m conntrack --ctstate ESTABLISHED -j ACCEPT
$	sudo iptables -A INPUT -p tcp --sport 80 -m conntrack --ctstate ESTABLISHED -j ACCEPT
```

##### SSH (server):
```
$	sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

##### HTTP:
```
$	sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

##### MySQL:
```
$	sudo iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
```

##### Squirrelmail, IMAP, SMTP:
```
$	sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
$	sudo iptables -A INPUT -p udp --sport 53 -m conntrack --ctstate ESTABLISHED -j ACCEPT
$	sudo iptables -A INPUT -p tcp --dport 143 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --sport 143 -m conntrack --ctstate ESTABLISHED -j ACCEPT
$	sudo iptables -A INPUT -p tcp --dport 25 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --sport 25 -m conntrack --ctstate ESTABLISHED -j ACCEPT
```


##### DNS:
```
$	sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
```

### IPtables Full Path
The full path to iptables for Ubuntu, Centos, Mint is:

```
/sbin/iptables
```

The full path to iptables for OpenSUSE is:

```
/usr/sbin/iptables
```

#### Extra:
##### DNS Masking
If your dns server address is 127.0.1.1, you have DNS masking which iptables can screw with. Essentially you have dns masking listening on port 53, which iptables can block. To disable it in Ubuntu edit this file.

```
$	sudo vim /etc/NetworkManager/NetworkManager.conf
```

and comment out this line:

```
$dns=dnsmadq
```

Then just restart the Network Manager daemon and you should be good to go (if that doesn't work, restart it):
```
$	sudo systemctl retsart network-manager.service
```
##### Firewalld

If you're running CentOS, you might want to uninstall Firewalld which shits all over iptables with it's chains.

```
$	sudo yum remove firewalld
```
#### Basics:

##### Chains
* INPUT: This Chain handles all incoming packets.
* OUTPUT: This Chain handles all outgoing packets.
* FORWARD: These are packets that are neither from or to the host, but the host is probably just routing them.

##### Packet Treatments 
* ACCEPT:	Allows the packet through
* DROP: Just drops the packet without saying anything
* Reject: Drops the packet but sends a message stating that it was rejected

##### IPTables States
* NEW:	This is a new connection, such as first time reachine a web server.
* Established:	This is a connection that has already been made, such as the outbound connection for an ssh server being remoted into.
* RELATED:	This is a new connection that is associated with an existing connection.

##### Common Ports
* Ftp/Telnet: 21
* SSH: 22
* Telnet: 23
* SMTP: 25
* DNS: 53
* TFTP: 69
* HTTP: 80
* Kerberos: 88
* POP3: 110
* NMPT: 119
* IMAP: 143
* LDAP: 389
* Direct Connect: 411-412
* HTTPS (SSL): 443
* Microsoft DS: 445
* Kerberos: 464
* SMTP (SSL): 465
* syslog: 514
* SMPT: 587
* LDAP (SSL): 636
* Exchange: 691
* ESXI: 902
* FTP (SSL): 989-990
* IMAP4 (SSL): 993
* POP3 (SSL): 995
* Microsoft RPC: 1025
* OpenVPN: 1194
* Halo: 2302
* MySQL: 3306

##### Rules vs Default Policy:
If a packet patches a rule in a chain, then what the rule says should happen will. If a packet doesn't meet any rules, then whatever the default policy is will happen.
	
