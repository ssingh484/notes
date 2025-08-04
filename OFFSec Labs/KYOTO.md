## NMAP TCP

```nmap-tcp-kyoto
??$ nmap -sC -sV -Pn $TARGET
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-14 13:58 UTC
Nmap scan report for 192.168.57.31
Host is up (0.00088s latency).
Not shown: 985 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp
| fingerprint-strings: 
|   GenericLines, Help, NULL, SMBProgNeg, SSLSessionReq: 
|_    220 Welcome to Simple FTP Server NG
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-04-14 13:58:19Z)
111/tcp  open  rpcbind?
| rpcinfo: 
|   program version    port/proto  service
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|_  100005  1,2,3       2049/udp6  mountd
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: Kyotosoft.com0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2049/tcp open  mountd        1-3 (RPC #100005)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: Kyotosoft.com0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: KYOTOSOFT
|   NetBIOS_Domain_Name: KYOTOSOFT
|   NetBIOS_Computer_Name: KYOTO
|   DNS_Domain_Name: Kyotosoft.com
|   DNS_Computer_Name: kyoto.Kyotosoft.com
|   DNS_Tree_Name: Kyotosoft.com
|   Product_Version: 10.0.20348
|_  System_Time: 2024-04-14T13:59:00+00:00
| ssl-cert: Subject: commonName=kyoto.Kyotosoft.com
| Not valid before: 2024-04-13T13:56:26
|_Not valid after:  2024-10-13T13:56:26
|_ssl-date: 2024-04-14T13:59:40+00:00; 0s from scanner time.
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.94SVN%I=7%D=4/14%Time=661BE0FB%P=x86_64-pc-linux-gnu%r(N
SF:ULL,25,"220\x20Welcome\x20to\x20Simple\x20FTP\x20Server\x20NG\r\n")%r(G
SF:enericLines,27,"220\x20Welcome\x20to\x20Simple\x20FTP\x20Server\x20NG\r
SF:\n\r\n")%r(Help,27,"220\x20Welcome\x20to\x20Simple\x20FTP\x20Server\x20
SF:NG\r\n\r\n")%r(SSLSessionReq,27,"220\x20Welcome\x20to\x20Simple\x20FTP\
SF:x20Server\x20NG\r\n\r\n")%r(SMBProgNeg,27,"220\x20Welcome\x20to\x20Simp
SF:le\x20FTP\x20Server\x20NG\r\n\r\n");
Service Info: Hosts: Simple, KYOTO; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-04-14T13:59:01
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 106.30 seconds
```


## TCP 21 (FTP)

Nothing

## TCP 389 (LDAP)

Needs credential for ldapsearch

## TCP 445 (SMB)

anonymous session

```smbclient-anonymous
smbclient -N -L //<target-ip>
```

 Found a non-standard share dev
```smbclient-share
smbclient -N \\\\<target-ip>\\dev 
```

Downloaded Files
Buffer Overflow