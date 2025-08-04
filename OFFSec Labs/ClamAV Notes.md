## NMAP

``` nmap-scan
$ nmap -sC -sV -p- -T 4 192.168.51.42    

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-20 21:44 UTC
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 42.86% done; ETC: 21:44 (0:00:08 remaining)
Nmap scan report for 192.168.51.42
Host is up (0.0066s latency).
Not shown: 65528 closed tcp ports (conn-refused)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
25/tcp    open  smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
| smtp-commands: localhost.localdomain Hello [192.168.49.51], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp    open  http        Apache httpd 1.3.33 ((Debian GNU/Linux))
|_http-title: Ph33r
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp   open  smux        Linux SNMP multiplexer
445/tcp   open  netbios-ssn Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
60000/tcp open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.14a-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-01-20T21:44:55-05:00
|_clock-skew: mean: 7h29m58s, deviation: 3h32m07s, median: 4h59m58s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: share (dangerous)
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: 0XBABE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.05 seconds
```

``` nmap-scan-udp
$ sudo nmap -sU -sV --top-ports=100 192.168.51.42

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-20 22:00 UTC
Stats: 0:00:07 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 21.83% done; ETC: 22:01 (0:00:25 remaining)
Stats: 0:01:06 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 4.76% done; ETC: 22:05 (0:03:20 remaining)
Stats: 0:02:56 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 54.76% done; ETC: 22:05 (0:01:39 remaining)
Nmap scan report for 192.168.51.42
Host is up (0.00060s latency).
Not shown: 58 closed udp ports (port-unreach), 40 open|filtered udp ports (no-response)
PORT    STATE SERVICE    VERSION
137/udp open  netbios-ns Samba nmbd netbios-ns (workgroup: WORKGROUP)
161/udp open  snmp       SNMPv1 server; U.C. Davis, ECE Dept. Tom SNMPv3 server (public)
Service Info: Hosts: 0XBABE, 0xbabe.local

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 230.13 seconds

```

## SNMP port 161

```snmpwalk
snmpwalk -v2c -c public <target-ip>
```

Found that clamav is running in blackhole mode for sendmail on port 25
Found clamav-milter exploit
ran it to pop shell with
```searchsploit-clamav
searchsploit --cve 2007-4560
```
