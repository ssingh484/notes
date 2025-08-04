
# NMAP

### Open Ports:
```
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49676/tcp open  unknown
49688/tcp open  unknown
49693/tcp open  unknown
49712/tcp open  unknown
```

# SMB

Found a null session smb share on the box called support-tools

Downloaded all the files which are all just executables and executables in zips

UserInfo.exe has creds as strings in it (found via Ghidra)

Found LDAP creds for a user:

`support\ldap : armando`

