+++
date = '2026-04-25T11:29:03-04:00'
draft = false
title = 'Sau'
+++

Sau is an easy-rated Linux box on HackTheBox. The attack chain starts with an SSRF vulnerability in Request Baskets 1.2.1 to reach an internally-bound Maltrail v0.53 instance, exploit an unauthenticated RCE for a reverse shell as `puma`, then abuse a `sudo` permission on `systemctl status` to escape to a root shell.

**Target:** `10.10.11.224`

---

## Enumeration

### NMAP

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-20 14:19 CST
Nmap scan report for 10.10.11.224
Host is up (0.0093s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    filtered http
8338/tcp  filtered unknown
55555/tcp open     unknown
```

SSH on 22, port 80 and 8338 filtered (not reachable directly), and something on 55555. The filtered ports are interesting — something is running there but not exposed externally.

---

### Port 55555

Browsing to port 55555 shows a web UI for **Request Baskets version 1.2.1**. Searchsploit came up empty, but a quick search online surfaced a known SSRF vulnerability in this version that lets you configure a basket to forward requests to arbitrary internal network resources.

Since port 80 showed as filtered in nmap (running but not externally accessible), I set up a basket pointing to `http://127.0.0.1:80` to reach it via the SSRF.

---

### Port 80 (via SSRF)

With the basket proxy in place, port 80 turns out to be running **Maltrail v0.53** — a network traffic monitoring tool.

---

## Foothold — Maltrail v0.53 Unauthenticated RCE

Maltrail v0.53 has a known unauthenticated RCE in its login endpoint. I found a public exploit for it and chained it through the Request Baskets SSRF to hit the internal Maltrail service.

Set up a `nc` listener then fired the exploit — got a shell back as `puma`.

---

## Privilege Escalation — puma → root

Found `user.txt` in puma's home directory.

Ran `sudo -l` to check what puma can run:

```
$ sudo -l
Matching Defaults entries for puma on sau:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User puma may run the following commands on sau:
    (ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service
```

`systemctl status` pipes its output through a pager (less). Since this runs as root via sudo, I can break out of it with `!/bin/bash` to spawn a root shell — a classic pager escape.

```bash
sudo /usr/bin/systemctl status trail.service
# in the pager:
!/bin/bash
```

### Root

Got a root shell and grabbed `root.txt`.
