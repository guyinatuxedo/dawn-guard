# htop, find, and usb

### htop:

To install htop:
```
$	sudo apt-get install htop
```

To use htop:
```
$	htop
```

* `u`: To view proccesses by the user running them
* `k` or `F9`: Kill a process, send `SIGKILL`
* `t` or `F5`: Display processes in Tree view
* `F6`: Sort by things such as State, Priority, and Time
* `F3`: Search for something
* `F7/F8`: `-/+` the nice livel (lower the level, higher the priority)

### Find:

To search recursively in your home directory for a file that contains the string `memes`:
```
$	grep 'memes' -r ~/*
```

To search in your home directory (not recursive) for a file that contains the string `memes`:
```
$	grep 'memes' -r ~/*
```

##### The rest of these searching are recursive

Find a file named `who` in your home directory:
```
$	find ~ -name "who"
```

Find all `.txt` files in your home directory:
```
$	find ~ -name "*.txt"
```

Search for all files with the string `be` in the name:
```
$	find ~ -name "*be*"
```

### USB:

##### OpenSUSE:

First create a directory to mount the USB:
```
$	sudo mkdir /mnt/usb/
```

Mount the USB Drive to the folder:
```
$	sudo mount /dev/sdb1 /mnt/usb
```

The contents of the `/mnt/usb` directory is now the contents of the flashdrive. To unmount the drive:
```
$	cd /
$	sudo umount /mnt/usb
```

##### Mint, CentOS, Ubuntu:

Create the directory to mount the usb drive:
```
$	sudo mkdir /media/flash_drive
```

Find the flash drive using fdisk

```
$	sudo fdisk -l
```

Mount the flash drive:
```
#	sudo mount /dev/sdb1 /media/flash_drive
```

Unmount the flash drive:
```
#	cd /
#	sudo unmount /media/flash_drive
```
