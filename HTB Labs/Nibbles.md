TARGET: 10.129.35.135
ME: 10.10.15.8

# NMAP

```
┌──(xmen㉿kali)-[~]
└─$ sudo nmap -sC -sV -p- -Pn 10.129.35.135 --max-retries=1
[sudo] password for xmen: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-28 18:23 -0400
Warning: 10.129.35.135 giving up on port because retransmission cap hit (1).
Host is up (0.027s latency).
Not shown: 65396 closed tcp ports (reset), 137 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Port 80

Basic hello world website

Apache/2.4.18 (Ubuntu)

Main hello world page has a message in the source:

```
<html><head></head><body><b>Hello world!</b>

<!-- /nibbleblog/ directory. Nothing interesting here! -->
</body></html>
```

This leads to an atom blog with directory listing on http://10.129.35.135/nibbleblog/admin/  and an admin panel on http://10.129.35.135/nibbleblog/admin.php