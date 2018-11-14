# Postfix Ubuntu 16.04

In order to set this up, you will need to have a DNS server preconfigured. You can refer to the DNS writeup I made, which this writeup will be using the same Bind DNS server from that. In here I will be using test email addresses and domains, which you will need to change to meet your own needs. In addition to Postfix, we will also be setting up Squirrelmail so users can have a web interface to use their emails.

My domain is:
```
nightfall.bz
```

My Email addresses are:
```
raven@nightfall.bz
fang@nightfall.bz
hunter@nightfall.bz
monster@nightfall.bz
alien@nightfall.bz
it@nightfall.bz
```

### Postfix Server Setup

First we will need to install Postfix.

```
#	sudo apt-get update
#	apt-get install postfix
```

Next we will need to edit the main config file for it

```
#	vim /etc/postfix/main.cf
```

Now the first thing you will need to set is the "myhostname" to that of the domain (This setting was named mydomain in previous versions)

```
myhostname = nightfall.bz
```

Now we need to change the server from aliases to virtual aliases. This will allow email from multiple users to be stored on the system. In order to do that, comment out this line (you don't need to do this, but it's good practice)

```
#alias_database = hash:/etc/aliases
```

Next we will need to specify where the virtual file is with the following line, which stores the email address that we have.

```
virtual_maps = hash:/etc/postfix/virtual
```

We are done with the config file. Next we will need to create the virual file which holds all of the email addresses. In addition to that, what it does specifically is it maps an email account to a user account. When an email account recieves an email, it stores it in the mailbox of the localuser's account that it is mapped to. For this i created a user account for each email, that is the string before the "@nightfall.bz".


```
#	cat /etc/postfix/virtual
raven@nightfall.bz		raven
fang@nightfall.bz		fang
hunter@nightfall.bz	hunter
monster@nightfall.bz	monster
alien@nightfall.bz		alien
it@nightfall.bz		it
```

With that we finish the configuration for Postfix, so we should be able to start it up.

```
#	systemctl enable postfix
#	systemctl start postfix
```

### MX Record

Even though the Postfix server is up and running, we need to add an MX record on the DNS record. A MX record is a mail record, which is used to resolve the domain name section (@nightfall.bz) into the ip address of the mail server. For this we need to edit the Forward Zone File on the master DNS server.

```
#	cat /etc/named/dns_zones/db.nightfall.bz 
; This is our Forwarding Zone

; SOA record
@	IN	SOA	0ns.nightfall.bz. root.nightfall.bz. (
		4
		604800
		86400
		2419200
		604800
)
;	DNS NS records
nightfall.bz.	IN	NS	0ns.nightfall.bz.
nightfall.bz.	IN	NS	1ns.nightfall.bz.

; Our DNS Server A Records
0ns.nightfall.bz.		IN	A	172.16.103.157
1ns.nightfall.bz.		IN	A	172.16.103.133

; Our DNS Client
client.nightfall.bz.		IN	A	172.16.103.125
mail.nightfall.bz.		IN	A	172.16.103.134

		
; CNAME record
www	IN	CNAME	0ns.nightfall.bz.

; Mail records
nightfall.bz.			IN	MX	10	mail.nightfall.bz.
```

As you can see I have an A record for the mail server which will allow us to resolve "mail.nightfall.bz" to the ip address of our mail server (172.16.103.134). Then I have an MX record which will point all mail heading to nightfall.bz to mail.nightfall.bz. Also you see that the MX record has the number "10", which signifies it's priority. The lower the priority signifies that it will be first mail server to be used in case it need to use multiple (and it will proceed in the order from lowest to highest priority number). Since we only have one, it doesn't matter too much.

Lastly we just need to restart the DNS named service to have our updated zone take effect.

```
#	systemctl restart named
```

### Testing Postfix

Now that we have installed and configured our Postfix system, and added our MX record, we should be ready to send an email to the Mail server. For this, we will be using telnet and the mail server's DNS name which we configured in the previous section "mail.nightfall.bz". I am running this on a different machine that is using the same DNS server, and has network access to the mail server.

```
#	telnet mail.nightfall.bz smtp
Trying 172.16.103.134...
Connected to mail.nightfall.bz.
Escape character is '^]'.
220 nightfall.bz ESMTP Postfix (Ubuntu)
mail from: hunter@nightfall.bz
250 2.1.0 Ok
rcpt to: alien@nightfall.bz
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
Remember the greatest moments lived in their lifetime, for another night. 
.
250 2.0.0 Ok: queued as 1550146C0A
quit
221 2.0.0 Bye
Connection closed by foreign host.
```

As you can see, we sent an email to raven@nightfall.bz that says "Remember the greatest moments lived in their lifetime, for another night.". Let's see if it worked using the mail server.

