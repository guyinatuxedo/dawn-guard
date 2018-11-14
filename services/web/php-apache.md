# PHP for Apache

This is a guide for PHP for Apache web servers on Debian 9 with PHP version 7.0.27

## Installation

Just to install php:
```
#	apt-get install php
```

If you are using this with lamp, you will want to install the php mod for apache (so apache can use php), and the mysql and mcrypt mods for PHP (so PHP can connect to the mysql db, and can encrypt data):
```
#	apt-get install libapache2-mod-php php-mcrypt php-mysql
```

## Config

All of these settings will be from the `php.ini` file. With this setup, that is located under `/etc/php/7.0/apache2/php.ini`.

#### language configs php.ini

This is to enable PHP scritpting language for Apache
```
engine = On
```

This setting will disable the user of PHP short tags (`<?` and `?>`) to specify php code
```
short_open_tag = Off
```

This is to specify how many digits a floating point number is displayed:
```
precision = 14
```

To specify the amount of data php will keep before sending it to the client. If it exceeds this limit, php will send the data to the client in chunks of roughly this size
```
output_buffering = 4096
```

This setting will allow you to redirect all of the output of your scripts to a php function such as `mb_output_handler()`
```
output_handler = 
```

This setting will enable `zlib` compression, so essentially all files sent will be compressed before being sent, than decompressed when they reach the clients
```
zlib.output_compression = Off
```

This specifies the level of compression zlib will use (0-9 9 is the highest). `-1` let's the server pick the compression level:
```
zlib.output_compression_level = -1
```

This setting will set a handler for `zlib` compression, which will just pipe all of the ouptut from the `zlib` compression through a php function: 
```
zlib.output_handler =
```

This option if enabled, will make php flush  it's output every time it sends output: 
```
implicit_fulsh = Off
```

This specifies the callcback function to be called if the unserializer finds an undefined class which should be instantiated:
```
unserialize_callback_func = 
```

This specifies the amount of signifcant features which a float will maintain when serialized:
```
serialize_precision = 17
```

If this option is et, it limits all file operations to the defined directory below:
```
open_basedir = 
```

This option will allow you to blacklist php functions, here we are blacklisting `pcntl_alarm()`, `pcntl_fork()`, `pcntl_waitpid()`, and `phpinfo()`
```
disable_functions = pcntl_alarm, pcntl_fork, pcntl_waitpid, phpinfo
```

Like `disable_functions` this is a blacklist for php classes:
```
disable_classes = 
```

This setting if enabled, will allow for a request to complete, even if the user aborts the request:
```
ignore_user_abort = On
```

This option specifies the size of the `realpath_cache`, which is a cache that stores filepaths to prevent extensive disk lookups:
```
realpatch_cache_size = 4096k
```

This specifies the amount of time in seconds which a filepath will remain in the cache:
```
realpath_cache_ttl = 120
```

##### Miscellaneous configs php.ini

This setting if turned on, will allow php to reveal the fact that it is installed on the server:
```
expose_php = Off
```

#### Resource Limits configs php.ini

This specifes the maximum execution time of each script in seconds:
```
max_execution_time = 30
```

This specifies the maximum amount of time in seconds which a script may spend parsing request data:
```
max_input_time = 60
```

This specifies the maximum depth (number of nesting levels) allowed for an input variable:
```
max_input_nesting_level = 64
```

This specifies the maximum amount of `GET/POST/COOKIE` input variables which will be accepted:
```
max_input_vars = 1000
```

This specifies the maximum amount of memory which a php script may use:
```
memory_limit = 128M
```

#### Logging configs php.ini

This specifies what is going to be logged, which is `E_ALL` (shows all errors, warnings, and notices) however `E_DEPRECATED` (warns about code which will not work in future versions) and `E_STRICT` (run time notices which will ensure the best practice for your code) are skipped:
```
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
```

This displays if, and how php will display errors:
```
display_errors = Off
```

This will display errors related to the startup sequence for PHP:
```
display_startup_errors = Off
```

This specifies if php will log errors, and it should store them in the `error_log` config below:
```
log_errors = On
```

This specifies the maximum length for an error log:
```
log_errors_max_len = 1024
```

