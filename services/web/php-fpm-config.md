## Config

PHP-FPM is essentially split amongst three different files by default with this configuation:

*	`/ect/php.ini`:	PHP Processor
*	`/etc/php-fpm.conf`:
*	`/etc/php-fpm.d/www.comf`

### /etc/php-fpm.conf

This line will include all files from `/etc/php-fpm.d/` that ends in `.conf` as config files (so it is like they are in this file):
```
include=/etc/php-fpm.d/*.conf
```

##### Global Options

To specify the file which holds the process id which php-fpm will run as:
```
pid = /run/php-fpm/php-fpm.pid
```

To specify the error log file, if it is set to `syslog` then it will just be sent to syslogd:
```
error_log = /var/log/php-fpm/error.log
```

To specify to syslog what type of program this is:
```
syslog.facility = daemon
```

To specify a message to be prepended to every log message to syslog:
```
syslog.ident = php-fpm
```

To specify the amount of logs which will be stored, values include `alert`, `error`, `warning`, `notice`, `debug` (default is `notice`, most verbose is probably `debug`)
```
log_level = notice
```

This specifies the number of child processes which exit with a SIGSEV or SIGBUS within the time interval set by `emergency_restart_interval` wil cause PHP-FPM to restart. A value of 0 means this feature is disabled:
```
emergency_restart_threshold = 0
```

This specifies the time interval for if there are more child processes that end in SIGSEV or SIGBUS specified by `emergency_restart_threshold` will cause PHP-FPM to restartL
```
emergency_restart_interval = 0
```

This specifies the time limit in seconds for child processes to wait for a reaction on signals from master. A value of 0 means no limit:
```
process_control_timeout = 0
```

This specifies the maximum number of processes which FPM will fork. A value of `0` means no limit:
```
process.max = 128
```

This specifies the nice priority which will apply to the master (if set). Values range from `-19` (highest priority) to `20` (lowest priority):
```
process.priority = -19
```

This specifies whether or not to keep PHP-FPM in the backgroun:
```
daemonize = yes
```

If you want to set an open file descriptio limit for the master process (will see this setting in other sections):
```
rlimit_files = 1024
```

This specifies the core maximum size for the master process, which is essentially the memory which the process uses. A value of `0` means no limit(will see this setting in other sections):
```
rlimit_core = 0
```

This specifies which event mechanism php-fpm wll use for logging. The options are `select`, `poll`, and `epoll`:

```
events.mechanism = epoll
```

The following will specify if php-fpm is built with systemd integration, the interval in seconds between health report notifications to systemd:

```
systemd_interval = 10
```

### Child processes (pools)

By default (check with the conf file discussed above to confirm) but the conf files for the child processes are stored in the following directory. This is what actually listens on a port:
```
#	cd /etc/php-fpm.d/
```

the default conf file is this:
```
#	vim /etc/php-fpm.d/www.conf
```

##### www

The following settings are under the following string:
```
[www]
```

To specify the directory which files/directories for the pool (things such as logs, and chroot directories):
```
;prefix = /path/to/pools/$pool
```

To specify the user which the child proess will be running as. The user should have access to everything that `php-fpm` needs (most convinient way is to set it to the same user as the webserver):
```
user = apache
```

To specify the group which the child proess will be running as. The group should have access to everything that `php-fpm` needs (most convinient way is to set it to the same group as the webserver):
```
group = apache
```

To specify what `PHP-FPM` will listen on. To have it listen on a socket:
```
listen = /run/php-fpm/www.sock
```

to have it listen on a TCP socket with the ip address and port `192.168.234.161:8080`

```
listen = 192.168.234.161:8080
```

This specifies the backlog socket for php-fpm to use with listen (see `man 2 listen` for more details): 
```
;listen.backlog = 511
```

To specify the user, group, and permissions which the unix socket will have. By default it has the user/group are set to the running user:
```
;listen.owner = nobody
;listen.group = nobody
;listen.mode = 0660
```

to specify an access control list which specifies a list of which users'groups can listen. If this is set then `listen.owner` and `listen.owner` are ignored:
```
listen.acl_users = apache,nginx
;listen.acl_groups = 
```

To specify which of the clients are allowed to connect:
```
listen.allowed_clients = 127.0.0.1
```

To specify how the process manager will control the number of child processes. If it is `static` then it is a se tmaximum, if `dynamic` then the number of servers will be set depending on `pm.max_children`, `pm.start_servers`, `pm.min_spare_servers`, and `pm.max_spare_servers`. If it is set to `ondemand` then children will be forked on request, and controlled by `max_children`, `pm.process_idle_timeout`:

