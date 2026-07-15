# NMAP

Me: 10.10.14.250
TARGET: 10.129.9.166

```
└─$ sudo nmap -sC -sV -p- -Pn 10.129.9.166 --max-retries 1 
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-05 01:49 -0400
Nmap scan report for 10.129.9.166
Host is up (0.049s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://silentium.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.67 seconds
```

# Port 80

Redirect to silentium.htb - added to hosts file to follow through

Simple "capital solutions" banking/fintech type website

Main page has a calculator with no text fields, only sliders
Also has a list of 3 names, could be useful for a userlist in enumeration

Navigating to /anything seems to redirect back which makes scanning with nikto or dirb harder to do

dirb only finds the assets folder with the simple JS for the sliders math

Tried domain and [[exploit-notes/web/api/index|subdomain_enum]] using ffuf and found staging.silentium.htb as another vHost on this box

```
─$ ffuf -u http://silentium.htb -c  -w /usr/share/wordlists/amass/subdomains-top1mil-110000.txt  -H "Host: FUZZ.silentium.htb" -mc 200 -o subdomains -fs 15949 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://silentium.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/amass/subdomains-top1mil-110000.txt
 :: Header           : Host: FUZZ.silentium.htb
 :: Output file      : subdomains
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
 :: Filter           : Response size: 15949
________________________________________________

staging                 [Status: 200, Size: 3142, Words: 789, Lines: 70, Duration: 62ms]
:: Progress: [114606/114606] :: Job [1/1] :: 1388 req/sec :: Duration: [0:01:27] :: Errors: 0 ::
```

## staging.silentium.htb

This appears to be a flowise website, some kind of visual/no-code AI agent builder

admin admin didn't work as default credentials

trying some of the leadership names as emails led to getting a different error message (invalid username or password instead of user not found) for ben@silentium.htb

This indicated ben@silentium.htb is a valid user, tried to use Hydra to brute force at this point but thankfully ran into a different issue at the same time

By searching for flowise password reset, there is a clear [vulnerability](https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-wgpv-6j63-x5ph) in versions <=3.0.5 where the reset token sent in email is also given out to the requester's browser:

```
{"user":{"id":"e26c9d6c-678c-4c10-9e36-01813e8fea73","name":"admin","email":"ben@silentium.htb","credential":"$2a$05$6o1ngPjXiRj.EbTK33PhyuzNBn2CLo8.b0lyys3Uht9Bfuos2pWhG","tempToken":"DwvRZNCoIVBzrD5scPuv7jEHnUgeLzrUPkjoKSZavpAhGgw2pSVAog7YNDOkvLyU","tokenExpiry":"2026-06-05T00:03:59.694Z","status":"active","createdDate":"2026-01-29T20:14:57.000Z","updatedDate":"2026-06-04T23:48:59.000Z","createdBy":"e26c9d6c-678c-4c10-9e36-01813e8fea73","updatedBy":"e26c9d6c-678c-4c10-9e36-01813e8fea73"},"organization":{},"organizationUser":{},"workspace":{},"workspaceUser":{},"role":{}}
```

This gives all info including the salted+hashed password too

Using the tempToken, able to just reset the password for ben@silentium.htb

This allows me to login to the flowise instance as ben which is an admin as seen in the name field for them

With authentication, I can make various components like agents, tools, tool MCPs and such. Seems like a vector for somehow getting arbitrary Javascript execution as flowise. Quick google search for Flowise RCE also shows this to be true as a GHSA [advisory](https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-3gcm-f6qx-ff7p)

This has a PoC showing how we can pass in arbitrary JS for the Function() constructor. Modified it to pass in a revshell from revshells.com and got a shell back