This setting will allow for the logging of repeated errors:
```
ignore_repeated_errors = Off
```

This setting if enable will ignore the source for things creating repeated error messages:
```
ignore_repeated_source = Off
```

If this setting is turned on, then memory leaks will be shown in the error logs:
```
report_memleaks = On
```

If this is turned on, the last error/warning message will be stored in $php_errormsg
```
track_errors = Off
```

If this setting is turned on, php error messages will be stored as html:
```
html_errors = On
```

This specifes a string which is prepended to error logs:
```
error_prepend_string = "<span style='color: #ff0000'>"
```
This specifies a string which is appended to error logs:
```
error_append_string = "</span>"
```

This setting specifies where error logs are stored:
```
error_log = syslog
```

#### Data Handling configs php.ini

to specify the arguments separator for php generated URLs:
```
arg_separator.output = "%amp;"
```

List of spearators used by PHP to parse input URLs into variables:
```
arg_separator.input = ";&"
```

This value specifes which super global arrays are registered when PHP starts up (options are G, P, C, E, & S):

```
variables_order = GPCS
```

This specifies what should be registered in the super global  super global data (options are G, P & C)

```
request_order = "GP"
```

This specifies whether php will registers (allow and use) `$argv` and `argc` each time it runs:

```
register_argc_argv = Off
```

This will specify to create the `ENV`, `REQUEST`, and `SERVER` variables when they are first used versus when the script starts. If the variables are not used, it results in a performance boost:

```
auto_globals_jit = On
```

This specifies the maximum size for a post request:

```
post_max_size = 8M
```

This will automatically add files before a PHP document:

```
auto_prepend_file = 
```

This will automatically add files after a PHP document:

```
auto_append_file = 
```

This specifies the default media type:
```
default_mimetype = "text/html"
```

This specifies the default charset for strings:
```
default_charset = "UTF-8"
```

This specifies the character encoding php uses for internal representation of strings, if it's blank then the default_charset is used:

```
interneal_encoding =
```

This specifies the character encoding php uses for input, if it's blank then the default_charset is used:

```
input_encoding =
```

This specifies the character encoding php uses for output, if it's blank then the default_charset is used:

```
output_encoding = 
```



#### File Uploads configs php.ini

This will enable uploading of files to the web server in php:
```
file_uploads = On
```

This will specify a temporary directory which files uploaded iwth php will be stored:
```
upload_tmp_dir =
```

This specifies the maximum size that a file can be uploaded with php (here it is 2 megabytes):
```
upload_max_filesize = 2M
```

This will specify the maximum amount of files which can be uploaded with a single request (here it is `20`):
```
max_file_uploads = 20
```

#### Fopen configs php.ini

This will allow for the treatment of URLs as files:
```
allow_url_fopen = On
```

This will allow the php commands `include/require` to open URLs as files:
```
allow_url_include = Off
```

This specifies the email adress which is sent as the value for from, for unathenticated ftp connextions:
```
from = "silent-horror@endless.night.nz"
```

This specifies the user-agent which PHP will send (default is empty):
```
user_agent="PHP"
``` 

This establishes the timeout in seconds for socket based streams
```
default_socket_timeout = 60
```
This line will enable automatic finding of End of Line characters, so `fgets` and `file` will work
```
auto_detect_line_endings = off
```

#### Paths and Directory configs php.ini

```
include_path = ".:/usr/share/php"
```

```
doc_root =
```

To specify what directory to open when the script uses `~`:
```
user_dir
```

This specifies whether or not to enable the dl() function, which can load a php extension at runntime
```
enable_dl = Off
```

This forbids people from calling PHP directly, and forces them to do it with a redirect:
```
cgi.force_redirect = 1
```

This will force cgi (which tells a webserver how to pass data to and from a webserver) to always return a status 200 request:

```
cgi.nph = 1
```

This option is a string that is set, so when redirection happens it let's the server know if continued execution is ok. It should be disabled for security reasons:

```
cgi.redirect_status_env =
```

This setting essentially allows for php to edit URLs. On older versions, it should be disabled since it could allow for remote code execution:

```
cgi.fix_pathinfo = 1
```

