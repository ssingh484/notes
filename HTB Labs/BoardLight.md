
# NMAP

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

# Port 80

Found a simple website

Used [[exploit-notes/web/api/index|subdomain_enum]] to subdomain enumerate via ffuf:

```
ffuf -u http://board.htb -c  -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-11000.txt  -H "Host: FUZZ.board.htb" -mc 200 -o subdomains -fs 15949
```

Found crm.board.htb and added it to hosts file

crm.board.htb is Dolibarr CRM/ERP version 17.0.0

Found [exploit](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253?tab=readme-ov-file) for RCE after authentication (admin/admin)

As www-data:

## www-data

Found conf.php containing some credentials and the low-level user is larissa

```
$dolibarr_main_db_user='dolibarrowner';
$dolibarr_main_db_pass='serverfun2$2023!!';
```

Able to SSH as larissa with same credentials

## Larissa

Used linpeass and found enlightenment binaries with SUID bit

found the version 0.25.3 which has an exploit on exploit-db for privesc

popped root with it
## Root
