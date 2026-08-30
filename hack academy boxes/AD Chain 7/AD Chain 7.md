 
ME: 192.168.1.217
Target: 192.168.1.218

Assumed breach creds:

```
eturner : Ya-Boy14
```

Started by scanning only box I can network to:

```
└─$ sudo nmap -sC -sV -p- -Pn --max-retries=1 192.168.1.218 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-08 17:16 -0400
Stats: 0:01:36 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 93.75% done; ETC: 17:18 (0:00:05 remaining)
Nmap scan report for 192.168.1.218
Host is up (0.00036s latency).
Not shown: 65519 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: HACK-ACADEMY
|   NetBIOS_Domain_Name: HACK-ACADEMY
|   NetBIOS_Computer_Name: CLIENT-1
|   DNS_Domain_Name: hack-academy.local
|   DNS_Computer_Name: CLIENT-1.hack-academy.local
|   Product_Version: 10.0.19041
|_  System_Time: 2026-08-08T21:19:36+00:00
|_ssl-date: 2026-08-08T21:19:55+00:00; +2s from scanner time.
| ssl-cert: Subject: commonName=CLIENT-1.hack-academy.local
| Not valid before: 2026-04-19T14:02:45
|_Not valid after:  2026-10-19T14:02:45
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp  open  pando-pub?
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
53937/tcp open  msrpc         Microsoft Windows RPC
59550/tcp open  msrpc         Microsoft Windows RPC
MAC Address: 08:00:27:37:BC:43 (Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: CLIENT-1, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:37:bc:43 (Oracle VirtualBox virtual NIC)
|_clock-skew: mean: -1s, deviation: 2s, median: -3s
| smb2-time: 
|   date: 2026-08-08T21:19:36
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 194.39 seconds

```

The domain is hack-academy.local

Also used netexec to check for access given by our assumed breach creds

# SMB as eturner

```
┌──(xmen㉿kali)-[~]
└─$ netexec smb 192.168.1.218 -u 'eturner' -p 'Ya-Boy14' --shares
SMB         192.168.1.218   445    CLIENT-1         [*] Windows 10 / Server 2019 Build 19041 x64 (name:CLIENT-1) (domain:hack-academy.local) (signing:False) (SMBv1:None)
SMB         192.168.1.218   445    CLIENT-1         [+] hack-academy.local\eturner:Ya-Boy14 
SMB         192.168.1.218   445    CLIENT-1         [*] Enumerated shares
SMB         192.168.1.218   445    CLIENT-1         Share           Permissions     Remark
SMB         192.168.1.218   445    CLIENT-1         -----           -----------     ------
SMB         192.168.1.218   445    CLIENT-1         ADMIN$                          Remote Admin
SMB         192.168.1.218   445    CLIENT-1         C$                              Default share
SMB         192.168.1.218   445    CLIENT-1         InternalApps    READ            
SMB         192.168.1.218   445    CLIENT-1         IPC$            READ            Remote IPC
SMB         192.168.1.218   445    CLIENT-1         IT-Support      READ  
```

# RDP as eturner

```
└─$ netexec rdp 192.168.1.218 -u 'eturner' -p 'Ya-Boy14' 
RDP         192.168.1.218   3389   CLIENT-1         [*] Windows 10 or Windows Server 2016 Build 19041 (name:CLIENT-1) (domain:hack-academy.local) (nla:True)
RDP         192.168.1.218   3389   CLIENT-1         [+] hack-academy.local\eturner:Ya-Boy14 

```

