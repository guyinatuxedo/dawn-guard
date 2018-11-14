# Call of Time Ubuntu 8.04 MySQL/Joomla

Essentially this is a linux defense strategy which is comprised of making a lot of the file system immutable (unable to be written to), watching it with the IDS Tripwire, and then having a backup from which you can replace things. 

```
/bin/
/boot/
/dev/
/etc/
/home/
/lib/
/media/
/mnt/
/opt/
/root/
/sbin/
/srv/
/sys/
/tmp/
/usr/
/var/
```

## Tripwire

If you have configured the filesystem as immutable, you might want to remove the immutable bit from everything just for the installation:
```
$	sudo chattr -R -i / 
```

First install tripwire:
```
$	sudo apt-get install tripwire
```

Next you are going to want to create the policy text file:

```
$	sudo vim /etc/tripwire/cot.txt
```

here is a policy file which will essentially monitor all of the directories listed recurively for files/directories whichh have been added/removed/modified:

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

next you can apply the changes of the new config file:

```
$	sudo twadmin -m P /etc/tripwire/cot.txt 
Please enter your site passphrase: 
Wrote policy file: /etc/tripwire/tw.pol
```

proceeding that you will need to initialize the database with the new policy, to essentially take a hash of everything you want to monitor to compare to later:

```
$	sudo tripwire --init 
```

proceeding that you can run a check, and then parse through the output:
```
#	tripwire --check > /var/log/cot
#	vim /var/log/cot
```
#### Extra

After you reboot, you will probably get thousands of modified files found when you check, so you will need to reinitialzie the database to remove those found entries.

## Chattr

To make the shadow file immutable:
```
$	sudo chattr +i /etc/shadow
```

to make the shadow file writeable again:
```
$	sudo chattr -i /etc/shadow
```

to make all diretories and files under `/usr/` immutable:
```
$	sudo chattr -R +i /usr/
```

#### Chattr Directories

Here are some directories which you can chattr:

```
$	sudo chattr -R +i /bin/
$	sudo chattr -R +i /etc/
$	sudo chattr -R +i /home/
$	sudo chattr -R +i /root/
$	sudo chattr -R +i /tmp/
$	sudo chattr -R +i /usr/
```
```
$	sudo chattr -R +i /opt/
$	sudo chattr -R +i /srv/
```
```
#	chattr -R +i /home/
#	chattr -R -i /home/guyinatuxedo/
```



```
$	sudo chattr +i -R /var/backups/
$	sudo chattr +i -R /var/cache/
$	sudo chattr +i -R /var/crash/
$	sudo chattr +i -R /var/games/
$	sudo chattr +i -R /var/mail/
$	sudo chattr +i -R /var/opt/
$	sudo chattr +i -R /var/spool/
$	sudo chattr +i -R /var/tmp/
$	sudo chattr +i -R /var/www
```

if you encounter this error with tripwire:
```
### Error: File could not be opened.
### Filename: /var/lib/tripwire/guyinatuxedo-desktop.twd
### Filename: twtempXXXXXX
### Solution: Check existence/permissions for directory specified by
### TEMPDIRECTORY in config file
### Exiting...
```

it can be solved by allowing the home directory of `guyinatuxedo` to be writeable:
```
$	sudo chattr -R -i /home/guyinatuxedo/
```

## Backup

make the backup directories:
```
#	mkidr /backup
#	mkdir /backup/bin
#	mkidr /backup/etc
#	mkidr /backup/home/
#	mkdir /backup/root
#	mkdir /backup/usr 
#	mkdir /backup/var
```

copy the directories which you wish to back up:
```
#	cp -avr /bin/ /backup/bin/
#	cp -avr /etc/ /backup/etc/
#	cp -avr /home/ /backup/home/
#	cp -avr /root/ /backup/root/
#	cp -avr /usr/ /backup/usr/
#	cp -avr /var/ /backup/var/
```

also you may want to add the backup directory to your tripwire policy and chattr it.
