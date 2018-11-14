# CoT CentOS 7 Mail

## Tripwire

If you have configured the filesystem as immutable, you might want to remove the immutable bit from everything just for the installation:

```
$	sudo chattr -R -i / 
```

First install tripwire

```
#	yum install tripwire
```

Next you will need to setup the keys for tripwire:

```
#	tripwire-setup-keyfiles
```

Proceeding that you will need to create the policy file:

```
#	sudo vi /etc/tripwire/cot.txt
```

add/edit in the following files:

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

Then declare the policy file:

```
#	twadmin -m P /etc/tripwire/cot.txt 
Please enter your site passphrase: 
Wrote policy file: /etc/tripwire/tw.pol
```

Initialize the file system (it might take a minute):

```
#	tripwire --init
```

proceeding that you can run a check with tripwire (will take like 30 seconds), then parse through the output with vi:

```
#	tripwire --check > /var/log/cot
#	vi /var/log/cot
```

## Chattr

These are the main directories which you will want to make immutable:

```
$	sudo chattr -R +i /bin/
$	sudo chattr -R +i /etc/
$	sudo chattr -R +i /root/
$	sudo chattr -R +i /tmp/
$	sudo chattr -R +i /usr/
```

These directories really shouldn't hold anything, so you can chattr them too:

```
$	sudo chattr -R +i /opt/
$	sudo chattr -R +i /srv/
```

I don't chattr `/home/` or `/var` because it is likely that mail is stored in either location, which by making those immutable could break things.
