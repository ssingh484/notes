
## NMAP: 

``` nmap-scan
$ nmap -sC -sV -p- -T 4 exfiltrated.pg

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-20 20:35 UTC
Nmap scan report for exfiltrated.pg (192.168.51.163)
Host is up (0.0066s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.16 seconds
                                                            
```

## Website at port 80:

Subrion version 4.9 - File upload vulnerability by uploading PHP with .phar extension

```
	ncat -lvnp 9000
	
```

TTY Shell spawn:
```TTY-SHELLSPAWN
python3 -c 'import pty;pty.spawn("/bin/bash")'; export TERM=xterm-256color
```

Used LinPeas to find info:
	Found vulnerable cronjob run by root every minute
	Cron script was not editable but could be read
	Cron script ran exiftool on all jpg images in the uploads folder of the Subrion CMS
	Found exiftool version with ```-ver```
	Found [CVE-2021-22204](https://github.com/convisolabs/CVE-2021-22204-exiftool/tree/master) in exiftool 11.88
	Used [Exploit script](https://github.com/UNICORDev/exploit-CVE-2021-22204)
	Root shell spawned with nc listener