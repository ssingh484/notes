# IP

TARGET:

```
192.168.54.153
```

ME:

```
192.168.49.54
```

# NMAP

```
??$ nmap -sS -p- 192.168.54.153                                         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-06 18:11 UTC
Stats: 0:01:42 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 95.20% done; ETC: 18:13 (0:00:05 remaining)
Nmap scan report for 192.168.54.153
Host is up (0.00052s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8000/tcp open  http-alt
```

```

```


Port 8000

Found a "secure REPL" that makes md5 hash of API KEY concatenated with the code (in b64)

Used john mask mode (hybrid masked mode)

```
john hash --format=Raw-MD5 --mask=?wZnVuY3Rpb24gaGVsbG8obmFtZSkgewogIHJldHVybiAnSGVsbG8gJyArIG5hbWUgKyAnISc7Cn0KCmhlbGxvKCdXb3JsZCcpOyAvLyBzaG91bGQgcHJpbnQgJ0hlbGxvIFdvcmxkJw== -w=<WORDLIST>
```