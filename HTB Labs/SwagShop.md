TARGET: 10.129.229.138
ME: 10.10.15.42

```NMAP
└─$ sudo nmap -sT -p- -Pn 10.129.229.138 --max-retries 1
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-07 16:42 -0500
Warning: 10.129.229.138 giving up on port because retransmission cap hit (1).
Nmap scan report for 10.129.229.138
Host is up (0.030s latency).
Not shown: 64000 closed tcp ports (conn-refused), 1533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 26.06 seconds
```

# Port 80

Apache httpd 2.4.29

Found a Magento installation

Dirbuster shows the /app directory which actually has a local.xml file with the install date and some SQL data too

Found the same installation date on a post auth RCE as well

Found CVE-2015-1397 which allows injecting a user in