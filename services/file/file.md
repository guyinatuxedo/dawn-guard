# Fedora 20 Vsftpd & Samba

## Vsftpd Installation

#### Install

First install `vsftpd`:
```
$	sudo yum -y install vsftpd
```

Next edit  the main vsftpd config file:
```
$	sudo vi /etc/vsftpd/vsftpd.conf
```

add/edit in the following settings to allow local users:
```
anonymous_enable=NO
write_enable=YES
chroot_local_user=YES
listen=YES
listen_ipv6=NO
```

next you will need to set the sebool to allow for ftp users to do things:

```
$	 setsebool -P allow_ftpd_full_access 1
```

In order to do egress GID filtering, you will need to add the users which will be logging in to a common group:

```
#	groupadd ftpusers
#	usermod -aG ftpusers guy
```

next just restart/enable `vsftpd`:

```
$	sudo systemctl restart vsftpd
$	sudo systemctl enable vsftpd
```

#### Add Users

This is the process of adding an ssh user:

```
#	useradd guy
#	usermod -s /sbin/nologin guy
#	passwd guy
#	chmod a-w /home/guy
```

might be a good idea to restart `vsftpd`:

```
#	systemctl restart vsftpd
```

## Samba Setup

First install samba:

```
#	yum install samba
```

Then edit the main smb conf file:

```
#	vi /etc/samba/smb.conf
```

append these settings to the end to add the fileshare `fighters` located at `/dbz/` in the file directory:

```
[fighters]
path = /dbz/
valid users = picolo gohan
read only = no
```

add the users which we will use for smb:

```
#	useradd picolo
#	useradd gohan
#	passwd picolo
#	passwd gohan
```

assign those users an smb password:

```
#	smbpasswd -a picolo
New SMB password:
Retype new SMB password:
Added user picolo.
#	smbpasswd -a gohan
New SMB password:
Retype new SMB password:
Added user gohan.
```

test the paramaters to verify no typos:

```
#	testparm
Load smb config files from /etc/samba/smb.conf
rlimit_max: increasing rlimit_max (1024) to minimum Windows limit (16384)
Processing section "[homes]"
Processing section "[printers]"
Processing section "[fighters]"
Loaded services file OK.
Server role: ROLE_STANDALONE
Press enter to see a dump of your service definitions

[global]
	workgroup = MYGROUP
	server string = Samba Server Version %v
	log file = /var/log/samba/log.%m
	max log size = 50
	idmap config * : backend = tdb
	cups options = raw

[homes]
	comment = Home Directories
	read only = No
	browseable = No

[printers]
	comment = All Printers
	path = /var/spool/samba
	printable = Yes
	print ok = Yes
	browseable = No

[fighters]
	path = /dbz/
	valid users = picolo, gohan
	read only = No
```

next enable this sebool to use smb:

```
#	setsebool -P samba_export_all_ro=1
```

Proceeding that add the smb users to a group, assign that group permissions to the fileshare:

```
#	mkdir /dbz
#	groupadd fighters
#	usermod -aG fighters picolo
#	usermod -aG fighters gohan
#	chown root:fighters /dbz/
#	chmod 774 -R /dbz/
```

lastly start/enable the smb service:

```
#	systemctl restart smb
#	systemctl enable smb
```
## Vsftpd Client

Here is an example of logging into an ftp server, and downloading a file:

```
$	ftp 192.168.234.196
Connected to 192.168.234.196.
220 (vsFTPd 3.0.2)
Name (192.168.234.196:guyinatuxedo): guy
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0              13 Apr 08 19:42 tux
226 Directory send OK.
ftp> get tux
local: tux remote: tux
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for tux (13 bytes).
226 Transfer complete.
13 bytes received in 0.00 secs (239.5342 kB/s)
ftp> exit
221 Goodbye.
$	cat tux 
guyinatuxedo
```

## Smb Client

First install an smbclient:
```
$	sudo apt-get install smbclient
```

Here is an example of logging into an smclient and downloading a file:

```
$	smbclient //192.168.234.196/fighters -U picolo
WARNING: The "syslog" option is deprecated
Enter picolo's password: 
Domain=[MYGROUP] OS=[Unix] Server=[Samba 4.1.17]
smb: \> ls
  .                                   D        0  Sun Apr  8 16:00:41 2018
  ..                                  D        0  Sun Apr  8 15:48:07 2018
  final_flash                         N       12  Sun Apr  8 16:00:41 2018
  hi                                  N        0  Sun Apr  8 15:55:11 2018

		18141508 blocks of size 1024. 11820504 blocks available
smb: \> get final_flash
getting file \final_flash of size 12 as final_flash (3.9 KiloBytes/sec) (average 3.9 KiloBytes/sec)
smb: \> exit
$	cat final_flash 
rip Kakarot
```
