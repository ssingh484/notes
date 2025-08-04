# IP

TARGET:

```
192.168.60.166
```

ME:

```
192.168.49.60
```

# NMAP

```
??$ nmap -sS -p- 192.168.60.166
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-09 00:36 UTC
Nmap scan report for 192.168.60.166
Host is up (0.00084s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
6379/tcp open  redis
```

```full-nmap-readys
??$ nmap -sC -sV -p22,80,6379 192.168.60.166 -oA Readys
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-09 00:37 UTC
Nmap scan report for 192.168.60.166
Host is up (0.00096s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Readys &#8211; Just another WordPress site
|_http-generator: WordPress 5.7.2
|_http-server-header: Apache/2.4.38 (Debian)
6379/tcp open  redis   Redis key-value store
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.06 seconds
```

# Port 80

simple wordpress website

wordpress 5.7.2
Apache 2.4.38