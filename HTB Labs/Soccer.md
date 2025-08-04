# NMAP

## Target

`10.10.11.194`

## Attacker

`10.10.14.4`

## TCP Ports

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-23 19:05 CST
Nmap scan report for 10.10.11.194
Host is up (0.011s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
9091/tcp open  xmltec-xmlmail
```

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-23 19:15 CST
Nmap scan report for 10.10.11.194
Host is up (0.0092s latency).

PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
9091/tcp open  xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 139
|     Date: Fri, 24 Jan 2025 01:17:40 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot GET /</pre>
|     </body>
|     </html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Date: Fri, 24 Jan 2025 01:17:40 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|_    </html>
```

# Port 80

soccer.htb - needed to add to hosts file

Used Dirb as well as [[exploit-notes/web/api/index|ffuf]] to enumerate directories and subdomains

Found no subdomains - all return 301 (page size 178)

Found /tiny directory which is "H3K Tiny File Manager"

Found file upload vulnerability by logging in with default creds: 
```
admin : admin123
```

Can upload to /var/www/html/tiny/uploads as admin and get shell as www-data

Cannot also upload to web root and still get shell

Regardless, got shell as www-data by uploading to tiny/uploads

# Low-level shell: www-data

Made fully interactive tty by following [[upgrade-to-fully-interactive-tty|TTY]]

Ran [[exploit-notes/linux/privilege-escalation/index|index]]

Found an nginx conf for another subdomain soc-player.soccer.htb by running cat command from [[exploit-notes/linux/privilege-escalation/index#Interesting Information|privilege_escalation_interesting_info]] to find out nginx conf

Added this subdomain to the hosts file and navigated to it

Had a login for which I made a new test user (no usual credentials worked for logging in)

Found a /check after logging in with a field that was being used (per page source) in a websocket request to port 9001 on the box

Used burpsuite to intercept this websocket request via proxy on page change

Found that adding `OR 1=1` created a false positive result showing blind SQLi

Used [[sql-injection-with-sqlmap]] to dump databases and find soccer_db with player's password

Used SSH to get to a session as player

# Low-Level Shell: Player

Running [[/exploit/linux/privilege-escalation/|linpeas]]

Found that doas is available for player user as a set-uid binary and lets us run dstat as root

dstat is a [[sudoedit-privilege-escalation|GTFOBINs]] Binary so followed instructions there

Got root shell

==DONE==


