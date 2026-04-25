+++
date = '2026-04-20T21:32:37-04:00'
draft = true
title = 'Broker'
+++

Broker is an easy-rated Linux box on HackTheBox. The path runs through a heavily exposed Apache ActiveMQ instance, exploiting a critical RCE vulnerability (CVE-2023-46604) to land a shell, then abusing a NOPASSWD nginx sudo entry to escalate to root.

---

## Enumeration

### NMAP

I ran a two-stage scan — first a fast full-port sweep, then a service/version scan against everything that responded:

```
nmap -sC -sV -p$(nmap -p- -Pn 10.10.11.243 | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") 10.10.11.243
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-26 18:09 EDT
Nmap scan report for 10.10.11.243
Host is up (0.032s latency).

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp    open  http       nginx 1.18.0 (Ubuntu)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Error 401 Unauthorized
1339/tcp  open  http       nginx 1.18.0 (Ubuntu)
|_http-title: Index of /
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-ls: Volume /
|   maxfiles limit reached (10)
| SIZE    TIME               FILENAME
| -       06-Nov-2023 01:10  bin/
| -       06-Nov-2023 01:10  bin/X11/
| 963     17-Feb-2020 14:11  bin/NF
| 129576  27-Oct-2023 11:38  bin/VGAuthService
| 51632   07-Feb-2022 16:03  bin/%5B
| 35344   19-Oct-2022 14:52  bin/aa-enabled
| 35344   19-Oct-2022 14:52  bin/aa-exec
| 31248   19-Oct-2022 14:52  bin/aa-features-abi
| 14478   04-May-2023 11:14  bin/add-apt-repository
| 14712   21-Feb-2022 01:49  bin/addpart
|_
1883/tcp  open  mqtt
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     ActiveMQ/Advisory/MasterBroker: 
|_    ActiveMQ/Advisory/Consumer/Topic/#: 
5672/tcp  open  amqp?
|_amqp-info: ERROR: AQMP:handshake expected header (1) frame, but was 65
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GetRequest, HTTPOptions, RPCCheck, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
|     AMQP
|     AMQP
|     amqp:decode-error
|_    7Connection from client using unsupported AMQP attempted
8080/tcp  open  http       nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/plain).
|_http-open-proxy: Proxy might be redirecting requests
8161/tcp  open  http       Jetty 9.4.39.v20210325
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-server-header: Jetty(9.4.39.v20210325)
|_http-title: Error 401 Unauthorized
34169/tcp open  tcpwrapped
61613/tcp open  stomp      Apache ActiveMQ
| fingerprint-strings: 
|   HELP4STOMP: 
|     ERROR
|     content-type:text/plain
|     message:Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolException: Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolConverter.onStompCommand(ProtocolConverter.java:258)
|     org.apache.activemq.transport.stomp.StompTransportFilter.onCommand(StompTransportFilter.java:85)
|     org.apache.activemq.transport.TransportSupport.doConsume(TransportSupport.java:83)
|     org.apache.activemq.transport.tcp.TcpTransport.doRun(TcpTransport.java:233)
|     org.apache.activemq.transport.tcp.TcpTransport.run(TcpTransport.java:215)
|_    java.lang.Thread.run(Thread.java:750)
61614/tcp open  http       Jetty 9.4.39.v20210325
|_http-server-header: Jetty(9.4.39.v20210325)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title.
61616/tcp open  apachemq   ActiveMQ OpenWire transport 5.15.15
```

This box had an unusually large attack surface — ports everywhere. The key things that jumped out were:

- **Port 80** — nginx with HTTP Basic Auth, realm named `ActiveMQRealm`. That's the ActiveMQ web console sitting behind a reverse proxy.
- **Port 8161** — Jetty server, same `ActiveMQRealm` challenge. This is the native ActiveMQ admin console.
- **Port 1883** — MQTT broker. Nmap even subscribed and pulled some topic names, confirming it's ActiveMQ.
- **Port 61616** — OpenWire transport, and nmap fingerprinted it as **ActiveMQ 5.15.15**. That version is important.
- **Port 1339** — An open directory listing serving what looks like the filesystem root. Interesting but not immediately the path in.

---

## Foothold — Apache ActiveMQ RCE (CVE-2023-46604)

### Getting into the Admin Console

Port 80 prompted for credentials. ActiveMQ ships with a well-known default of `admin:admin`, so I tried that first — it worked straight away.

Inside the admin panel, the version was confirmed as **5.15.15**. A quick search on that version turned up [CVE-2023-46604](https://attackerkb.com/topics/IHsgZDE3tS/cve-2023-46604/rapid7-analysis) — a critical remote code execution vulnerability in Apache ActiveMQ's OpenWire protocol. It was patched in 5.15.16, so this box is running a deliberately vulnerable version.

### Exploiting CVE-2023-46604

CVE-2023-46604 allows an unauthenticated attacker to execute arbitrary shell commands by abusing a ClassInfo header in the OpenWire protocol. The exploit sends a crafted message that causes the broker to fetch and execute a remote ClassPathXmlApplicationContext — essentially pointing ActiveMQ at an attacker-controlled XML file containing a reverse shell payload.

I set up the exploit with an XML-escaped reverse shell pointing back to my machine, triggered it against port 61616, and caught a shell as the `activemq` service account. User flag was in the home directory.

---

## Privilege Escalation — activemq → root

### sudo -l

First thing I checked after landing was `sudo -l`:

```
activemq@broker:~$ sudo -l
sudo -l
Matching Defaults entries for activemq on broker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User activemq may run the following commands on broker:
    (ALL : ALL) NOPASSWD: /usr/sbin/nginx
```

`activemq` can run nginx as root with no password. GTFOBins didn't have anything for nginx directly, but a search turned up this gist:

> [nginx-sudo-privilege-escalation](https://gist.github.com/DylanGrl/ab497e2f01c7d672a80ab9561a903406)

### nginx Sudo PrivEsc

The idea behind the technique is that nginx can be configured to run with a custom config file — and since we're running it as root via sudo, we control what that config does. The gist walks through writing a minimal nginx config that:

1. Runs nginx as `root`
2. Configures it to serve files from `/` (or write to sensitive paths)

One common variant is using nginx's `dav_methods PUT` to write an SSH key or a cron job, or configuring it to expose `/etc/shadow`. Since we control the config and nginx runs as root, the privilege boundary disappears entirely.

Followed the gist steps and escalated to root.

---

## Root

```
# id
uid=0(root) gid=0(root) groups=0(root)
```
