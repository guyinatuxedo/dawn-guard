# Nginx_Reverse_Proxy_Debian 6

### Legacy Repositories

For legacy repoistories go to `http://snapshot.debian.org/`. It is a giant list of practically every debian update, which it stores by months. If you wanted to view all of the updates from 29/02/2016, go to:
```
http://snapshot.debian.org/archive/debian/?year=2016;month=02
```
There you can find the updates for Debian for the entire month. You can pick the closest day that would be to the end of life for the Debian distribution. Once you pick a day/time, just click on it and it will lead you to the url for that snapshot. For Squuexe (Debian 9) which end of lifed on 02/29/2016 I picked `http://snapshot.debian.org/archive/debian/20160229T050110Z/`. Once you pick your snapshot, you can change your repo by editing `/etc/apt/sources.list` like this:

```
deb http://snapshot.debian.org/archive/debian/20160229T050110Z squeeze main
deb-src http://snapshot.debian.org/archive/debian/20160229T050110Z squeeze main
deb http://snapshot.debian.org/archive/debian-security/20160229T050110Z squeeze/updates main
deb-src http://snapshot.debian.org/archive/debian-security/20160229T050110Z squeeze/updates main
```
Make sure to change `squeeze` with the codename of your Debian version. Here is a list with Debian versions, their codenames, and when they End of Lifed:

*	Debian 1.1: Buzz
*	Debian 1.2: Rex
*	Debian: 1.3
*	Debian 2: Hamm	
*	Debian 2.1: Slink		10/30/2000
*	Debian 2.2: Potato	06/30/2003
*	Debian 3.0: woody	06/30/2006
*	Debian 3.1: Sarge		03/31/2008
*	Debian 4.0: Etch		02/15/2010
*	Debian 5.0: Lenny	02/06/2012
*	Debian 6.0: Squeeze	02/29/2014
*	Debian 7.0: Wheezy	05/2018
*	Debian 8.0: Jessie	04/2018
*	Debian 9.0: Stretch

### Setup

##### Installation:

First you might want to install sudo:
```
#	apt-get install sudo
```

To install ngninx:
```
$	sudo apt-get install nginx
```

To start the nginx daemon:
```
$	sudo /etc/init.d/nginx enable
```

Install chkconfig:
```
$	sudo apt-get install chkconfig
```

Enable nginx:
```
$	sudo /sbin/chkconfig --add nginx
$	sudo /sbin/chkconfig nginx on
```

##### Configuration:

First we will need to add some proxy settings to a file:
```
#	cat /etc/nginx/proxy
proxy_set_header Host $http_host;
proxy_set_header X-Real_IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

client_max_body_size 100M;
client_body_buffer_size 1m;
proxy_intercept_errors on;
proxy_buffering on;
proxy_buffer_size 128k;
proxy_buffers 256 16k;
proxy_busy_buffers_size 256k;
proxy_temp_file_write_size 256k;
proxy_max_temp_file_size 0;
proxy_read_timeout 300;
```

Next we will need to edit the site config file for the default site (you can add your own if you want). The IP of the reverse proxy i s`172.16.103.150` and the ip of the http server it's sending to is `172.16.103.148`:

```
#	cat /etc/nginx/sites-available/default
server {

	listen   172.16.103.150:80; ## listen for ipv4

	server_name  localhost;

	access_log  /var/log/nginx/localhost.access.log;

	location / {
		proxy_pass http://172.16.103.148:80;
		include /etc/nginx/proxy;
	}
}
```

Now reload Nginx:
```
#	service nginx reload
```

Now when you access the Nginx server @ 172.16.103.150:80, it should redirect you to 172.16.103.148:80.

### Networking

To set a static ip, edit this file:
```
$	cat /etc/network/interfaces
# The loopback network interface
atuo lo
iface lo inet loopback

# The primarynetwork interface
allow-hotplug etho
iface eth0 inet static
	address 172.16.103.150
	netmask 255.255.255.0
	gateway 172.16.103.2
```

Then you can edit the `resolv.conf` to add the dns servers:
```
/usr/bin/sudo vim /etc/resolv.conf
```

example `resolv.conf`:
```
nameserver 172.16.103.174
```
