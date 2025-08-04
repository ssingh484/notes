# NMAP - TCP

```nmap-oneliner
nmap -sC -sV -p$(nmap -p- -Pn 10.10.10.152 | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") 10.10.10.152
```

```target
10.10.10.152
```

FTP anonymous login led to user.txt already

Also found backup file for PRTG Network Monitor that had a password which was exposed due to an issue in a version that was disclosed online

password was rotated by changing 2018 to 2019 and logging in with that allowed access into PRTG

the version had an RCE exploit post auth

Used this to add a user to local admins and logged in as it via evil-winrm

==DONE==