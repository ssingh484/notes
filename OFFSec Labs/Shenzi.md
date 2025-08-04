
## NMAP

```nmap-shenzi
??$ nmap -sC -sV 192.168.56.55 -oA Shenzi
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-16 14:28 UTC
Nmap scan report for 192.168.56.55
Host is up (0.00080s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
80/tcp   open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.56.55/dashboard/
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
| http-title: Welcome to XAMPP
|_Requested resource was https://192.168.56.55/dashboard/
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
3306/tcp open  mysql?
| fingerprint-strings: 
|   NULL: 
|_    Host '192.168.49.56' is not allowed to connect to this MariaDB server
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.94SVN%I=7%D=6/16%Time=666EF68C%P=x86_64-pc-linux-gnu%r
SF:(NULL,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.56'\x20is\x20not\x20
SF:allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Port 80

Basic XAMPP 7.4.6
Not sure about any good RCE style exploit

Tried dirb
Did not find the shenzi folder(which is really wordpress site)
But found it after looking at smb share

/admin gives wordpress login page

```creds
5) WordPress:

   User: admin
   Password: FeltHeadwallWight357
```

Found a [good article](https://sevenlayers.com/index.php/179-wordpress-plugin-reverse-shell) on using wordpress admin UI to drop a reverse-shell via a PHP Plugin

Followed those steps to get shell - didn't quite work

*Went into appearance and theme editor*

Edited the 404 page to be Ivan Cineck revshell
## Port 21

fileZilla ftpd 0.9.41
can't do anonymous login

## Port 3306

MYSQL:

```mysql-nmap-enum
nmap -sV -p 3306 --script mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122 <IP>
```

tried to connect `mysql -h <Hostname> -u root`
didn't work

## Port 445 (SMB)

```
???(kali?kali)-[~]
??$ smbcsmbclient -U ''  -L \\\\192.168.56.55\\
Password for [WORKGROUP\]:

        Sharename       Type      Comment
        ---------       ----      -------
        IPC$            IPC       Remote IPC
        Shenzi          Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 192.168.56.55 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
                                                         
```

Can access Shenzi share without login:

```shenzi-smb
???(kali?kali)-[~]
??$ smbclient -U ''  \\\\192.168.56.55\\Shenzi
Password for [WORKGROUP\]:
Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!              
smb: \> 
```

Considering the passwords file mentioned wordpress and had wordpress credentials:

`http://192.168.56.55/shenzi/`
exists, though wasn't found by dirbuster
[[Shenzi#Port 80]]

## Local Shell

Got shell via php revshell as user shenzi

```local.txt
f7efa6c2938d40b0fa1478913068b86c
```

### Priv Esc

Checked for [[Always Install Elevated]]
Seems like we can use this method as both reg keys show up as 0x1