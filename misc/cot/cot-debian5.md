# Call of Time Debian 5 DNS/DB

## Tripwire

First install tripwire:

```
#	apt-get install tripwire
```

Next create the tripwire policy file:

```
#	vim /etc/tripwire/cot.txt
```

Should have the following rule:

```
( rulename = "Call of Time", severity = 100)

{
        /bin               -> $(ReadOnly) (recurse = true) ;
        /boot              -> $(ReadOnly) (recurse = true) ;
        /dev             -> $(ReadOnly) (recurse = true) ;
        /etc              -> $(ReadOnly) (recurse = true) ;
        /home               -> $(ReadOnly) (recurse = true) ;
        /lib               -> $(ReadOnly) (recurse = true) ;
        /media               -> $(ReadOnly) (recurse = true) ;
        /mnt               -> $(ReadOnly) (recurse = true) ;
        /opt               -> $(ReadOnly) (recurse = true) ;
        /root               -> $(ReadOnly) (recurse = true) ;
        /sbin               -> $(ReadOnly) (recurse = true) ;
        /srv               -> $(ReadOnly) (recurse = true) ;
        /tmp               -> $(ReadOnly) (recurse = true) ;
        /usr               -> $(ReadOnly) (recurse = true) ;
        /var               -> $(ReadOnly) (recurse = true) ;
}
```

next add the policy file to tripwire:
```
#	twadmin -m P /etc/tripwire/cot.txt
```

then initialize the database (hash all of the files it checks for later comparison):

```
#	tripwire --init
```

then just run the check (should take about 10 seconds):

```
#	tripwire --check > /var/log/cot
```

## Chattr

These are the main files which you will want to make immutable:

```
$	sudo chattr -R +i /bin/
$	sudo chattr -R +i /home/
$	sudo chattr -R +i /root/
$	sudo chattr -R +i /tmp/
$	sudo chattr -R +i /usr/
```
```
#	chattr -R +i /etc/
#	chattr -R -i /etc/networkConnection
```

You might want to unchattr this directory (user's home directory that installed tripwire) to prevent issues with tripwire:

```
$	sudo chattr -R -i /home/guyinatuxedo/
```

These directories really shouldn't hold anything, so you can chattr them too:

```
$	sudo chattr -R +i /opt/
$	sudo chattr -R +i /srv/
```