This will allow for the PHP CGI binary to be placed outside of the web directory, so people won't be able to circumvent .htaccess security:

```
cgi.discard_path = 1
```

This turns on logging when using FastCGI (if enabled):

```
fastcgi.lggong = 0
```

This specifies what type of headers to use while sending HTTP response codes. If it set to 0, php will send a `RFC 3875` status. If it set to 1, it will set `RFC 2616` headers. 

```
cgi.rfc2616_headers = 0
```

This specifies if CGI PHP will check for the string `#!` (known as shebang) at the top of the script, and honors it's content:

```
cgi.check_shebang_line = 1
```

#### mysqli configs php.ini

`mysqli` is a php library which will grant the abillity to connect to, and interface with the mysqldb. This section will start with this tag:

```
[MySQLi]
```

This is the maximum number of mysql connections which can be made, `-1` or `0` means infinite:
```
mysqli.max_persistent = -1
```

This option allows for php to access local files with `LOAD DATA` statements:
```
;mysqli.allow_local_infile = On
```

This option allows for persistent connections, which are connections that do not close after the php script ends:

```
mysqli.allow_persistent = On
```

This specifies the maximum amount of links, which are essentially identifiers for mysql connections. `-1` or `0` means unlimited links:
```
mysqli.max_links = -1
```

This specifes the size of the `mysqli` cache, which is used to store queries if specified:
```
mysqli.cache_size = 2000
```

This specifies the default port mysqli will try to connect on, unless otherwise specified in the php code with `mysqli_connect()`:
```
mysqli.default_port = 3306
```

This specifes the default socket will` mysqli` will use, if nothing is specified then it will uses the built-in mysql defaults:
```
mysqli.default_socket =
```

This specifies the default host which `mysqli` will use when it isn't specified in the code with `mysqli_connect()`:
```
mysqli.default_host =
```

This specifies the default user which `mysqli` will use when it isn't specified in the code with `mysqli_connect()`:
```
mysqli.default_user =
```

This specifies the default password which `mysqli` will use when it isn't specified in the code with `mysqli_connect()`:
```
mysqli.default_pw =
```

This option specifies if `mysqli` will automatically try to reconnect if the connection is lost:
```
mysqli.reconnect = Off
```

#### mysqlnd configs php.ini

mysqlnd is a php library which will allow for predefined code for connecting, and interfacing with MySQL databases. This section will start with this tag:

```
[mysqlnd]
```

`mysqlnd.collect_statistics` specifies if statistics can be gathered about the performance of mysql, can be viewed with the php commands `mysqli_get_client_stats()`, ` mysqli_get_connection_stats()`, `mysqli_get_cache_stats()`, and in the `mysqlnd` section of `phpinfo()`:
```
mysqlnd.collect_statistics = On
```

`mysqlnd.collect_statistics` specifies if statistics can be gathered about the memory usage of mysql, can be viewed with the php commands `mysqli_get_client_stats()`, ` mysqli_get_connection_stats()`, `mysqli_get_cache_stats()`, and in the `mysqlnd` section of `phpinfo()`:
```
mysqlnd.collect_memory_statistics = Off
```

`mysqlnd.debug =` records all communication from all php extensions using `mysqlnd` to the log file specified after `=`:
```
;mysqlnd.debug =
```

This specifies which queries are logged, default value is `0` which disables it, this works kind of like linux permissions where different values have different numbers. `2048` will log everything. The logs should be stored in apache logs, if it is being used as an apache mod:
```
mysqlnd.log_mask = 2048
```

`mysqlnd.mempool_default_size` specifies the default memory pool for `mysqlnd` to use: 
```
;mysqlnd.mempool_default_size = 16000
```

`mysqlnd.net_cmd_buffer_size` establishes the amount of space in bytes which is allocated for every connection:
```
;mysqlnd.net_cmd_buffer_size = 2048
```

`mysqlnd.net_read_buffer_size = 32768` specifies the maximum amount of space in bytes when reading  a mysql command packet (probably max size of packet while talking to mysql)
```
;mysqlnd.net_read_buffer_size = 32768
```

`mysqlnd.net_read_timeout` specifies the timeout in seconds for a mysql connection, if this isn't set than the default php timeout applies:
```
;mysqlnd.net_read_timeout = 31536000
```

