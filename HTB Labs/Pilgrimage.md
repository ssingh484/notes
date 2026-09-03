TARGET: 10.129.51.87
ME: 10.10.15.8

```
┌──(xmen㉿kali)-[~]
└─$ sudo nmap -sC -sV -p- -Pn --max-retries=1 10.129.51.87
[sudo] password for xmen: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-01 20:11 -0400
Nmap scan report for 10.129.51.87
Host is up (0.078s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
|_  256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: Did not follow redirect to http://pilgrimage.htb/
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.04 seconds
```

# Port 80

Added pilgrimage.htb to hosts file and navigated to the website being served behind nginx 1.18.0

This seems to be some kind of "free image shrinker" with an unauthenticated file upload for the image to be shrunk.

There is a registration which allows for our  "shrunk" files to show up as links in a dashboard allowing going to them and reviewing past submissions.

Running nmap again after adding the vHost saved me from having to run dirb as it seems the .git directory for this (what appears to be a git repo) is accessible. From here I should be able to reconstruct the filesystem and the PHP code for the submitted images to see how to upload a webshell.

```
80/tcp open  http    nginx 1.18.0
| http-git: 
|   10.129.51.87:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: Pilgrimage image shrinking service initial commit. # Please ...
|_http-title: Pilgrimage - Shrink Your Images
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```




