# IP

192.168.54.237
# NMAP

```
???(kali?kali)-[~]
??$ nmap -p22,80 -sC -sV 192.168.54.237 -oA Marshalled
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-05 17:13 UTC
Nmap scan report for 192.168.54.237
Host is up (0.0010s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.68 seconds
```

# Port 80

only small site, can't seem to find anything with dirb

used ffuf to subdomain enumerate as shown in [[exploit-notes/web/api/index|index]]

```
 ffuf -u http://marshalled.pg -c  -w /usr/share/wordlists/amass/subdomains-top1mil-5000.txt  -H "Host: FUZZ.marshalled.pg" -mc 200 -o subdomains
```

http://monitoring.marshalled.pg/dashboard FOUND

this is a sort of monitoring website showing lots of sys info
(had to login with admin : admin)

The remember me option returns a token which is serialized and urlencoded YAML:

Had to use [cyberchef](https://gchq.github.io/CyberChef/) to decode

![[Pasted image 20241005140155.png]]

Got:

```
--- !ruby/object:User
concise_attributes:
- !ruby/object:ActiveModel::Attribute::FromDatabase
  name: id
  value_before_type_cast: 104
- !ruby/object:ActiveModel::Attribute::FromDatabase
  name: username
  value_before_type_cast: admin
- !ruby/object:ActiveModel::Attribute::FromDatabase
  name: password_digest
  value_before_type_cast: "$2a$12$ogjC9QG2BTiHN7NYilSN.SFejO"
- !ruby/object:ActiveModel::Attribute::FromDatabase
  name: created_at
  value_before_type_cast: '2022-09-13 20:06:13.809506'
- !ruby/object:ActiveModel::Attribute::FromDatabase
  name: updated_at
  value_before_type_cast: '2022-09-13 20:06:13.809506'
new_record: false
active_record_yaml_version: 2
```


Read up on:

https://blog.stratumsecurity.com/2021/06/09/blind-remote-code-execution-through-yaml-deserialization/

Got a reverseshell using same YAML from post

