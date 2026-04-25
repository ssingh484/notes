
# NMAP TCP

Used [[NMAP Oneliner]]

TARGET: 10.129.231.194
ME: 10.10.14.208

```
└─$ nmap -sC -sV -p$(nmap -p- -Pn 10.129.231.194 | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") 10.129.231.194
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-06 17:55 EST
Nmap scan report for 10.129.231.194
Host is up (0.028s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
|_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
80/tcp open  http    Apache httpd
|_http-title: Did not follow redirect to http://linkvortex.htb/
|_http-server-header: Apache
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.66 seconds
```

# Port 80

Needed to add linkvortex.htb to /etc/hosts

Found a blog running Ghost CMS

Nothing found by dirbuster

found dev via [[exploit-notes/web/api/index|subdomain_enum]] :

```
ffuf -u http://linkvortex.htb -c  -w /usr/share/wordlists/amass/subdomains-top1mil-110000.txt  -H "Host: FUZZ.linkvortex.htb" -mc 200 -o subdomains -fs 15949
```

Added dev to /etc/hosts

Dirb found a .git directory which can be dumped using git-dumper to get source code

Dockerfile shows version 5.58.0 of Ghost

running git status shows some untracked file changes not commited yet

running git diff --cached shows that a file changed:

```
--- a/ghost/core/test/regression/api/admin/authentication.test.js
+++ b/ghost/core/test/regression/api/admin/authentication.test.js
@@ -53,7 +53,7 @@ describe('Authentication API', function () {
 
         it('complete setup', async function () {
             const email = 'test@example.com';
-            const password = 'thisissupersafe';
+            const password = 'OctopiFociPilfer45';
 
             const requestMock = nock('https://api.github.com')
                 .get('/repos/tryghost/dawn/zipball')
```

Looks like a password OctopiFociPilfer45 which works for admin@linkvortex.htb on the main site

Also found that version 5.58.0 has a vulnerability allowing arbitrary file read - https://scout.docker.com/vulnerabilities/id/CVE-2023-40028?s=github&n=ghost&t=npm&vr=%3C5.59.1&utm_source=hub&utm_medium=ExternalLink&_gl=1*uh8vhu*_ga*MTI0NDEzMDczNi4xNzY3NzQyMDk4*_ga_XJWPQMJYHQ*czE3Njc3NDIwOTgkbzEkZzEkdDE3Njc3NDIxNzEkajYwJGwwJGgw

This was found by examining that image on dockerhub for vulnerabilities in ghost

Using a PoC for CVE-2023-40028 we can read the file mentioned in the dockerfile `/var/lib/ghost/config.production.json`

This shows:

```
{
  "url": "http://localhost:2368",
  "server": {
    "port": 2368,
    "host": "::"
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": ["stdout"]
  },
  "process": "systemd",
  "paths": {
    "contentPath": "/var/lib/ghost/content"
  },
  "spam": {
    "user_login": {
        "minWait": 1,
        "maxWait": 604800000,
        "freeRetries": 5000
    }
  },
  "mail": {
     "transport": "SMTP",
     "options": {
      "service": "Google",
      "host": "linkvortex.htb",
      "port": 587,
      "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
      }
    }
}
```

# SSH as bob

Can SSH using bob@linkvortex.htb : fibber-talented-worth

User flag found

sudo -l shows that the /opt/ghost/clean_symlink.sh script can be ran without password

It also shows that the CHECK_CONTENT environment var is preserved

```clean_symlink.sh
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi
```


This can allow a file to be a symlink to something benign where it moves to quarantine and another process modifying the symlink before the CHECK_CONTENT conditional is executed. This is a TOC-TOU vulnerability.

By running a `while true; do x; done` loop forcing a symlink to /root/.ssh/id_rsa in one terminal and then running the sudo command, we can exploit the TOCTOU

Gives id_rsa for root

# SSH as root

Get root.txt

==DONE==

