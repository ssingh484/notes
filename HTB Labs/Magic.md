# NMAP TCP

TARGET: 10.129.19.124
ME: 10.10.15.237

```nmap-tcp
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-09 20:01 -0500
Nmap scan report for 10.129.19.124
Host is up (0.037s latency).
Not shown: 198 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
|_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Magic Portfolio
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Port 80

Simple image hosting site that logs in via basic SQLi

username -> ' or 1=1;-- -
password -> blank

Get a file upload page

Used revshell to generate a php revshell

Was able to bypass file check filters by adding jpeg extension and jpeg magic bytes using [[image-file-reparing]] and then modifying clobbered comment to be valid again starting with `<?php`

To get this upload to execute, used burp proxy to modify filename to test.php.jpeg which still allowed it to be LFI'd as a php file

Got shell

# www-data

Start with ![[upgrade-to-fully-interactive-tty]]

Found in /var/www/Magic/db.php5 sql creds:

```
ww-data@ubuntu:/var/www/Magic$ cat db.php5
<?php
class Database
{
    private static $dbName = 'Magic' ;
    private static $dbHost = 'localhost' ;
    private static $dbUsername = 'theseus';
    private static $dbUserPassword = 'iamkingtheseus';

    private static $cont  = null;

    public function __construct() {
        die('Init function is not allowed');
    }

    public static function connect()
    {
        // One connection through whole application
        if ( null == self::$cont )
        {
            try
            {
                self::$cont =  new PDO( "mysql:host=".self::$dbHost.";"."dbname=".self::$dbName, self::$dbUsername, self::$dbUserPassword);
            }
            catch(PDOException $e)
            {
                die($e->getMessage());
            }
```

No mysql on machine but a mysqldump binary which allowed dumping the database "Magic"

This gave another set of creds:

```
LOCK TABLES `login` WRITE;
/*!40000 ALTER TABLE `login` DISABLE KEYS */;
INSERT INTO `login` VALUES (1,'admin','Th3s3usW4sK1ng');
```

Used this to su to theseus

# Theseus

Ran linpeas as theseus
Found /bin/sysinfo is an unknown suid binary

Using ghidra/strings in general shows that it runs lshw with some options to do the hardware info part

Since this is a relative path, able to add a custom binary to /tmp and add /tmp to path to get that to be executed by the SUID binary

Made a simple payload.c and compiled by gcc on machine in tmp folder

Exported pre-pended PATH env var

Ran /bin/sysinfo

Got shell as root

==DONE==
