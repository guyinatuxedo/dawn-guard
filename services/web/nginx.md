# Nginx

This is a guide for Nginx `1.12.1` on Fedora 27.

## Installation

### Install Ngginx

To install nginx:
```
$ sudo yum install nginx
$ sudo systemctl start nginx
$ sudo systemctl enable nginx
```
also by default, Fedora has IPTables rules that will block access from the website, you can remove them with this command:
```
$ sudo iptables -F
```
### Setup New Virtual Host

To setup a New Virtual Host, first edit the Nginx Conf file:
```
# vim /etc/nginx
```

Next in the conf file, add a new block for your new virtual host. It should look like this:
```
    server {
        listen       192.168.197.130:80;
        server_name  dragonslayer.endless.night.nz;
        root         /usr/share/nginx/dg;
        index dragonslayer.html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```
Some details of my config
* Listens on ip address 192.168.197.130 on port 80
* The name of the server is `dragonslayer.endless.night.nz`
* The web directory is located at `/usr/share/nginx/dg`
* The default file which will be served is `dragonslayer.html`

next create the directory, and add the content:
```
# mkdir /usr/share/nginx/dg
# vim /usr/share/nginx/dg/dragonslayer.html
```

next grant the user nginx is running as (should be `nginx`) permission to the web directory:
```
# chown nginx:nginx  /usr/share/nginx/dg/
# chown -R nginx:nginx /usr/share/nginx/dg/*
```

Then finally just restart nginx and you should be good to go:
```
# systemctl restart nginx
```

### Reverse Proxy
A reverse proxy is essentially a virtual server that will just redirect to another web server. To set it up, first edit your `nginx` config:

```
#   vim /etc/nginx/nginx.conf
```

add another server enctry:
```
server {
        listen 192.168.197.130:8080;

        server_name path.endless.night.nz;

        set $upstream 192.168.197.130;

        location / {

        proxy_pass_header Authorization;
        proxy_pass http://192.168.197.130;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarder-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        }
```
This one has the following options:
*   It listens on the ip address `192.168.197.130` on port `8080`
*   It will redirect the requests to `http://192.168.197.130` on port `80`

lastly you will need to change a SE Linux boolean, to allow for httpd to connect to the network:
```
setsebool -P httpd_can_network_connect 1
```

Then you should be able to restart nginx, and you should be good to go:
```
#   systemctl restart nginx
```

### Setup HTTPS using SSL

First generate the SSL Certificate (will consist of a private key and a public cert) for HTTPS to use:
```
$	sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/nginx/nginx.key -out /etc/ssl/nginx/nginx.crt
``` 

After you run that command, you will be prompted for information regarding your organization.

