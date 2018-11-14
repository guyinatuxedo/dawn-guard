# Samba FIle Server Inject

This is a guide on how to do the samba file server inject on Ubuntu 17.04.

## Setup

First install samba:
```
$	sudo apt-get install samba
```

Edit the main samba config file:
```
$	sudo vim /etc/samba/smb.conf
```

append the following settings to the end to create the file share titled `visions` at the file path `/visons/`, which the users `doom` and `end` have access to:
```
[visions]
path = /visions/
valid users = doom end
read only = no
```

next you need to create the users:
```
$	sudo adduser doom
$	sudo adduser end
```

next you need to create the password which samba will use for the two users `doom` & `end`:
```
$	sudo smbpasswd -a doom
$	sudo smbpasswd -a end
```

next we can test the config with `testparm`:
```
$	testparm
Load smb config files from /etc/samba/smb.conf
rlimit_max: increasing rlimit_max (1024) to minimum Windows limit (16384)
WARNING: The "syslog" option is deprecated
Processing section "[printers]"
Processing section "[print$]"
Processing section "[visions]"
Loaded services file OK.
Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions
```

next we need to assign the permissions to the two users to the directory (and create it), which we will do by adding them to a group and assigning that group permissions to the directory:
```
$	sudo groupadd visions
$	usermod -aG visions end
$	usermod -aG visions doom
$	sudo mkdir /visions
$	sudo chown root:visions /visions/
$	sudo chmod 774 -R /visions/
```

after that you can start and enable the service:
```
$	sudo systemctl start smbd
$	sudo systemctl enable smbd
Synchronizing state of smbd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable smbd
```

## Client Usage

After that you can use the smb client to connect to the file server at `192.168.234.189` with the user `end`:
```
$	sudo apt-get install smbclient
$	smbclient //192.168.234.189/visions -U end
```

If you want to list all of the shares (NETBIOS is blocked by the firewall, so that is why the last part fails):
```
$	smbclient -L //192.168.234.189/visions -U end
WARNING: The "syslog" option is deprecated
Enter end's password: 
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.6.7-Ubuntu]

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	visions         Disk      
	IPC$            IPC       IPC Service (ubuntu server (Samba, Ubuntu))
Connection to 192.168.234.189 failed (Error NT_STATUS_IO_TIMEOUT)
NetBIOS over TCP disabled -- no workgroup available
```

If you want to make the directory `end` through the smbclient:
```
smb: \> md end
NT_STATUS_ACCESS_DENIED making remote directory \end
```

If you want to delete the directory `end` through the smbclient:
```
smb: \> rmdir end
```

If you want to upload the flie `screaming` through the smbclient:
```
$	ls | grep screaming 
screaming
$	smbclient //192.168.234.189/visions -U end
WARNING: The "syslog" option is deprecated
Enter end's password: 
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.6.7-Ubuntu]
smb: \> put screaming
```

If you want to download the file `villany`through the smbclient:
```
smb: \> ls
  .                                   D        0  Sun Apr  1 16:08:03 2018
  ..                                  D        0  Sun Apr  1 15:48:41 2018
  screaming                           A       10  Sun Apr  1 16:08:03 2018
  villany                             N        9  Sun Apr  1 16:07:16 2018

		20509264 blocks of size 1024. 16878140 blocks available
smb: \> get villany
getting file \villany of size 9 as villany (2.9 KiloBytes/sec) (average 2.9 KiloBytes/sec)
```

If you want to exit the smbclient:
```
smb: \> exit
```

## Iptables

Here are the iptables rules to allow ingres/egress whitelist filtering for SSH (port `25`) and SMB (port `445`):

```
$	sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
$	sudo iptables -A INPUT -p tcp --dport 445 -j ACCEPT
$	sudo iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
$	sudo iptables -A OUTPUT -p tcp --sport 445 -j ACCEPT
$	sudo iptables -P INPUT DROP
$	sudo iptables -P OUTPUT DROP
$	sudo iptables -P FORWARD DROP
$	sudo ip6tables -P INPUT DROP
$	sudo ip6tables -P OUTPUT DROP
$	sudo ip6tables -P FORWARD DROP
```

## Source
https://help.ubuntu.com/community/How%20to%20Create%20a%20Network%20Share%20Via%20Samba%20Via%20CLI%20%28Command-line%20interface/Linux%20Terminal%29%20-%20Uncomplicated,%20Simple%20and%20Brief%20Way!
