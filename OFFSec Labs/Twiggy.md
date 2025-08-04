## NMAP

```nmap-tcp
??$ nmap -sC -sV -p- $TARGET -oA Twiggy         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-10 17:26 UTC
Stats: 0:01:16 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 66.42% done; ETC: 17:28 (0:00:39 remaining)
Nmap scan report for 192.168.55.62
Host is up (0.00075s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 44:7d:1a:56:9b:68:ae:f5:3b:f6:38:17:73:16:5d:75 (RSA)
|   256 1c:78:9d:83:81:52:f4:b0:1d:8e:32:03:cb:a6:18:93 (ECDSA)
|_  256 08:c9:12:d9:7b:98:98:c8:b3:99:7a:19:82:2e:a3:ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-title: Home | Mezzanine
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Site doesn't have a title (application/json).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 123.95 seconds

```

## DNS - 53

```dig

```

## Port 4505

ZeroMQ ZMTP 2.0 has CVE-2020-11651 and CVE-2020-11652
Found exploit [PoC](https://github.com/jasperla/CVE-2020-11651-poc)
Read passwd
Uploaded a new passwd with test user and password 12345

```openssl-passwd
openssl passwd 12345
```

==DONE==

