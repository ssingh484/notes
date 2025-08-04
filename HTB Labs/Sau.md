# NMAP

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-20 14:19 CST
Nmap scan report for 10.10.11.224
Host is up (0.0093s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    filtered http
8338/tcp  filtered unknown
55555/tcp open     unknown
```
# Port 55555

Seems to be a webserver running Request Baskets version 1.2.1

Searchsploit shows nothing

Found SSRF vulnerability online which shows how to make the Requests-Basket instance make requests to any other network resource

This allows access to the port 80 internally accessible as seen in NMAP scan

This lets access to 127.0.0.1:80 which is running Maltrail v0.53

## Maltrail

Found an RCE exploit online so can use above to proxy the exploit onto the Maltrail service via a basket

Used that to get a reverse shell to nc listener

# User: Puma

Found user.txt

Running `sudo -l` shows:

```
$ sudo -l
sudo -l
Matching Defaults entries for puma on sau:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User puma may run the following commands on sau:
    (ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service
```

Systemctl status opens a text editor to show the status but this can lead to a shell (as root due to sudo) by doing `!/bin/bash` to spawn the bin bash process

Got root

# Root

Got root.txt

==DONE==