After that we will want to create a strong Diffie-Hellman group, which is used as part of the process of clients communicating with the server:
```
$	sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Next we will need to configure Nginx to use the SSL public cert and private key we just made for it. First let's create an SSL Conf file which will specify SSL settings for Nginx (like what type of ciphers and protocols to use):

```
$	sudo vim /etc/nginx/conf.d/ssl.conf
```

it should have the following settings:
```
ssl_protocols TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_ecdh_curve secp384r1;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header Strict-Transport-Security "max-age=6307200; includeSubdomains";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
```

Next you will need to make a new virtual host which will run the https site. First edit the main Nginx conf file:
```
$	sudo vim /etc/nginx/nginx.conf
```

add/edit in the following settings:
```
    server {
    	ssl_certificate /etc/ssl/nginx/nginx.crt;
	ssl_certificate_key /etc/ssl/nginx/nginx.key;
	ssl_dhparam /etc/ssl/certs/dhparam.pem;
    
    	listen 192.168.234.157:443 ssl http2 default_server;
	server_name _;
	root /usr/share/nginx/https;

	include /etc/nginx/default.d/*.conf;

	location / {
	}

	}
```

Next you should just have to restart nginx (to apply config changes) and you should be good to go:
```
systemctl restart nginx
```

### Setup Password Authentication

It is possible to require a password to visit a website. To do that, first create a file with the username `guy` and password (it will prompt you while running the `openssl` command):
```
#	echo "guy:" >> /etc/nginx/.htpasswd
#	openssl passwd -apr1 >> /etc/nginx/.htpasswd
```

The file should look like this (may have to reformat it):
```
#	cat /etc/nginx/.htpasswd 
guy:$apr1$WrnefoMy$tHUxsAIGuffVH6PUuW7wd1
```

You can also do this same process using apache utillities in order to create the password file:
```
#	yum install httpd-tools
```

Next we can create the username/password using `htpasswd` (for a different user):
```
#	htpasswd /etc/nginx/.htpasswd tux
```

If it is a new password file, you will need to add the `-c` to specify to create a new password file:
```
#	htpasswd -c /etc/nginx/.htpasswd tux
```

Now that you have created a password file, you can add password authentication to a `server`. First edit the main nginx conf file:
```
#	vim /etc/nginx/nginx.conf
```

add/edit in these settings to the `server` that you want to password protect:

```
location / {
auth_basic "Unspeakable Horrors Lie Ahead";
auth_basic_user_file /etc/nginx/.htpasswd;
}
```

## Config

The config directory for Nginx is:
```
/etc/nginx
```

The main config file is `/etc/nginx/nginx.conf`. Additional configurations are typically stored in `/etc/nginx/conf.d/`.

### nginx.conf settings

To specify the user which Nginx will be running as:
```
user nginx;
```

To specify the number of worker processes which Nginx can spawn (the setting `auto` means the server will decide):
```
worker_processes auto;
```

To specify the error log for Nginx, which stores logs related to errors:
```
error_log /var/log/nginx/error.log;
```

To specify the file which hold the PID (Process ID) which Nginx will run as:
```
pid /run/nginx.pid;
```

To include other config files, in this case conf files in the `modules` directory:
```
include /usr/share/nginx/modules/*.conf;
```

To specify the maximum number of connections which a worker process can have open at a single time:
```
events {
worker_connections 1024;
}
```

To include the config files for the config files in `conf.d`:
```
include /etc/nginx/conf.d/*.conf;
```

### nginx.conf server context

The following configurations will be in the `server` context of the config file `/etc/nginx/nginx.conf`, meaning that they will be inbetween the brackets for server, like this:
```
server {
  # Configs here
}
```

To specify what port/ip to listen on for IPv4 (port 80), and the `default_server` means that it is the default server if a request matches no other server names
```
        listen       80 default_server;
```

To specify that the default server will be listening on the ip `192.168.` on port `80` (if no port is specified, it will listen on port `80`, and if no ip address is specified, it will listen on `0.0.0.0`)
```
        listen       192.168.197.130:80 default_server;
```

To specify what port/ip to listen on for IPv6 (port 80), and the `default_server` means
```
        listen       [::]:80 default_server;
```

To disable listening on IPv6, just comment out that block
```
#        listen       [::]:80 default_server;
```

To specify the server name, which will assist with nginx matching a http request for a specific server with that virtual server. The server name `_` acts as a catch all, for all requests not directed towards another virtual host (other catch alls include `--` and `!@#`):
```
        server_name  _;
```

To specify if Nginx will not send the version number of the Nginx server:
```
server_tokens off;
```

To specify the document root for the virtual host:
```
        root         /usr/share/nginx/html;
```

To specify the default documents to be served as `dragonslayer.html` and `silent-horror.html`:
```
        index dragonslayer.html silent-horror.html;
```

The specify that the default conf files are included in the virtual host:
```
        include /etc/nginx/default.d/*.conf;
```

This option specifes settings that only apply to the directory `/` of the web directory for the virtual server, so in this case the entire web directory:
```
        location / {
        }
```

This specifes that the string `Unspeakable Horrors Lie Ahead` is presented when auser is authenticating:
```
auth_basic "Unspeakable Horrors Lie Ahead";
```

This option specifes that password authentication is enabled, and that the pasword file is `/etc/nginx/.htpasswd`:
```
auth_basic_user_file /etc/nginx/.htpasswd;
```

To specify the error page displayed when there is a `404` error:
```
        error_page 404 /404.html;
            location = /40x.html {
        }
```

To specify the error page displayed when there is any error between `500-599`:
```
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
```

### SSL Configs

To specify the SSL public certificate for a `server` (virtual host):
```
ssl_certificate /etc/ssl/nginx/nginx.crt;
```

To specify the SSL private key for the corresponding public cert for a `server` (virtual host):
```
ssl_certificate_key /etc/ssl/nginx/nginx.key;
```

To specify the Defie-Hellman group for a `server`, which is used in communications inbetween the client and the server with `https`:
```
ssl_dhparam /etc/ssl/certs/dhparam.pem;
```

To specify the SSL protocols which can be used:
```
ssl_protocols TLSv1.1 TLSv1.2;
```

To specify the server ciphers should be used over client ciphers:
```
ssl_prefer_server_ciphers on;
```

To specify which ssl ciphers to use:
```
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
```

To specify the Diffie-Hellman eliptic curve, which is used for a server and client to establish a shared secret over an insecure channel:
```
ssl_ecdh_curve secp384r1;
```

To specify the type, and size of the cache which will store SSL session parameters:
```
ssl_session_cache shared:SSL:10m;
```

To enable or disable session resumptions through TLS:
```
ssl_session_tickets off;
```

To enable or disable OSCP stapling, which is a way to determine if  an SSL Certificate is valid:
```
ssl_stapling on;
```

This specifies the dns resolver which will be used for `OCSP` stapling:
```
resolver 8.8.8.8 8.8.4.4 valid=300s;
```

This specifies the timeout for DNS resolutions for `OSCP` stapling:
```
resolver_timeout 5s;
```

This setting will specify that a client can not back out of the `HSTS` (HTTP Strict Transport Security) policy, for a duration specified by `max-age` in seconds:
```
add_header Strict-Transport-Security "max-age=6307200; includeSubdomains";
```

This will disable a page from being displayed in a frame, or ifreame, which will prevent clickjacking:
```
add_header X-Frame-Options DENY;
```

This will disable content-type sniffing on some browsers:
```
add_header X-Content-Type-Options nosniff;
```
### Nginx Variables

Variables are set by Nginx, and used in several different config files. Here is a list of them, and where they are set:

This variable is equal to the directory root of the virtual host it's serving, it is set equal to the `root` option in `nginx.conf`:
```
$document_root;
```

This variable stores the `uri` (path to something like a webserver) which is being used:
```
$document_uri;
```

```
$fastcgi_script_name;
```

This specifies the `uri` which was originally submitted by the client as part of a request:
```
$request_uri;
```

This specifies the protocol of the request (examples `HTTP/1.0`, `HTTP/1.1`, `HTTP/2.0`): 
```
$server_protocol;
```

This variable stores the scheme of the request, either `http` or `https`
```
$scheme;
```

`https` is set if https is being used, otherwise it is an empty string:
```
$https if_not_empty;
```

These two variables together will show the current `fastcgi` script being ran, by combining the `$document_root` variable with the `$fastcgi_script_name` which is set equal to the current `fastcgi` script being ran:
```
$document_root$fastcgi_script_name;
```

This enviornment variable is used to pass the parameters of the request:
```
$query_string;
```

This variable stores the request method (`POST`, `GET` etc.):
```
$request_method;
```

This variable stores the content type for the request, which could be `text`, `image` etc:
```
$content_type;
```

This specifies the length of the content (the size):
```
$content_length;
```
This variable stores the nginx version being used:
```
$nginx_version;
```

This specifies the ip address which accepted the request:
```
$server_addr;
```

This specifies the port which accepted the request:
```
$server_port;
```

This specifies the name of the server which accepted the request:
```
$server_name;
```

This specifies the ip address of the client:
```
$remote_addr;
```

This specifies the port of the client:
```
$remote_port;
```

### FastCGi

FastCGI (Fast Common Gateway Interface) is a binary protocol for interfacing programs with a web server. There are two conf files dealing with nginx variables for this (that just specify important information) in the nging conf directory `/etc/nginx`. Most of the variables in there have been discussed in the previous section `nginx variables`, however here are some hard coded values that were not shown:

This specifies the gateway interface:
```
fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
```

This just specifies the server software which is running the web server:
```
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;
```

This specifies the status code for redirects:
```
fastcgi_param  REDIRECT_STATUS    200;
```

Here are the two conf files:

`fastcgi.conf`:
```
# cat fastcgi.conf

fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

`fastcgi_params`:
```
# cat fastcgi_params

fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

### scgi

SCGI (Simple Common Gateway Interface) is a binary protocol for applications to interface with web servers, and is an alternative to fastcgi. The config file for scgi dealing with nginx variables is `/etc/nginx/scgi_params`

```
# cat scgi_params

scgi_param  REQUEST_METHOD     $request_method;
scgi_param  REQUEST_URI        $request_uri;
scgi_param  QUERY_STRING       $query_string;
scgi_param  CONTENT_TYPE       $content_type;

scgi_param  DOCUMENT_URI       $document_uri;
scgi_param  DOCUMENT_ROOT      $document_root;
scgi_param  SCGI               1;
scgi_param  SERVER_PROTOCOL    $server_protocol;
scgi_param  REQUEST_SCHEME     $scheme;
scgi_param  HTTPS              $https if_not_empty;

scgi_param  REMOTE_ADDR        $remote_addr;
scgi_param  REMOTE_PORT        $remote_port;
scgi_param  SERVER_PORT        $server_port;
scgi_param  SERVER_NAME        $server_name;
```

### uwsgi

Uwsgi servers a similar purpose to `scgi` and `fastcgi`, where it works to help applications talk to web servers. The config file for `uwsgi` that deals with nginx variables is `/etc/uwsgi_params`:

```
# cat uwsgi_params

uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

### Character mappings

Nginx will have to deal with different character sets (`UTF`, `koi`, `win`). To assist with this, Nginx has some character maps that will convert one character from one character set to another. Here are the three map files:
```
/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/win-utf
```

Here are the actual values of the files:

`koi-utf`:
```
# cat koi-utf

# This map is not a full koi8-r <> utf8 map: it does not contain
# box-drawing and some other characters.  Besides this map contains
# several koi8-u and Byelorussian letters which are not in koi8-r.
# If you need a full and standard map, use contrib/unicode2nginx/koi-utf
# map instead.

charset_map  koi8-r  utf-8 {

    80  E282AC ; # euro

    95  E280A2 ; # bullet

    9A  C2A0 ;   # &nbsp;

    9E  C2B7 ;   # &middot;

    A3  D191 ;   # small yo
    A4  D194 ;   # small Ukrainian ye

    A6  D196 ;   # small Ukrainian i
    A7  D197 ;   # small Ukrainian yi

    AD  D291 ;   # small Ukrainian soft g
    AE  D19E ;   # small Byelorussian short u

    B0  C2B0 ;   # &deg;

    B3  D081 ;   # capital YO
    B4  D084 ;   # capital Ukrainian YE

    B6  D086 ;   # capital Ukrainian I
    B7  D087 ;   # capital Ukrainian YI

    B9  E28496 ; # numero sign

    BD  D290 ;   # capital Ukrainian soft G
    BE  D18E ;   # capital Byelorussian short U

    BF  C2A9 ;   # (C)

    C0  D18E ;   # small yu
    C1  D0B0 ;   # small a
    C2  D0B1 ;   # small b
    C3  D186 ;   # small ts
    C4  D0B4 ;   # small d
    C5  D0B5 ;   # small ye
    C6  D184 ;   # small f
    C7  D0B3 ;   # small g
    C8  D185 ;   # small kh
    C9  D0B8 ;   # small i
    CA  D0B9 ;   # small j
    CB  D0BA ;   # small k
    CC  D0BB ;   # small l
    CD  D0BC ;   # small m
    CE  D0BD ;   # small n
    CF  D0BE ;   # small o

    D0  D0BF ;   # small p
    D1  D18F ;   # small ya
    D2  D180 ;   # small r
    D3  D181 ;   # small s
    D4  D182 ;   # small t
    D5  D183 ;   # small u
    D6  D0B6 ;   # small zh
    D7  D0B2 ;   # small v
    D8  D18C ;   # small soft sign
    D9  D18B ;   # small y
    DA  D0B7 ;   # small z
    DB  D188 ;   # small sh
    DC  D18D ;   # small e
    DD  D189 ;   # small shch
    DE  D187 ;   # small ch
    DF  D18A ;   # small hard sign

    E0  D0AE ;   # capital YU
    E1  D090 ;   # capital A
    E2  D091 ;   # capital B
    E3  D0A6 ;   # capital TS
    E4  D094 ;   # capital D
    E5  D095 ;   # capital YE
    E6  D0A4 ;   # capital F
    E7  D093 ;   # capital G
    E8  D0A5 ;   # capital KH
    E9  D098 ;   # capital I
    EA  D099 ;   # capital J
    EB  D09A ;   # capital K
    EC  D09B ;   # capital L
    ED  D09C ;   # capital M
    EE  D09D ;   # capital N
    EF  D09E ;   # capital O

    F0  D09F ;   # capital P
    F1  D0AF ;   # capital YA
    F2  D0A0 ;   # capital R
    F3  D0A1 ;   # capital S
    F4  D0A2 ;   # capital T
    F5  D0A3 ;   # capital U
    F6  D096 ;   # capital ZH
    F7  D092 ;   # capital V
    F8  D0AC ;   # capital soft sign
    F9  D0AB ;   # capital Y
    FA  D097 ;   # capital Z
    FB  D0A8 ;   # capital SH
    FC  D0AD ;   # capital E
    FD  D0A9 ;   # capital SHCH
    FE  D0A7 ;   # capital CH
    FF  D0AA ;   # capital hard sign
}
```
`koi-win`:
```
# cat koi-win

charset_map  koi8-r  windows-1251 {

    80  88 ; # euro

    95  95 ; # bullet

    9A  A0 ; # &nbsp;

    9E  B7 ; # &middot;

    A3  B8 ; # small yo
    A4  BA ; # small Ukrainian ye

    A6  B3 ; # small Ukrainian i
    A7  BF ; # small Ukrainian yi

    AD  B4 ; # small Ukrainian soft g
    AE  A2 ; # small Byelorussian short u

    B0  B0 ; # &deg;

    B3  A8 ; # capital YO
    B4  AA ; # capital Ukrainian YE

    B6  B2 ; # capital Ukrainian I
    B7  AF ; # capital Ukrainian YI

    B9  B9 ; # numero sign

    BD  A5 ; # capital Ukrainian soft G
    BE  A1 ; # capital Byelorussian short U

    BF  A9 ; # (C)

    C0  FE ; # small yu
    C1  E0 ; # small a
    C2  E1 ; # small b
    C3  F6 ; # small ts
    C4  E4 ; # small d
    C5  E5 ; # small ye
    C6  F4 ; # small f
    C7  E3 ; # small g
    C8  F5 ; # small kh
    C9  E8 ; # small i
    CA  E9 ; # small j
    CB  EA ; # small k
    CC  EB ; # small l
    CD  EC ; # small m
    CE  ED ; # small n
    CF  EE ; # small o

    D0  EF ; # small p
    D1  FF ; # small ya
    D2  F0 ; # small r
    D3  F1 ; # small s
    D4  F2 ; # small t
    D5  F3 ; # small u
    D6  E6 ; # small zh
    D7  E2 ; # small v
    D8  FC ; # small soft sign
    D9  FB ; # small y
    DA  E7 ; # small z
    DB  F8 ; # small sh
    DC  FD ; # small e
    DD  F9 ; # small shch
    DE  F7 ; # small ch
    DF  FA ; # small hard sign

    E0  DE ; # capital YU
    E1  C0 ; # capital A
    E2  C1 ; # capital B
    E3  D6 ; # capital TS
    E4  C4 ; # capital D
    E5  C5 ; # capital YE
    E6  D4 ; # capital F
    E7  C3 ; # capital G
    E8  D5 ; # capital KH
    E9  C8 ; # capital I
    EA  C9 ; # capital J
    EB  CA ; # capital K
    EC  CB ; # capital L
    ED  CC ; # capital M
    EE  CD ; # capital N
    EF  CE ; # capital O

    F0  CF ; # capital P
    F1  DF ; # capital YA
    F2  D0 ; # capital R
    F3  D1 ; # capital S
    F4  D2 ; # capital T
    F5  D3 ; # capital U
    F6  C6 ; # capital ZH
    F7  C2 ; # capital V
    F8  DC ; # capital soft sign
    F9  DB ; # capital Y
    FA  C7 ; # capital Z
    FB  D8 ; # capital SH
    FC  DD ; # capital E
    FD  D9 ; # capital SHCH
    FE  D7 ; # capital CH
    FF  DA ; # capital hard sign
}
```

`win-utf`:
```
# cat win-utf

# This map is not a full windows-1251 <> utf8 map: it does not
# contain Serbian and Macedonian letters.  If you need a full map,
# use contrib/unicode2nginx/win-utf map instead.

charset_map  windows-1251  utf-8 {

    82  E2809A ; # single low-9 quotation mark

    84  E2809E ; # double low-9 quotation mark
    85  E280A6 ; # ellipsis
    86  E280A0 ; # dagger
    87  E280A1 ; # double dagger
    88  E282AC ; # euro
    89  E280B0 ; # per mille

    91  E28098 ; # left single quotation mark
    92  E28099 ; # right single quotation mark
    93  E2809C ; # left double quotation mark
    94  E2809D ; # right double quotation mark
    95  E280A2 ; # bullet
    96  E28093 ; # en dash
    97  E28094 ; # em dash

    99  E284A2 ; # trade mark sign

    A0  C2A0 ;   # &nbsp;
    A1  D18E ;   # capital Byelorussian short U
    A2  D19E ;   # small Byelorussian short u

    A4  C2A4 ;   # currency sign
    A5  D290 ;   # capital Ukrainian soft G
    A6  C2A6 ;   # borken bar
    A7  C2A7 ;   # section sign
    A8  D081 ;   # capital YO
    A9  C2A9 ;   # (C)
    AA  D084 ;   # capital Ukrainian YE
    AB  C2AB ;   # left-pointing double angle quotation mark
    AC  C2AC ;   # not sign
    AD  C2AD ;   # soft hypen
    AE  C2AE ;   # (R)
    AF  D087 ;   # capital Ukrainian YI

    B0  C2B0 ;   # &deg;
    B1  C2B1 ;   # plus-minus sign
    B2  D086 ;   # capital Ukrainian I
    B3  D196 ;   # small Ukrainian i
    B4  D291 ;   # small Ukrainian soft g
    B5  C2B5 ;   # micro sign
    B6  C2B6 ;   # pilcrow sign
    B7  C2B7 ;   # &middot;
    B8  D191 ;   # small yo
    B9  E28496 ; # numero sign
    BA  D194 ;   # small Ukrainian ye
    BB  C2BB ;   # right-pointing double angle quotation mark

    BF  D197 ;   # small Ukrainian yi

    C0  D090 ;   # capital A
    C1  D091 ;   # capital B
    C2  D092 ;   # capital V
    C3  D093 ;   # capital G
    C4  D094 ;   # capital D
    C5  D095 ;   # capital YE
    C6  D096 ;   # capital ZH
    C7  D097 ;   # capital Z
    C8  D098 ;   # capital I
    C9  D099 ;   # capital J
    CA  D09A ;   # capital K
    CB  D09B ;   # capital L
    CC  D09C ;   # capital M
    CD  D09D ;   # capital N
    CE  D09E ;   # capital O
    CF  D09F ;   # capital P

    D0  D0A0 ;   # capital R
    D1  D0A1 ;   # capital S
    D2  D0A2 ;   # capital T
    D3  D0A3 ;   # capital U
    D4  D0A4 ;   # capital F
    D5  D0A5 ;   # capital KH
    D6  D0A6 ;   # capital TS
    D7  D0A7 ;   # capital CH
    D8  D0A8 ;   # capital SH
    D9  D0A9 ;   # capital SHCH
    DA  D0AA ;   # capital hard sign
    DB  D0AB ;   # capital Y
    DC  D0AC ;   # capital soft sign
    DD  D0AD ;   # capital E
    DE  D0AE ;   # capital YU
    DF  D0AF ;   # capital YA

    E0  D0B0 ;   # small a
    E1  D0B1 ;   # small b
    E2  D0B2 ;   # small v
    E3  D0B3 ;   # small g
    E4  D0B4 ;   # small d
    E5  D0B5 ;   # small ye
    E6  D0B6 ;   # small zh
    E7  D0B7 ;   # small z
    E8  D0B8 ;   # small i
    E9  D0B9 ;   # small j
    EA  D0BA ;   # small k
    EB  D0BB ;   # small l
    EC  D0BC ;   # small m
    ED  D0BD ;   # small n
    EE  D0BE ;   # small o
    EF  D0BF ;   # small p

    F0  D180 ;   # small r
    F1  D181 ;   # small s
    F2  D182 ;   # small t
    F3  D183 ;   # small u
    F4  D184 ;   # small f
    F5  D185 ;   # small kh
    F6  D186 ;   # small ts
    F7  D187 ;   # small ch
    F8  D188 ;   # small sh
    F9  D189 ;   # small shch
    FA  D18A ;   # small hard sign
    FB  D18B ;   # small y
    FC  D18C ;   # small soft sign
    FD  D18D ;   # small e
    FE  D18E ;   # small yu
    FF  D18F ;   # small ya
}
```
### MIME types

The config file which maps MIME types to file extensions can be found at `/etc/nging/`:

Here is an example of an entry that maps `audio/x-flac` to `flac`:
```
audio/x-flac                                    flac;
```

## Security

*	Disable Server Tokens
*	Disable unnecissary HTTP methods
*	Clickjacking Prevention
*	XSS Protection
*	SSL
*	IPTables


#### Disable Server Tokens

Nginx by defaultwill display information about the web server, including the version number (can be helpful for picking which Metasploit module to use))
```
server_tokens off;
```

#### Disable unnecissary HTTP methods

For the msot part, the only HTTP methods you need are `GET`, `HEAD`, and `POST`. So you should whitelist these three methods and disable all others, since they could be fairly helpful for an attacker (such as `TRACE`, `DELETE`, `PUT`, and `OPTIONS`). To do that, add in this setting to server blocks you wish to protect:

```
if ($request_method !~ ^(GET|HEAD|POST)$)
{
        return 405;
}
```

#### Clickjacking Prevention

Nginx can defend against clickjacking, by specifing that browsers can only load resources from the same origin, by adding the following config to the `server` block:

```
add_header X-Frame-Options "SAMEORIGIN";
```

#### XSS Protection

Nginx comes with some built in XSS mitigation with HTTP Headers. To enable this, add this config to the `server` block:

```
add_header X-XSS-Protection "1; mode=block";
```

#### SSL

To install and configure HTTPS, I covered this in `Installation` and `Config`.

#### Firewall Rules

For firewall rules, if you are hosting `HTTP` you will need to allow inbound and outbound port `80`. If you are hosting `HTTPS` you will need to allow inbound and outbound port `443`. If you are not using one, you can go ahead and block it:

Input Rules:
```
#	iptables -A INPUT -p TCP --dport 80 -j ACCEPT
#	iptables -A INPUT -p TCP --dport 443 -j ACCEPT
```

Output Rules:
```
#	iptables -A OUTPUT -p TCP --sport 80 -j ACCEPT
#	iptables -A OUTPUT -p TCP --sport 443 -j ACCEPT
```

Default Policies:
```
#	iptables -P INPUT DROP
#	iptables -P OUTPUT DROP
#	iptables -P FORWARD DROP
#	ip6tables -P FORWARD DROP
#	ip6tables -P INPUT DROP
#	ip6tables -P OUTPUT DROP
```

## Logging

The default file which stores nginx error logs is `/var/log/nginx/error.log`. Below is an example of an error, when the `server_tokens` option is specified outside of a `server` block:

```
#	cat /var/log/nginx/error.log
2018/03/05 09:15:31 [emerg] 3310#0: "server_tokens" directive is not allowed here in /etc/nginx/nginx.conf:9
```

Here is the config from `/etc/nginx/nginx.conf` which specifies the error log:
```
error_log /var/log/nginx/error.log;
```

The default file which stores who accessed what on the web server is `/var/log/nginz/access.log`. Below is an example of the a `GET` request from `192.168.234.1`:

```
#	cat /var/log/nginx/access.log
192.168.234.1 - - [05/Mar/2018:08:09:06 -0800] "GET / HTTP/1.1" 200 3700 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0" "-"
```

Here is the config from `/etc/nginx/nginx.conf` which specifies the access log:
```
    access_log  /var/log/nginx/access.log  main;
```

Also the main system logging file (in this case, it's `/var/log/messages`) for your distro can hold a lot of information about `nginx`. Below is an example of logs which signify a syntax problem with an `if` statement outside of a `server` block: 

```
#	cat /var/log/messages
Mar  5 09:26:15 localhost nginx[3492]: nginx: [emerg] "if" directive is not allowed here in /etc/nginx/nginx.conf:109
Mar  5 09:26:15 localhost nginx[3492]: nginx: configuration file /etc/nginx/nginx.conf test failed
```

## Migrating
To migrate Nginx, simply copy over the web directory to the desired server. Then copy over (or just edit to mirror) any configurations that you want to remain persistent.

## Extra

To view your version of Nginx:
```
#	nginx -v
nginx version: nginx/1.12.1
```