```
pm = dynamic
``` 

To specify the maximum number of children processes:
```
pm.max_children = 50
```

To specify the maximum number of child processes created on startup:
```
pm.start_servers = 5
```

To specify the minimum number of idel server processes:
```
pm.min_spare_servers = 5
```

To specify the maximum number of idle server processes:
```
pm.max_spare_servers = 35
```

To specify the number of seconds after which an idle process will be killed (only used on `ondemand`):
```
pm.process_idle_timeout = 10s;
```

To specify the number of requests each child process hould execute before respawning:
```
;pm.max_requests = 500
```

To view a status page for `php-fpm` for the website `http://www.foo.bar/`, pass the following query string to the website:
```
http://www.foo.bar/status?full
``` 

this will return output similar to this:
```
Example output:
************************
pid:                  31330
state:                Running
start time:           01/Jul/2011:17:53:49 +0200
start since:          63087
requests:             12808
request duration:     1250261
request method:       GET
request URI:          /test_mem.php?N=10000
content length:       0
user:                 -
script:               /home/fat/web/docs/php/test_mem.php
last request cpu:     0.00
last request memory:  0
```

To detmine the path which will provide the status page:
```
;pm.status_path = /status
```

This specifies the path which will provide a page which will report if the page is respond to pings:
```
;ping.path = /ping
```

This specifies the expected response to be `pong`, which is a reply to a ping request:
```
;ping.response = pong
```

To specify an access log file to be set:
```
access.log = log/$pool.access.log
```

To speicfy a log file for slowly executing scripts:
```
slowlog = /var/log/php-fpm/www-slow.log
```

This is the timeout specified where a slowly executing php script will be terminated and dumped to the `slowlog` file:

```
request_slowlog_timeout = 0
```

This specifies the timeout for a worker process to kill itself after serving a single request:
```
request_terminate_timeout = 0
```

To specify a chrrot to this directory. Essentially this makes it so the process can only access files/directories in the chroot, since it makes that directory the root directory for the process:
```
;chroot = 
```

This will have the process change it's directory to this when the process starts:

```
;chdir = /var/www
```

This setting is to enable redirecting worker stdout and stderr into the main error log:
```
;catch_workers_output = yes
```

This setting will determine if arbitrary environment variables will be cleared, so they cannot be reached by `php-fpm`:
```
;clear_env = no
```

To limit the extensions of scripts which php-fpm can parse and run:
```
;security.limit_extensions = .php .php3 .php4 .php5 .php7
```

To specify environment variables which will be passed to the process:
```
;env[HOSTNAME] = $HOSTNAME
;env[PATH] = /usr/local/bin:/usr/bin:/bin
;env[TMP] = /tmp
;env[TMPDIR] = /tmp
;env[TEMP] = /tmp
```

This specifies filepaths, commands, and values for things such as sendmail, error logs, and the memory limits:
```
;php_admin_value[sendmail_path] = /usr/sbin/sendmail -t -i -f www@my.domain.com
;php_flag[display_errors] = off
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
;php_admin_value[memory_limit] = 128M
```

These establish directories which php-fpm owns, and can use:
```
php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache
;php_value[opcache.file_cache]  = /var/lib/php/opcache
```

### /etc/php.d/ & /etc/php-zts.d

These direcotries contains php modules which can be used by `php` and `php-zts` (thread-safe php interpeter for Apache HTTP Serber). All of these files should by default:

```
#	ls /etc/php.d/
20-bz2.ini       20-curl.ini      20-ftp.ini      20-json.ini     20-tokenizer.ini
20-calendar.ini  20-exif.ini      20-gettext.ini  20-phar.ini
20-ctype.ini     20-fileinfo.ini  20-iconv.ini    20-sockets.ini
#	cat /etc/php.d/20-bz2.ini 
; Enable bz2 extension module
extension=bz2.so
#	ls /etc/php-zts.d/
20-bz2.ini       20-curl.ini      20-ftp.ini      20-json.ini     20-tokenizer.ini
20-calendar.ini  20-exif.ini      20-gettext.ini  20-phar.ini
20-ctype.ini     20-fileinfo.ini  20-iconv.ini    20-sockets.ini
#	cat /etc/php-zts.d/20-bz2.ini 
; Enable bz2 extension module
extension=bz2.so
```

You can find the actual location of the shared library files using `find`:
```
#	 find . -name "calendar.so"
./usr/lib64/php/modules/calendar.so
./usr/lib64/php-zts/modules/calendar.so
```

### php.ini

For this (which is the config file which essentially deals with the php code being executed) check the other php writeup (it has everything there):
