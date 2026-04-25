+++
date = '2026-04-20T21:18:23-04:00'
draft = false
author = 'Siddhant Singh'
title = 'Boardlight'
+++

BoardLight is an easy-rated Linux box on HackTheBox. The path goes through subdomain enumeration, exploiting an authenticated RCE in a Dolibarr CRM install, credential reuse to get a foothold as a real user, and then a SUID binary on an outdated version of Enlightenment to pop root.

---

## Enumeration

### NMAP

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Standard two-port box — SSH and a web server. Nothing exotic on the surface.

### Port 80

Port 80 gave a fairly basic static-looking website. Nothing immediately interesting on the surface, but the domain `board.htb` was enough of a hint to go looking for subdomains.

I ran subdomain enumeration using `ffuf`, filtering out the default page size to cut out false positives:

```
ffuf -u http://board.htb -c  -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-11000.txt  -H "Host: FUZZ.board.htb" -mc 200 -o subdomains -fs 15949
```

That turned up `crm.board.htb`. Added it to `/etc/hosts` and browsed over.

---

## Foothold — Dolibarr RCE (CVE-2023-30253)

`crm.board.htb` turned out to be **Dolibarr CRM/ERP version 17.0.0** — and the default credentials `admin/admin` worked straight away on the login page.

I found a public exploit for authenticated RCE against this version:

> [Exploit-for-Dolibarr-17.0.0-CVE-2023-30253](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253?tab=readme-ov-file)

CVE-2023-30253 abuses a PHP code injection vulnerability in Dolibarr's website module. Since I had valid credentials (even if just admin/admin), I could trigger the RCE and land a shell as `www-data`.

---

## Lateral Movement — www-data → larissa

Once I had a shell as `www-data`, I started digging around the web root. Inside the Dolibarr configuration I found `conf.php`, which had database credentials in plaintext:

```
$dolibarr_main_db_user='dolibarrowner';
$dolibarr_main_db_pass='serverfun2$2023!!';
```

I took a shot at password reuse for SSH. Checked `/etc/passwd` style — the low-privilege local user was `larissa`. Tried the DB password against her SSH account and it worked.

```
ssh larissa@board.htb
```

---

## Privilege Escalation — larissa → root

### LinPEAS

I ran `linpeas` to look for anything interesting. It flagged some **Enlightenment** binaries with the SUID bit set — specifically version **0.25.3**.

A quick search turned up a local privilege escalation exploit on Exploit-DB targeting this exact version. Ran it and got root.

### Root

```
# id
uid=0(root) gid=0(root) groups=0(root)
```