```
#	cat /var/mail/alien
From hunter@nightfall.bz  Wed May 24 17:31:10 2017
Return-Path: <hunter@nightfall.bz>
X-Original-To: alien@nightfall.bz
Delivered-To: alien@nightfall.bz
Received: from 0ns.nightfall.bz (0ns.nightfall.bz [172.16.103.157])
	by nightfall.bz (Postfix) with SMTP id 1550146C0A
	for <alien@nightfall.bz>; Wed, 24 May 2017 17:30:40 -0700 (PDT)

Remember the greatest moments lived in their lifetime, for another night.

```

As you can see, we recieved an email for the alien user which we sent mail to containing the same message we sent. So it works.

### Dovecot setup

First we will need to install Dovecot

```
#	apt-get update
#	apt-get install dovecot-core dovecot-imapd
```

After that is done, we will need to configure the default mail locations for dovecot. The config file for that is here

```
#	vim /etc/dovecot/conf.d/10-mail.conf
```

In it, you should see a line that resembles this

```
mail_location = mbox:~/mail:INBOX=/var/mail/%u
```

right now the mailbox is being stored in the user's home folder. We are going to change that to /var/mail/mailbox, and let dovecot create it (btw %u means the user account's name). 

```
mail_location = mbox:/var/mail/mailbox/%u:INBOX=/var/mail/%u
```

With that done, the last thing we should need to do is handle permissions. Thing is when the user account logs in to Squirrelmail, it uses the user's account to do everything. So we are going to need to add all of the email user's to a group, change the group owner of /var/mail to that group, then change the permissions of "/var/mail" to 670 (of course this is all to prevent chmod 777).

First add all of the users to the mail group
```
#	addgroup mail
#	adduser raven mail
#	adduser fang mail
#	adduser hunter mail
#	adduser monster mail
#	adduser alien mail
#	adduser it mail
```

Next we need to change the group ownership of "/var/mail", and the permissions if it to allow read and write to the onwer, read write execute to the group, and nothing for everyone else

```
#	chgrp mail /var/mail
#	chmod -R 670 /var/mail
```

Lastly we should delete everything from /var/mail, that way dovecot will create it, and make sure apache and dovecot are enabled and running

```
#	rm -rf /var/mail*
#	systemctl enable apache2
#	systemctl enable dovecot
#	systemctl start apache2
#	systemctl start dovecot
```

### Squirrelmail Setup

Now we will work on setting up Squirrelmail, which will act as a web front end for users to log in and use their email. We will first need to install it.

```
#	apt-get update
#	apt-get install squirrelmail
```

Next we will need to configure apache to point to squirrelmail, which it doesn't by default. To do this, you will need to edit the default site file (the exact path may very)

```
#	vi /etc/apache2/sites-enabled/000-default.conf
```

Now in between the two Virtual Host tags that specify the site's settings and look like this

```
<VirtualHost *:80>
</VirtualHost>
```

you are going to want to input an alias, which maps the directory which holds squirrelmail's files to a subdirectory of the website

```
Alias	/squirrelmail /usr/share/squirrelmail
```

With that, squirrelmail should be ready. Just navigate in a web browser to the ip address of your mail server followed by "/squirrelmail". To log in, just use the local user account's credentials that are tied to the email account. Also for logging in or sending emails, you don't need to include the domain (@nightfall.bz).

```
http://172.16.103.134/squirrelmail/
```

### Set Static IP

To set a static IP in Ubuntu, first you need to know your adapter name which you can find using an ifconfig command.

```
#	ifconfig
ens33     Link encap:Ethernet  HWaddr 00:0c:29:db:96:cc  
          inet addr:172.16.103.134  Bcast:172.16.103.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fedb:96cc/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:13233 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4773 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:10028038 (10.0 MB)  TX bytes:419291 (419.2 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:299 errors:0 dropped:0 overruns:0 frame:0
          TX packets:299 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:22642 (22.6 KB)  TX bytes:22642 (22.6 KB)

```

As you can see here, our ip address is 172.16.103.134 and out Network Adapter is ens33. Now to set a static ip we will need to edit this file

```
#	vim /etc/network/interfaces
```

If these lines are present, or lines similar to it are present, comment them out (make sure not to effect other network adapters)

```
#auto ens33
#iface ens33 inet dhcp
```

Now we will add out own statci ip information with the following lines

```
auto ens33
iface ens33 inet static
address 172.16.103.134
netmask 255.255.255.0
gateway 172.16.103.1
dns-nameservers 172.16.103.133
```

With that, we are done with the config file. Now we just need to flush the ip address and restart the networking daemon to properly apply the change.

```
#	ip addr flush ens33 && systemctl restart networking
```

### Writeups

This writeups was made using the following writeups:

https://www.digitalocean.com/community/tutorials/how-to-install-and-setup-postfix-on-ubuntu-14-04

http://www.configserverfirewall.com/ubuntu-linux/ubuntu-set-static-ip-address/