`mysqlnd.sha256_server_public_key` specifies the file which holds the MySQL public RSA key (if it is using it). If this isn't specified either here or with the `mysqli_options()` command, it will need to be exchanged durring the SHA-256 Authentication Plugin authentication procedure.
```
;mysqlnd.sha256_server_public_key =
```

#### [SQL] php.ini

These settings will be below the following tab:
```
[SQL]
```

The following setting if turned On, will enable sql safe mode which overwrite all user defined values with default values
```
sql.safe_mode = Off
```

#### [mail function]

To specify the port for smtp:
```
smtp_port = 25
```

To specify the path to the sendmail binary:
```
sendmail_path
```

To specify arguments to be passed to the sendmail bindary:
```
mail.force_extra_parameters =
```

to specify to add header which includes the uid of the script, fllowed by the filename:
```
mail.add_x_header = On
```

to specify the log for mail:
```
mail.log = syslog
```

## Security
Here is a breif look at what you can do to lockdown php:

*	Blacklist php functions
*	Remote FIle Inclusion
*	Disable File Uploads
*	Set Base Dir
*	Don't expose PHP
*	Limit Post Requests Size
*	Check MySQLi Confs
*	Check resources (memory usage)
*	Configure Logging


#### Blacklist php functions

Certain php functions such as `exec`. Here is a list of php functions you should blacklist:
```
disable_functions = exec, passthru, shell_exec, system, proc_open, popen, curl, exec, curl_multi_exec, parse_ini_file, show_source, pcntrl_exec, eval, assert, preg_replace, create_function, include, include_one, require, require_once, phpinfo,  pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
```
#### Remote File Inclusion

You should disable `allow_url_fopen` and `allow_url_include`, since they can be used by an attacker to access files on the server, which they can use in exploits (Remote File Inclusion):

```
allow_url_fopen = Off
allow_url_include = Off
```

#### Blacklist File Uploads

To disable file uploads (which will disable an attacker from uploading things such as web shell) add/edit in these settings:

```
file_uploads = Off
upload_max_filesize = 0
max_file_uploads = 0
```

#### Set Base Dir

You can limit the file operations of php to a defined directory, which can limit what an attacker can do:

```
open_basedir = /var/www/html
```

#### Don't expose PHP

PHP can expose that it is installed on a server, which will be potentially helpful to an attacker. To disable it add/edit in this conf:
```
expose_php = Off
```

#### Limit Post Requests Size

Depending on what you are doing, you should probably limit the size of post requests which PHP will take, since this will help prevent certain type of DOS attacks:

```
post_max_size = 512K
```

#### Check MySQL Confs

You should probably take a second and look over the conf files for `mysqli` (or whatever php mysql library you are using). Here are the important lines to check:

If these are set to valid credentials, then red just needs to call the `mysqli` connect function to pivot to the db:
```
mysqli.default_user =
```
```
mysqli.default_host =
```
```
mysqli.default_pw =
```

Should probably check the port/socket `mysqli` is using:
```
mysqli.default_port = 3306
```

```
mysqli.default_socket =
```

#### Check resource management

By default, a php script can consume up to 128 Megabytes, which for our needs is too much. This can be descreased to 10M, in order to help prevent DOS attacks:
```
memory_limit = 10M
```

#### Configure Logging

Make sure to enable logging with these commands

```
log_errors = On
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
```

also check to ensure that the logging location is acurate (should probably be commented out by default):
```
; error_log = syslog
```

## Logs

Since PHP in our case acts as an Apache plugin, the logs for it will appear in apache's logs. So errors should appear in `/var/log/apache2/error.log`

An example log for banned php function being called:
```
[Fri Mar 02 12:18:56.505245 2018] [:error] [pid 3506] [client 192.168.234.1:47880] PHP Warning:  phpinfo() has been disabled for security reasons in /var/www/html/php/index.php on line 2
```

