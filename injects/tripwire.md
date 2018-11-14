# Tripwire IDS Ubuntu 16.04

## Installation:

First install tripwire. Durring the setup process, you will be asked if you want to make some passphrases for tripwire. Do it:

```
$	sudo apt-get install tripwire
```

Next you will need to initiate the database. You should see errors with this next command about certian directories missing, but you can ignore it as long as it ends with saying that the database was successfully generated.

```
$	sudo tripwire --init
```

Now Tripwire is installed and running. Thing is Tripwire probably has some files it is looking for that aren't on your system. I went through and made a script that will comment out all of the files for us. To start, we will need a list of all of the files we are missing:

```
#	tripwire --check | grep Filename > /etc/tripwire/fp
```

Now that we have that, we can make the script. Just place this script in /etc/tripwire and run it:

```
#	pwd
/etc/tripwire
#	cat clean_policy.py 
with open('tw.txt', 'r') as file:
	data = file.readlines()
c = -1

with open('fp') as f:
	for l in f:
		p = l.replace("     Filename: ", "")
		p = p.replace("\n", "")
		print "Searching for: " + p
		c = -1

		for i in data:
			c = c + 1
			if p in i:
				print "Commenting out: " + i
				data[c] = '#' + data[c]
				with open('tw.txt', 'w') as file:
					file.writelines(data)
```

Next we can run the script to comment out all of the files which Tripwire is checking for, however don't exist on our system (make sure this script is in the same directory as the `tw.txt` file, which should be in `/etc/tripwire`) This script may not comment out some files located in `/etc/proc`.:

```
#	python clean_policy.py 
Searching for: /usr/sbin/fixrmtab
Commenting out:      /usr/sbin/fixrmtab                -> $(SEC_BIN) ;

Searching for: /usr/share/grub/i386-redhat/e2fs_stage1_5
Commenting out:      /usr/share/grub/i386-redhat/e2fs_stage1_5      -> $(SEC_CRIT) ;

Searching for: /usr/share/grub/i386-redhat/fat_stage1_5
Commenting out:      /usr/share/grub/i386-redhat/fat_stage1_5       -> $(SEC_CRIT) ;

...
```

after that we need to specify that /etc/tripwire/tw.txt is the new policy file:

```
$	sudo twadmin -m P /etc/tripwire/tw.txt
Please enter your site passphrase: 
Wrote policy file: /etc/tripwire/tw.pol
```

Then we can reinitialize the database. This time you should see no missing file errors:

```
#	tripwire --init
```

Now when you check on tripwire, there should be no missing file errors (except for some files in `/etc/tripwire`):

```
#	tripwire --check
```

## Use:

To use Tripwire to see if any of the files it monitors has been changed, just run a check:

```
$	sudo tripwire --check
```

## Config

If you want to have Tripwire monitor a directory or file of your choosing, first edit your tripwire policy config file:
```
#	vim /etc/tripwire/tw.txt 
```

then just append this to the end, if you want to monitor the file located at `/etc/planets`, and the directory `/etc/heretic/`:
```
(
 rulename = "Custom Checks",
 severity = $(SIG_HI)
)

{
        /etc/planets            -> $(SEC_CRIT);
        /etc/heretic/           -> $(SEC_CRIT);
}
```

After you make any config changes, make sure you update the policy file and database for it to take effect:

```
#	twadmin -m P /etc/tripwire/tw.txt 
Please enter your site passphrase: 
#	tripwire --init
Please enter your local passphrase: 
Parsing policy file: /etc/tripwire/tw.pol
Generating the database...
*** Processing Unix File System ***
...
Wrote database file: /var/lib/tripwire/localhost.localdomain.twd
The database was successfully generated.
```

## Automation 
If you want to automate Tripwire reports, you can use crontabs for that.

```
#	vim /etc/crontab
```

Then you can just add the command like it's a nomral cronjob. Here is the setting to run a check every minute, and output it to /tmp/report:

```
* * * * * root tripwire --check > /tmp/report
```
# Tripwire IDS CentOS 7

## Installation:

