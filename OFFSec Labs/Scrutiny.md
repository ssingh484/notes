## NMAP TCP

```
??$ nmap -sC -sV -p- 192.168.54.91 -oA Scrutiny
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-24 18:11 UTC
Nmap scan report for 192.168.54.91
Host is up (0.00056s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
25/tcp  open   smtp    Postfix smtpd
| ssl-cert: Subject: commonName=onlyrands.com
| Subject Alternative Name: DNS:onlyrands.com
| Not valid before: 2024-06-07T09:33:24
|_Not valid after:  2034-06-05T09:33:24
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: onlyrands.com, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
80/tcp  open   http    nginx 1.18.0 (Ubuntu)
|_http-title: OnlyRands
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp closed https
Service Info: Host:  onlyrands.com; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 118.66 seconds
```

## PORT 80

Website onlyrands.com
nginx 1.18.0

Found another subdomain being served:

teams.onlyrands.com

This is a JetBrains TeamCity instance version 2023.05.4 (build 129421)

Searchsploit results:
	RCE in 2023.05.3

Also found https://raw.githubusercontent.com/H454NSec/CVE-2023-42793/main/CVE-2023-42793.py

This did not work

Going to https://www.jetbrains.com/privacy-security/issues-fixed/?product=TeamCity shows that CVE-2024-27198 could be leveraged for auth bypass and admin actions

Found [exploit](https://github.com/W01fh4cker/CVE-2024-27198-RCE.git)

Using exploit to add admin user to teamcity let us in

In teamcity, found an id_rsa file uploaded accidentally by user marcot@onlyrands.com

Used ssh2john to turn this into a hash to crack the passphrase as it is needed for ssh

Found `marcot:cheer`

`ssh marcot@TARGET -i marcot`

followed by entering password cheer

## Low Privilege - Initial Foothold

Found local.txt

In /var/mail for marcot - found another username and pass:

`matthewa:IdealismEngineAshen476`

In matthewa home folder:

`Dach's password is "RefriedScabbedWasting502". I saw it once when he had to use my terminal to check TeamCity's status.`

So new user:

`briand:RefriedScabbedWasting502`

sudo -l yielded a result here as briand can run systemctl status teamcity-server.service

```
briand@onlyrands:/home$ sudo -l
Matching Defaults entries for briand on onlyrands:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User briand may run the following commands on onlyrands:
    (root) NOPASSWD: /usr/bin/systemctl status teamcity-server.service
```

Using this: [systemctl](https://gtfobins.github.io/gtfobins/systemctl/) as a GTFOBINs

Got shell as root

==DONE==

## Appendix

```

Marco,

Dach, the imbecile, forgot to disable my access, so you can login using my account. The password is "IdealismEngineAshen476" (without the quotation marcot).

I've left you a parting gift--your eyes only.
I'm gonna miss you, pal. Catch you on the flip side.

Sincerely,
Matthew A.
```

```
Dach's password is "RefriedScabbedWasting502". I saw it once when he had to use my terminal to check TeamCity's status.
```







