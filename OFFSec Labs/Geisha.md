# Info

IP:
```
192.168.52.82
```

My IP:
```
192.168.49.52
```

# NMAP

```
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
3389/tcp open  ms-wbt-server
7080/tcp open  empowerid
7125/tcp open  unknown
8088/tcp open  radan-http
9198/tcp open  unknown
```

```
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           vsftpd 3.0.3
22/tcp   open  ssh           OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 1b:f2:5d:cd:89:13:f2:49:00:9f:8c:f9:eb:a2:a2:0c (RSA)
|   256 31:5a:65:2e:ab:0f:59:ab:e0:33:3a:0c:fc:49:e0:5f (ECDSA)
|_  256 c6:a7:35:14:96:13:f8:de:1e:e2:bc:e7:c7:66:8b:ac (ED25519)
80/tcp   open  http          Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Geisha
3389/tcp open  http          nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Seppuku
7080/tcp open  ssl/empowerid LiteSpeed
|_http-server-header: LiteSpeed
|_ssl-date: TLS randomness does not represent time
|_http-title: Did not follow redirect to https://192.168.52.82:7080/
| ssl-cert: Subject: commonName=geisha/organizationName=webadmin/countryName=US
| Not valid before: 2020-05-09T14:01:34
|_Not valid after:  2022-05-09T14:01:34
| tls-alpn: 
|   h2
|   spdy/3
|   spdy/2
|_  http/1.1
7125/tcp open  http          nginx 1.17.10
|_http-title: Geisha
|_http-server-header: nginx/1.17.10
8088/tcp open  http          LiteSpeed httpd
|_http-title: Geisha
|_http-server-header: LiteSpeed
9198/tcp open  http          SimpleHTTPServer 0.6 (Python 2.7.16)
|_http-title: Geisha
|_http-server-header: SimpleHTTP/0.6 Python/2.7.16
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```