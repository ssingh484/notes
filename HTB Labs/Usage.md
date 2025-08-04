# NMAP

**Target:** `10.10.11.18`
**Source:** `10.10.14.23`

## TCP Scan

```
──╼ [★]$ nmap -sC -sV -p22,80 10.10.11.18 -oA Usage
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-25 17:27 CST
Nmap scan report for 10.10.11.18
Host is up (0.0088s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a0:f8:fd:d3:04:b8:07:a0:63:dd:37:df:d7:ee:ca:78 (ECDSA)
|_  256 bd:22:f5:28:77:27:fb:65:ba:f6:fd:2f:10:c7:82:8f (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://usage.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.04 seconds
```

# Port 80

set hosts to usage.htb for this since it's a redirect

Nginx version 1.18.0

On visiting usage.htb, there is an admin subdomain admin.usage.htb