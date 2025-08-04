# NMAP

```TCP-Ports
22/tcp ssh
80/tcp http
```

```nmap-full-scan
└─$ sudo nmap -sC -sV -p- 10.10.11.177
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-14 20:01 EST
Nmap scan report for 10.10.11.177
Host is up (0.034s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 9e:1f:98:d7:c8:ba:61:db:f1:49:66:9d:70:17:02:e7 (RSA)
|   256 c2:1c:fe:11:52:e3:d7:e5:f7:59:18:6b:68:45:3f:62 (ECDSA)
|_  256 5f:6e:12:67:0a:66:e8:e2:b7:61:be:c4:14:3a:d3:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Is my Website up ?
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Port 80

Some sort of "is it up" checker website

Ran dirb and found a dev folder with what looks to be a whole git project and files that can be downloaded

