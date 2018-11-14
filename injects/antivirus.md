# ClamAV Inject

#### Ubuntu

Install  ClamAV:
```
$	sudo apt-get install clamav
```

Update the file signatures for ClamAV:
```
$	sudo freshclam
```

To check the version of ClamAV:
```
$	clamscan -V
ClamAV 0.99.3/24315/Wed Feb 14 09:29:19 2018
```

You should be able to scan the entire root filesystem recursively, and store the summary in `~/av_scan_injext`

```
$	sudo clamscan -r clamscan -r / -l ~/av_clamscan_inject
```

Now the scan report will have a list of all the files that it scanned. If you want to delete all of them so only the summary remains open it in `Vim`:

```
$	sudo vim ~/av_clamscan_injext
```

Run the following commands. They just search for all lines with the string inbetween the `/` characters, and deletes them (or you can just copy the results to another file).

```
:g/ok/d
:g/Symbolic/d
:g/proc/d
:g/ERROR/d
:g/Empty/d
:g/Permission/d
```

#### CentOS

Install ClamAV (you will need to install Extra Packages for Enterprise Linux first):
```
$	sudo yum install epel-release
$	sudo yum install clamav clamav-update
```

Update the ClamAV database for signatures:
```
$	sudo freshclam
```

get the current version of ClamAV:
```
$	clamscan -V
ClamAV 0.99.3/24315/Wed Feb 14 12:29:19 2018
```

to scan the entire root filesystem recursively, and store the output fo the scan in `~/av_scan_inject`:
```
$	clamscan -r / -l ~/av_clamscan_inject
```

See `Ubuntu` section for filtering the report.

#### Remote Scan:
This will be covering scanning a Centos image from Ubuntu, and Vice Ve

On Ubuntu install `sshfs`, which will allow you to mount another system's filesystem:
```
$	 sudo apt-get install sshfs
```

On CentOS, you will also need to install `sshfs`:
```
$	sudo yum install sshfs
```

Next you will need to create the directory which we will mount the remote file system to:
```
$	sudo mkdir /mnt/scan_remote
```

Next we can mount the remote file system at `192.168.234.145`, using the `wheel` user `guyinatuxedo`:
```
$	sudo sshfs -o allow_other guyinatuxedo@192.168.234.145:/ /mnt/scan_remote/
```

And to scan the remote filesystem at `192.168.234.144`, using the `sudo` user `guyinatuxedo`:
```
sudo sshfs -o allow_other guyinatuxedo@192.168.234.144:/ /mnt/scan_remote/
```

and then you can scann the remote files system (in both cases) with the following command:
```
#	sudo clamscan -r /mnt/scan_remote/ -l ~/remote_av_scan
```

apply the output filtering regulations as wanted look at the last part of the `Ubuntu` segment.


#### Issues

May run into an issue while trying to update, the log file `/var/log/clamav/freshclam.log` is being used by another process. To free it, first find the Process ID:

```
$	sudo lsof /var/log/clamav/freshclam.log 
lsof: WARNING: can't stat() fuse.gvfsd-fuse file system /run/user/108/gvfs
      Output information may be incomplete.
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
freshclam 860 clamav    3wW  REG    8,1     2288 551220 /var/log/clamav/freshclam.log
```

Next kill the process:
```
$	sudo kill -9 860
```
