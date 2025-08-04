# NMAP

```
└──╼ [★]$ nmap -sC -sV -p22,80 10.10.11.20 -oA Editorial
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-04 18:24 CST
Nmap scan report for editorial.htb (10.10.11.20)
Host is up (0.0091s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
|_  256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Editorial Tiempo Arriba
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

# Port 80

Found domain from nmap scan so added to hosts file and redid scan

Nginx 1.18.0
hosting some kind of a blog website