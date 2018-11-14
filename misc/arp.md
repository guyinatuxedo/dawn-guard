# ARP Poisoning
ARP (Address Resolution Protocol) is the protocol thta maps MAC Addresses to an IP, so the computer knows which MAC address to send network data. ARP Poisoning is a Man In The Middle attack where the hax0r sends manufactured arp messages that are designed to change the MAC address of let's say the default gateway on the victim to that of the attacker, so all data will pass through them and they become the man in the middle.

## Security:

This is to set your current arp entries as static. Keep in mind that your arp entries are not persisten across reboots, so we will have to configure it to add them upon each starutp. First export your current arp entries to the rc.local file:
```
#	arp -a >> /etc/rc.local
```

Next you need to edit the file. You will need to know the network adapter that the arp entries will be for, then you can format it like this:
```
arp -i ens33 -s 172.16.66.2 00:50:56:f2:0e:b0
arp -i ens33 -s 172.16.66.254 00:50:56:fe:f6:e8
exit 0
```

Next you will need to make the rc.local file executable by root:

```
$	sudo chmod 770 /etc/rc.local
```

Finally you should run the script, then check the arp table to confirm that it worked:
```
#	/etc/rc.local
#	arp -a
```

## Commands:

To view your arp table:
```
$	arp -a
```

To drop the arp entry for the ip address at 172.16.66.136:
```
$	sudo arp -d 172.16.66.131
```

To statically set an arp entry:
```
arp -i ens33 -s 172.16.66.2 00:50:56:f2:0e:b0
```

To view your own MAC address:
```
$	ifconfig -a | grep HWaddr
```

If you need to clear all of the ARP entries, i recogmend rebooting.


## Logging:

You will need to install arpwatch:
```
$	sudo apt-get install arpwatch
```

Next you will need to enable it for the interface you want:
```
$	arpwatch -i ens33
```

With that you will now see arp records appear in "/var/log/syslog". 

This log signifies a new ARP entry:
```
Jun 5 07:13:27	ubuntu arpwatch: new station 172.16.66.1 00:0c:21:81:97 ens33
```

This is what an arp poisoning attack looks like:
```
Attacker's IP: 172.16.66.129
Attacker's Mac Address: 00:0c:29:21:81:97
Gateay Address: 172.16.66.2
Gateway Mac Address: 00:50:56:fe:f6:e8
```

```
Jun 5 07:59:09 ubuntu arpwatch: flip flop 172.16.66.2 00:50:56:fe:f6:e8 (00:0c:29:21:81:97) ens33
Jun 5 07:59:09 ubuntu arpwatch: flip flop 172.16.66.2 00:0c:29:21:81:97 (00:50:56:fe:f6:e8) ens33
```

```
Jun 5 08:49:09 ubuntu arpwatch: flip flop 172.16.66.2 00:0c:29:21:87:97 (00:50:56:fe:f6:e8) ens33
Jun 5 08:49:09 ubuntu arpwatch: flip flop 172.16.66.2 00:50:56:fe:f6:e8 (00:0c:29:21:87:97) ens33
```

 
