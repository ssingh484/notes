
**TARGET** 10.129.1.185
**ME** 10.10.15.62
# NMAP TCP

```

```


# PORT 80

Found a basic ASCII art website

Found a robots.txt

```
User-agent: *
Disallow: /webservices/tar/tar/source/
Disallow: /webservices/monstra-3.0.4/
Disallow: /webservices/easy-file-uploader/
Disallow: /webservices/developmental/
Disallow: /webservices/phpmyadmin/
```


Only http://10.129.1.185/webservices/monstra-3.0.4/ yields a 200

Seems to be a default monstra website

Default creds admin admin let us log in

Rabbit hole found

Also found webservices/wp is a barebones wordpress site with vulnerability due to plugin [gwolle-gb](https://www.exploit-db.com/exploits/38861)

Used that and php revshell to get shell as www-data

# www-data

![[upgrade-to-fully-interactive-tty]]

sudo -l shows I can exec tar as onuma

it is a GTFOBin letting me spawn /bin/sh as onuma

# onuma

Found user.txt

Found a shadow_bkp file but it is just a symlink to /dev/null

Found that a sql server is locally available on port 3306

```
-rwxr-xr-x 1 root root 2963 Jan 21  2021 /var/www/html/webservices/wp/wp-config.php                                                                                                                                                         
define('DB_NAME', 'wp');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'w0rdpr3$$d@t@b@$3@cc3$$');
define('DB_HOST', 'localhost');
```
That was a dead end

Used PSPY from [[Monitor Running Processes PrivEsc]] to find that this sript /usr/sbin/backuperer runs once in a few minutes as root and can let us read files in its error log by exploiting the archive location which is 


