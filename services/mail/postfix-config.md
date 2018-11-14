# Postfix Config

## Files/Directories

The postfix config directory is here:
```
#	cd /etc/postfix/
#	ls
dynamicmaps.cf  main.cf.proto  master.cf.proto  postfix-script  sasl
main.cf         master.cf      postfix-files    post-install
```

The file `main.cf.proto` provides a lot of documentation and sample config for the `main.cf` file (main config file for postfix):

### Settings

All of these settings are found in `/etc/postfix/main.cf`

This setting will set the compatibillity mode, which is used for backwards-compatibillity after an upgrade:

```
compatibility_level = 2
```

The `soft_bounce` option is used for debugging, and will have mail remained queued that would otherwise be bounced (mail that is returned to sender, since it can't be sent):

```
soft_bounxe = no
```

To specify the location of the postfix queue
```
queue_directory = /var/spool/postfix
```

To specify the directory which will store the postfix commands (binaries whcih will run postfix):

```
command_directory = /usr/sbin
```

This will specify the directory which will store all of the daemon binaries:
```
daemon_directory = /usr/lib/postfix/sbin
```

To specify the directory which will store the location of data files which Postfix can write to:
```
data_directory = /var/lib/postfix
```

To specify the user which will own the Postfix queue, and most of the postfix daemon processes will run as:
```
#mail_owner = postfix
```

```
#default_privs = nobody
```

This setting will determine the internet hostname of this mail system:

```
#myhostname = host.domain.tld
```

This setting will determine the internet domain name, which b default uses `$myhostname` minus the host name componet:

```
#mydomain = domain.tld
```

This option specifies the domain that locally-posted email appears to come from:
```
$myorigin = $myhostname
```

To specify the network addresses which this mail system will receive mail on ( to speciiy all adapaters, set it equal to `all`):
```
#inet_interfaces = $myhostname, localhost
```

To specify the network addresses that this mail server will use as proxies:
```
#proxy_interfaces =
```

The `mydestination` option specifies what domains this machine will deliver to, instead of forwarding to another machine:

```
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
```

This option specifies the lookup tables for local users or addresses (addresses that match $mydestination). If a user does not match it and is determined to be a local account, the mail is rejected:

```
#local_recipient_maps = unix:passwd.byname $alias_maps
```

To specify the SMTP server response for when a local user is rejected (using the setting above this):

```
unknown_local_recipient_reject_code = 550
```

To specify the IP blocks which will have trusted SMTP clients, which means that they are allowed to relay mail through Postfix (have it pass mail through the server):

```
#mynetworks = 168.100.189.0/28, 127.0.0.0/8
```

This option specifies what this mail server will forward mail to for non-local mail:

```
#relay_domains = $mydestination
```

This option specifies a lookup table for users that want to relay mail through this server. If it is sent and a user that is trying to relay mail through here isn't found on the map, it will be rejected:

```
#relay_recipient_maps = hash:/etc/postfix/relay_recipients
```

This option specifies how long postfix will wait after receiving an email, before receiving a new email (this option is disabled on some versions due to a bug):

```
#in_flow_delay = 1s
```

The `alias_maps` option specifies the alis databases, which stores aliases for redirecting mail for local recipients:

```
#alias_maps = dbm:/etc/aliases
```

This specifies the alias databases for local delivery that are updated with `newaliases` or `sendmail -bi`

To specify the string which will separate user names and addresses:

```
#recipient_delimiter = +
```

This option specifies the location of the mailbox file, relative to that user's home directory:

```
#home_mailbox = Maildir/
```

The `mail_spool_directory` specifies the mail directory for Unix style mailbpxes:

```
#mail_spool_directory = /var/mail
```

This option specifies the command that is ran for mailbox delivery:

```
#mailbox_command = /usr/bin/procmail -a "$EXTENSION"
```

This specifies the socket which will be used for mailbox delivery to all local recipients:

```
#mailbox_transport = lmtp:unix:/var/imap/socket/lmtp
```

This specifies the transport which will be used as a fallback for recipients that are not found in the UNIX passwd database:

```
#fallback_transport = lmtp:unix:/file/name
```

This setting specifies an optional destination address for unknown recipients:
```
#luser_relay = $user@other.host
```

This options specifies an optional table, which if a message header mataches a pattern in the file, the corresponding action is executed:

```
header_checks = regexp:/etc/postfix/header_checks
```

To specify domains that are eligible for the Fast Etrn Service, which essentially allows for certian mail in the queue to be immediately sent if requested using the customers mail server:

```
#fast_flush_domains = $relay_domains
```

This option specifies the banner which is displayed in the SMTP greeting, received headers, and in bounced mail:

```
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
```

This option specifies the maximum number of parallel deliveries via the local mail transport using the same recipient:

```
#local_destination_concurrency_limit = 2
```

This option specifies the default value for the maximum number of parallel deliveries via the local mail transport using the same recipient:

```
#default_destination_concurrency_limit = 20
```

This specifies the level of verbose logging when an SMTP client or server host name or address matches a pattern in the `debug_list_paramter`:

```
#debug_peer_level = 2
```

This option specifies a domain or network address, so when a SMTP client or server host name or address matches this, the log levels are elevated to the level specified by `debug_peer_level`:

```
#debug_peer_list = 127.0.0.1
```

This option specifies the command which is run when a postfix program is run with the `-D` option:

```
debugger_command =
         PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
         ddd $daemon_directory/$process_name $process_id & sleep 5
```

and you can run it with gdb like this:

```
# debugger_command =
#       PATH=/bin:/usr/bin:/sbin:/usr/sbin; export PATH; screen
#       -dmS $process_name gdb $daemon_directory/$process_name
#       $process_id & sleep 1
```

This is a Sendmail compatibility feature that specifies the location of the Postfix `sendmail` (a different SMTP email) command:

```
sendmail_path =
```

Another Sendmail compativility feature that specifies the comand for `newaliases`, which can be used to rebuild to the local aliases database:

```
newaliases_path =
```

A sendmail compatibility feature that specifies where the Postfix `mailq` command is, which can be used to view the Postfix mail queue:

```
mailq_path =
```

This sets the group ownership of Postfix commands and group-writeable Postfix directories:

```
setgid_group =
```

The location of Postfix HTML files that describe how to build, configure or operate specific Posftix subsytems or features:

```
html_directory =
```

This is the direcotry where the manpages for Postfix are installed:

```
manpage_directory =
```

This option specifies the directory with example Postfix configuration files:

```
sample_directory =
```

This is the location of Postfix readme files, which provides information on various aspects of Postfix:

```
readme_directory =
```

This option specifies the address type (`ipv4`, `ipv6`, or `any`)
 that the SMTP client will try to send mail to first, when there is an MX record for both `ipv4` and `ipv6` to the mail server it is sending mail to:
```
inet_protocols = ipv4
```