An example log for when a php file outside of the base directory specified by `open_basedir = /var/www/html` is attempted to be ran:
```
#	cat /var/log/apache2/error.log
[Fri Mar 02 12:43:41.419122 2018] [:error] [pid 3927] [client 192.168.234.1:48252] PHP Warning:  Unknown: open_basedir restriction in effect. File(/var/www/html/php/index.php) is not within the allowed path(s): (/var/www/sh) in Unknown on line 0
[Fri Mar 02 12:43:41.419170 2018] [:error] [pid 3927] [client 192.168.234.1:48252] PHP Warning:  Unknown: failed to open stream: Operation not permitted in Unknown on line 0
[Fri Mar 02 12:43:41.419180 2018] [:error] [pid 3927] [client 192.168.234.1:48252] PHP Fatal error:  Unknown: Failed opening required '/var/www/html/php/index.php' (include_path='.:/usr/share/php') in Unknown on line 0
```

An example log a php script requesting more memory than it's allocated:
```
#   cat /var/log/apache2/error.log
[Fri Mar 02 13:34:05.074285 2018] [:error] [pid 6014] [client 192.168.234.1:49524] PHP Fatal error:  Allowed memory size of 2097152 bytes exhausted (tried to allocate 266240 bytes) in /var/www/html/classes/PrestaShopAutoload.php on line 285
```

You can view people accessing your php files through Apache's access logs:
```
#	cat /var/log/apache2/access.log
192.168.234.1 - - [02/Mar/2018:12:46:27 -0500] "GET /php/ HTTP/1.1" 200 297 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:57.0) Gecko/20100101 Firefox/57.0"
```

You can view infomration regarding how the server is interacting with php in the system logs:
```
#	cat /var/log/syslog
Mar  2 12:39:19 Silent-Horror systemd[1]: Starting Clean php session files...
Mar  2 12:39:19 Silent-Horror systemd[1]: Started Clean php session files.
```

## Migrate

You shouldn't have to migrate PHP. Just install it on the other server, and then edit the config files as needed.

## Extra

To get the current php version:
```
#	php -v
PHP 7.0.27-0+deb9u1 (cli) (built: Jan  5 2018 13:51:52) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
    with Zend OPcache v7.0.27-0+deb9u1, Copyright (c) 1999-2017, by Zend Technologies
```
To see a list of all php mods enabled:
```
php -m
[PHP Modules]
calendar
Core
ctype
date
exif
fileinfo
filter
ftp
gettext
hash
iconv
json
libxml
openssl
pcntl
pcre
PDO
Phar
posix
readline
Reflection
session
shmop
sockets
SPL
standard
sysvmsg
sysvsem
sysvshm
tokenizer
Zend OPcache
zlib

[Zend Modules]
Zend OPcache
```

How mods work in php is they are all stored in the directory `mods-available`. When they are enabled, a symbolic link is created to the mod and stored in the `conf.d` directory (the number is the filenames under `conf.d` are there to keep filenames unique):
```
#   ls /etc/php/7.0/mods-available/
calendar.ini  fileinfo.ini  iconv.ini	 pdo.ini    readline.ini  sysvmsg.ini  tokenizer.ini
ctype.ini     ftp.ini	    json.ini	 phar.ini   shmop.ini	  sysvsem.ini
exif.ini      gettext.ini   opcache.ini  posix.ini  sockets.ini   sysvshm.ini
ls /etc/php/7.0/apache2/conf.d/
10-opcache.ini	 20-exif.ini	  20-iconv.ini	20-readline.ini  20-sysvsem.ini
10-pdo.ini	 20-fileinfo.ini  20-json.ini	20-shmop.ini	 20-sysvshm.ini
20-calendar.ini  20-ftp.ini	  20-phar.ini	20-sockets.ini	 20-tokenizer.ini
20-ctype.ini	 20-gettext.ini   20-posix.ini	20-sysvmsg.ini
```

To enable a mod, you can use the `phpenmod` command (or just make a symbolic link yourself):
```
#   phpenmod readline
#   ln -s /etc/php/7.0/mods-available/calendar.ini /etc/php/7.0/apache2/conf.d/20-calendar.ini
```

To disable a mod, you can use the `phpdismod` command (or just delete the symbolic link yourself):
```
#   phpdismod readline
#   rm /etc/php/7.0/apache2/conf.d/20-calendar.ini
```