First install tripwire:

```
$	sudo yum install epel-release 
$	sudo yum install tripwire
```

Next you will want to setup keys, which are basically the passwords for tripwire:

```
$	sudo tripwire-setup-keyfiles
```

Next you will need to initiate the database. You should see errors with this next command about certian directories missing, but you can ignore it as long as it ends with saying that the database was successfully generated.

```
$	sudo tripwire --init
```

Now Tripwire is installed and running. Thing is Tripwire probably has some files it is looking for that aren't on your system. If it is like mine, it is a couple hundred. I went through and made a script that will comment out all of the files for us. To start, we will need a list of all of the files we are missing:

```
#	tripwire --check | grep Filename > /etc/tripwire/fp
```

Now that we have that, we can make the script. Just place this script in /etc/tripwire and run it:

```
#	pwd
/etc/tripwire
#	cat clean_policy.py 
with open('twpol.txt', 'r') as file:
	data = file.readlines()
c = -1

with open('fp') as f:
	for l in f:
		p = l.replace("     Filename: ", "")
		p = p.replace("\n", "")
		print "Searching for: " + p
		c = -1

		for i in data:
			c = c + 1
			if p in i:
				print "Commenting out: " + i
				data[c] = '#' + data[c]
				with open('twpol.txt', 'w') as file:
					file.writelines(data)

```

Next we can run the script to comment out all of the files which Tripwire is checking for, however don't exist on our system (make sure this script is in the same directory as the `twpol.txt` file, which should be in `/etc/tripwire`):

```
#	python clean_policy.py 
Searching for: /usr/sbin/fixrmtab
Commenting out:      /usr/sbin/fixrmtab                -> $(SEC_BIN) ;

Searching for: /usr/share/grub/i386-redhat/e2fs_stage1_5
Commenting out:      /usr/share/grub/i386-redhat/e2fs_stage1_5      -> $(SEC_CRIT) ;

Searching for: /usr/share/grub/i386-redhat/fat_stage1_5
Commenting out:      /usr/share/grub/i386-redhat/fat_stage1_5       -> $(SEC_CRIT) ;

...
```

after that we need to specify that /etc/tripwire/twpol.txt is the new policy file:

```
$	sudo twadmin -m P /etc/tripwire/tw.txt
Please enter your site passphrase: 
Wrote policy file: /etc/tripwire/tw.pol
```

Then we can reinitialize the database. This time you should see no missing file errors:

```
#	tripwire --init
```

Now when you check on tripwire, there should be no missing file errors:

```
#	tripwire --check
```

## Use:

To use Tripwire to see if any of the files it monitors has been changed, just run a check:

```
$	sudo tripwire --check
```

## Config

If you want to have Tripwire monitor a directory or file of your choosing, first edit your tripwire policy config file:
```
#	vim /etc/tripwire/twpol.txt 
```

then just append this to the end, if you want to monitor the file located at `/etc/planets`, and the directory `/etc/heretic/`:
```
(
 rulename = "Custom Checks",
 severity = $(SIG_HI)
)

{
        /etc/planets            -> $(SEC_CRIT);
        /etc/heretic/           -> $(SEC_CRIT);
}
```

After you make any config changes, make sure you update the policy file and database for it to take effect:

```
#	twadmin -m P /etc/tripwire/twpol.txt 
Please enter your site passphrase: 
#	tripwire --init
Please enter your local passphrase: 
Parsing policy file: /etc/tripwire/tw.pol
Generating the database...
*** Processing Unix File System ***
Wrote database file: /var/lib/tripwire/localhost.localdomain.twd
The database was successfully generated.
```

## Automation 
If you want to automate Tripwire reports, you can use crontabs for that.

```
#	vim /etc/crontab
```

Then you can just add the command like it's a nomral cronjob. Here is the setting to run a check every minute, and output it to /tmp/report:

```
* * * * * root tripwire --check > /tmp/report
```

## Writeups 
https://komunity.komand.com/learn/article/host-intrusion-detection/how-to-install-and-configure-tripwire-ids-on-centos-7/
