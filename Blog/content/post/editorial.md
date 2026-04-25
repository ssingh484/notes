+++
date = '2026-04-20T22:25:04-04:00'
draft = true
title = 'Editorial'
+++

Editorial is an easy-rated Linux box on HackTheBox. The box presents a publishing platform site (`editorial.htb`) — only SSH and HTTP open. Got the domain from the nmap scan, added it to `/etc/hosts`, and went poking at the web app.

**Target:** `10.10.11.20`

---

## Enumeration

### NMAP

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

Standard two-port box — SSH locked down, all the action is going to be on port 80. The nmap title leaked the domain name `editorial.htb` so I added that to `/etc/hosts` before going further.

---

## Port 80

The site is running nginx 1.18.0 and hosts what looks like a book publishing platform — "Editorial Tiempo Arriba". Nothing immediately exploitable on the surface, but it's a web app so there's somewhere to go from here.
