# NMAP TCP

**TARGET** 10.129.10.54
**ME** 10.10.15.75

```nmap-tcp
─$ sudo nmap -sC -sV -p- -Pn 10.129.10.54 --max-retries 1
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-26 21:24 -0500
Stats: 0:03:02 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 93.94% done; ETC: 21:27 (0:00:00 remaining)
Stats: 0:04:09 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 95.83% done; ETC: 21:28 (0:00:00 remaining)
Nmap scan report for 10.129.10.54
Host is up (0.032s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  http    JRun Web Server
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 252.01 seconds
```

# Port 8500

A JRun WebServer 
Running coldfusion 8

Found a few exploits for a few vulns on searchsploit

Used 50057.py -> Uploading and running a JSP allowing RCE

```
C:\ColdFusion8\runtime\bin>whoami
whoami
arctic\tolis

C:\ColdFusion8\runtime\bin>
```

# Shell as tolis

systeminfo and winPEAS shows no hotfixes applied

Could have used windowsExploitSuggestor to find out that the chimmichuri exploit would work

Used pre-compiled exe to get shell as root

==DONE==