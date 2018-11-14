# nmap scan
### Installation:

Ubuntu:
```
$	sudo apt-get install nmap
```

CentOS:
```
$	sudo yum install nmap
```

### Scan

To run a quick ping scan on the 172.16.103/24 network:
```
#	nmap -sP 172.16.103.0/24 > ~/preport
```

To scan the 172.16.103.0/24 network, check for ports between 1-65535, and try to see what the host is:
```
#	nmap -p 1-65535 -sV -sS -T4 172.16.103.0/24 > ~/sreport
```