+++
date = '2026-04-20T21:48:24-04:00'
draft = true
title = 'Dog'
+++

Dog is an easy-rated Linux box on HackTheBox. The box is running a Backdrop CMS site with a publicly accessible `.git` directory — pulling that down reveals a database password in `settings.php` and a username buried in the commit history. That gets me authenticated into the CMS, where a known CSRF-to-RCE exploit lets me drop a PHP shell. From there `su` to a real user works with the same recycled password, and a misconfured `sudo` rule on the Backdrop CLI tool hands over root.

**Target:** `10.129.231.223`  
**My IP:** `10.10.14.208`

---

## Enumeration

### NMAP

Used my [NMAP oneliner](https://github.com) — full-port scan first, then feed results into a `-sC -sV` run.

```
nmap -sC -sV -p$(nmap -p- -Pn 10.129.231.223 | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") 10.129.231.223
```

---

## Port 80 — Backdrop CMS & Git Exposure

Browsed to the site on port 80. The first thing I noticed was a `.git` folder sitting in the web root — publicly accessible, no restrictions. Pulled the whole thing down with `git-dumper`.

Inside the dumped repo I found `settings.php` which had a plaintext MySQL connection string:

```
mysql://root:BackDropJ2024DS2024
```

The git commit log also leaked a username. Grepping through the dump for any `@dog.htb` addresses:

```
└─$ grep -r "@dog.htb" .
./files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/update.settings.json:        "tiffany@dog.htb"
./.git/logs/refs/heads/master:0000000000000000000000000000000000000000 8204779c764abd4c9d8d95038b6d22b6a7515afa root <dog@dog.htb> 1738963331 +0000     commit (initial): todo: customize url aliases. reference:https://docs.backdropcms.org/documentation/url-aliases
./.git/logs/HEAD:0000000000000000000000000000000000000000 8204779c764abd4c9d8d95038b6d22b6a7515afa root <dog@dog.htb> 1738963331 +0000  commit (initial): todo: customize url aliases. reference:https://docs.backdropcms.org/documentation/url-aliases
```

That gave me `tiffany@dog.htb` from the config file. Tried logging in to the CMS with that address and the database password — it worked.

**Credentials:** `tiffany@dog.htb` / `BackDropJ2024DS2024`

---

## Foothold — CSRF to RCE on Backdrop CMS

With an authenticated session on the CMS, I looked for known exploits against Backdrop and found [this CSRF-to-RCE PoC](https://github.com/V1n1v131r4/CSRF-to-RCE-on-Backdrop-CMS). The exploit works by uploading a malicious module — essentially a `.tar` archive containing a PHP file — through the admin module installer.

I modified the reference `.tar` from the PoC and repacked it with a PHP reverse shell from [revshells.com](https://revshells.com), then triggered the upload through the exploit path. Got a callback as `www-data`.

---

## Shell as www-data

Landed a shell. Read `/etc/passwd` to map out the users on the box:

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
fwupd-refresh:x:111:116:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:113:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
jobert:x:1000:1000:jobert:/home/jobert:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:114:119:MySQL Server,,,:/nonexistent:/bin/false
johncusack:x:1001:1001:,,,:/home/johncusack:/bin/bash
_laurel:x:997:997::/var/log/laurel:/bin/false
```

Two users with login shells: `jobert` and `johncusack`. Ran LinPEAS to look for anything obvious while I figured out the lateral movement path.

---

## Lateral Movement — johncusack

Tried `su johncusack` with the same password from the database connection string (`BackDropJ2024DS2024`) and it worked. Password reuse across the CMS database config and a local system user.

---

## Privilege Escalation — bee (Backdrop CLI) via sudo

Checked `sudo -l` as johncusack. The user can run `bee` — the Backdrop CMS command-line tool — as root with no password required.

`bee` has a PHP eval capability built in for executing custom PHP against the CMS. Since it runs as root under sudo, that's a clean path to a root shell.
