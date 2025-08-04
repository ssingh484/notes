# NMAP

## Ports - TCP:

```
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
8081/tcp open  blackice-icecap
```

## Ports - TCP - FullScan:

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.49.52
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 a2:ec:75:8d:86:9b:a3:0b:d3:b6:2f:64:04:f9:fd:25 (RSA)
|   256 b6:d2:fd:bb:08:9a:35:02:7b:33:e3:72:5d:dc:64:82 (ECDSA)
|_  256 08:95:d6:60:52:17:3d:03:e4:7d:90:fd:b2:ed:44:86 (ED25519)
80/tcp   open  http        Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16
|_http-title: Apache HTTP Server Test Page powered by CentOS
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp  open  netbios-ssn Samba smbd 4.10.4 (workgroup: SAMBA)
3306/tcp open  mysql       MariaDB (unauthorized)
8081/tcp open  http        Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
|_http-title: 400 Bad Request
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16
Service Info: Host: QUACKERJACK; OS: Unix

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.10.4)
|   Computer name: quackerjack
|   NetBIOS computer name: QUACKERJACK\x00
|   Domain name: \x00
|   FQDN: quackerjack
|_  System time: 2025-01-11T10:17:31-05:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-01-11T15:17:29
|_  start_date: N/A
|_clock-skew: mean: 1h40m01s, deviation: 2h53m14s, median: 0s
```

# Port 21 - FTP

- Anonymous FTP Login is allowed
- Run ftp client:
	- couldn't get any listing

# Port 80 - HTTP

Just some website, may enumerate more later

# Port 8081 - HTTPS

A self-signed cert
rConfig login page

`rConfig version 3.9.4`

Used exploit found by searchsploit that gets password hashes while unauthenticated (48208.py)

Modified it to use `verify=False` to ignore self-signed cert and prevent errors

Found hash for admin user which is MD5 for abgrtyu

```Admin-rConfig
admin:abgrtyu
```

Logged in with it and able to use another authenticated RCE exploit to get a reverse shell as well

# Local account

```
bash-4.2$ whoami
whoami
apache
```


Found local.txt flag

## Privilege Escalation

Run Linpeass:

Found DB Username and Password:

```
/home/rconfig/config/config.inc.php:define('DB_PASSWORD', 'RconfigUltraSecurePass');           
/home/rconfig/config/config.inc.php:define('DB_USER', 'rconfig_user');
```

```
???????????? SUID - Check easy privesc, exploits and write perms
? https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid
strace Not Found                                                                               
-rwsr-xr-x. 1 root root 195K Oct 30  2018 /usr/bin/find      
```

Looks like find with a suid bit which could be a GTFOBin for LoL escalation

Tried it

```
whoami
root
```

Got root

# Root user

Found flag

`d569fec7638635ecb1e9a9d6447f9db1`