# NMAP TCP

TARGET: 10.129.6.121
ME: 10.10.15.62

```
└─$ sudo nmap -sC -sV -p- -Pn 10.129.6.121            
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-31 15:11 -0500
Stats: 0:00:09 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 5.77% done; ETC: 15:13 (0:02:11 remaining)
Nmap scan report for 10.129.6.121
Host is up (0.036s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12
|_http-title: Did not follow redirect to http://popcorn.htb/
|_http-server-header: Apache/2.2.12 (Ubuntu)
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.57 seconds
```

# Port 80

on popcorn.htb/torrent

There is a Torrent Hoster which has a file upload vuln in screenshots for existing torrents by faking mime type in POST request

Uploaded reverse php shell

Used ![[upgrade-to-fully-interactive-tty]]
# shell as www-data

Found db creds and an old version of mysql using linpeas:

```
╔══════════╣ Searching passwords in config PHP files
/var/www/torrent/config.php:    $dbpass         = $CFG->dbPassword;                                                                                                                                                                         
/var/www/torrent/config.php:    $dbuser         = $CFG->dbUserName;
/var/www/torrent/config.php:  $CFG->dbPassword = "SuperSecret!!";       //db password
/var/www/torrent/config.php:  $CFG->dbUserName = "torrent";    //db username
```

Can login and then use ![[mysql-pentesting#Local]]
