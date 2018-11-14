# php-fpm
This is a guide for php-fpm, which is php that runs on as a server, which can listen on a port.

## Installation

To install php-fpm:
```
$	sudo yum install php php-fpm
```

##### Setup Nginx to use PHP-FPM

First install nginx, and start it:
```
$	sudo yum install nginx
$	sudo systemctl start engine
```

might want to flush firewall rules, so you can reach it:
```
$	sudo iptables -F
```

Now you should ensure that nginx and php-fpm are talking to each other on the same socket. To ensure this, make sure that the following config options are compatible (simplest way, make sure that their values are compatible).

For php-fpm, check this file:
```
#	vim /etc/php-fpm.conf
```

This is the config 
```
listen = /run/php-fpm/www.sock
```
if you want it to be on an IP address and port `192.168.234.163:8080`:
```
listen = 192.168.234.163:8080
```

For Nginx, check this file:
```
#	vim /etc/nginx/conf.d/php-fpm.conf
```

here is the option to check:
```
upstream php-fpm {
server unix:/run/php-fpm/www.sock;
}
```

if you want it to be over the ip address and port `192.168.234.163:8080`:
```
upstream php-fpm {
server 192.168.234.163:8080;
}
```

You will also need to set an SE Linux bool, so Nginx can access the network to reach php-fpm, if you are talking to it over an ip address and port:
```
setsebool httpd_can_network_connect 1 -P
```

## Security

Here is a brief look at what you can do to lockdown php-fpm:

##### PHP-FPM
*	Chroot
*	Specify what clients can run code
*	Privileges of the service
*	Where is it listening 
* IPTables rules

##### PHP
Check the normal PHP writeup for info regarding that

##### PHP-FPM: Chroot

Chroot will essentially restrict the activity of php-fpm to a signle directory. This can be done by changing the following config:
```
#	vim /etc/php-fpm.d/www.conf
```

add these configurations which will essentially specify that php-fpm is not allowed to interact with anything outside of the directory Nginx should host websites out of `/usr/share/nginx`:
```
prefix = /usr/share/nginx/
chroot = $prefix
```

You will also need to change the document root for Nginx:
```
root         /html/;
```

##### PHP-FPM: Clients

PHP-FPM has a setting which will specify what ip addresses are allowed to connect to PHP-FPM and run scripts. To sepcif that, edit this file:

```
#	vim /etc/php-fpm.d/www.conf
```

To only allow the ip address `192.168.234.163` to connect and run php code:

```
listen.allowed_clients = 192.168.234.163
```

##### PHP-FPM: Privileges

You can sepcify the permissions which php-fpm will run with (and the socket) by changing the user/group which it runs as. To do that edit this file:

```
#	vim /etc/php-fpm.d/www.conf
```

to configure the user/group which php-fpm will run as:
```
user = nginx
group = nginx
```
To sepcify the user/group which the unix socket will run with:

```
listen.owner = nginx
listen.group = nginx
```

To sepcify an access control list user/group which the unix socket will run with. This will override the above `listen.owner` and `listen.group` setiings:

```
listen.acl_users = nginx
listen.acl_groups = nginx

```

##### PHP-FPM: listening

Make sure that php is listening on a socket that will expose it the least (so use a socket that isn't binded to an IP address, or use once that is binded to 127.0.0.1 if you need to, unless if the php-fpm and nginx servers are on different boxes):

```
#	vim /etc/php-fpm.d/www.conf
```

to have it talk over a socket that isn't binded to an IP address:
```
listen = /run/php-fpm/www.sock
```
to have it talk over a socket binded to the ip address and port `127.0.0.1:8080`:
```
listen = 127.0.0.1:8080
```
##### IPTables rules

With IPTables, we will whitelist the connections that we need to make, and block everything else. These rules are for a server that both hosts php-fpm, and needs to talk to it for a web app. If you are only hosting it, you should be good with the first rule from each, and if you are only a client to php-fpm you should be good with only the second rule from each. Php-fpm in this configuration is talking over port `8080` and the only server and client is `192.168.234.163`:

for input:
```
$	sudo iptables -A INPUT -p tcp --dport 8080 -s 192.168.234.163 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --sport 8080 -s 192.168.234.163 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

for output:
```
$	sudo iptables -A OUTPUT -p tcp --sport 8080 -d 192.168.234.163 -j ACCEPT
$ sudo iptables -A OUTPUT -p tcp --dport 8080 -d 192.168.234.163 -j ACCEPT
$	sudo iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
```

block everything else:
```
$ sudo iptables -P INPUT DROP
$ sudo iptables -P OUTPUT DROP
$ sudo iptables -P FORWARD DROP
$ sudo ip6tables -P INPUT DROP
$ sudo ip6tables -P OUTPUT DROP
$ sudo ip6tables -P FORWARD DROP
```

## Logs

Here is an Nginx log which details a failed attempt to get `php-fpm` to run code, since it is not coming from a whitelisted IP client address:
```
#	cat /var/log/nginx/error.log
2018/03/07 18:55:57 [error] 7971#0: *1 upstream prematurely closed connection while reading response header from upstream, client: 192.168.234.1, server: _, request: "GET /index.php HTTP/1.1", upstream: "fastcgi://192.168.234.163:8080", host: "192.168.234.163"
```

Here is an example of a blacklisted php function attemping to be ran:
```
#	cat /var/log/nginx/error.log
2018/03/08 13:21:27 [error] 4162#0: *5 FastCGI sent in stderr: "PHP message: PHP Warning:  exec() has been disabled for security reasons in /html/index.php on line 3" while reading response header from upstream, client: 192.168.234.1, server: _, request: "GET /index.php HTTP/1.1", upstream: "fastcgi://192.168.234.163:8080", host: "192.168.234.163"
```
## Migrate

To migrate php-fpm install it on another box and either transfer the configs, or edit the configs to mirror the original.

## Extra

To see what version of php-fpm you are running:
```
#	php-fpm -v
PHP 7.1.15 (fpm-fcgi) (built: Feb 28 2018 11:19:18)
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
```
