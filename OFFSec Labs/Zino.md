
# IP - 192.168.53.64

```nmap-zino
??$ nmap -sC -sV 192.168.53.64 -p- -oA Zino
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-28 14:16 UTC
Nmap scan report for 192.168.53.64
Host is up (0.00068s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b2:66:75:50:1b:18:f5:e9:9f:db:2c:d4:e3:95:7a:44 (RSA)
|   256 91:2d:26:f1:ba:af:d1:8b:69:8f:81:4a:32:af:9c:77 (ECDSA)
|_  256 ec:6f:df:8b:ce:19:13:8a:52:57:3e:72:a3:14:6f:40 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, NULL, SMBProgNeg, SSLSessionReq, TerminalServerCookie, oracle-tns: 
|_    Host '192.168.49.53' is not allowed to connect to this MariaDB server
8003/tcp open  http        Apache httpd 2.4.38
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-02-05 21:02  booked/
|_
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Index of /
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.94SVN%I=7%D=9/28%Time=66F8103D%P=x86_64-pc-linux-gnu%r
SF:(NULL,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.53'\x20is\x20not\x20
SF:allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(DNSStat
SF:usRequestTCP,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.53'\x20is\x20
SF:not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(
SF:SSLSessionReq,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.53'\x20is\x2
SF:0not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r
SF:(TerminalServerCookie,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.53'\
SF:x20is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20se
SF:rver")%r(SMBProgNeg,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.53'\x2
SF:0is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20serv
SF:er")%r(oracle-tns,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.53'\x20i
SF:s\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server
SF:");
Service Info: Hosts: ZINO, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h20m01s, deviation: 2h18m36s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2024-09-28T14:18:49
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: zino
|   NetBIOS computer name: ZINO\x00
|   Domain name: \x00
|   FQDN: zino
|_  System time: 2024-09-28T10:18:52-04:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 155.78 seconds
```

# Port 8003

**Web server serving a filesystem with booked directory which is booked scheduler**

Apache 2.4.38

Found some exploits for "booked scheduler"
Though the booked scheduler itself doesn't seem to load

```booked-scheduler
??$ searchsploit booked
------------------------------------------- ---------------------------------
 Exploit Title                             |  Path
------------------------------------------- ---------------------------------
Booked Scheduler 2.7.5 - Remote Command Ex | php/webapps/46486.rb
Booked Scheduler 2.7.5 - Remote Command Ex | php/webapps/50594.py
Booked Scheduler 2.7.7 - Authenticated Dir | php/webapps/48428.txt
------------------------------------------- ---------------------------------
Shellcodes: No Results
```