# IP - 192.168.53.108

``` NMAP-Splodge
??$ nmap -sC -sV -p- 192.168.53.108 -oA Splodge -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-28 19:51 UTC
Nmap scan report for 192.168.53.108
Host is up (0.00076s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 43:77:53:46:f8:78:c6:cb:c4:c6:b5:f2:61:2a:64:13 (RSA)
|   256 a5:b4:45:1f:eb:10:ac:1d:fc:64:de:4b:87:ed:7d:ca (ECDSA)
|_  256 44:7c:68:45:db:3d:45:9b:ec:7c:0d:94:6b:9e:31:f5 (ED25519)
80/tcp   open  http       nginx 1.16.1
| http-git: 
|   192.168.53.108:80/.git/
|     Git repository found!
|     .git/config matched patterns 'user'
|     .gitignore matched patterns 'bug' 'key'
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|     Last commit message: initial commit 
|_    Project type: node.js application (guessed from .gitignore)
|_http-title: 403 Forbidden
|_http-server-header: nginx/1.16.1
1337/tcp open  http       nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-title: Commando
5432/tcp open  postgresql PostgreSQL DB 12.3 - 12.4
8080/tcp open  http       nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-title: Splodge | Home

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 174.16 seconds
```

# Port 80

Found a git repository being served up - able to get .git and .gitignore

The .gitignore file is:

```gitignore-splodge
/node_modules
/public/hot
/public/storage
/storage/*.key
/vendor
/.idea
/.vagrant
Homestead.json
Homestead.yaml
npm-debug.log
yarn-error.log
.env
```

Can use https://github.com/internetwache/GitTools to dump the entire .git folder in terms of objects and refs

Using this, can have the .git folder locally used for a git checkout to rebuild the state of the original git commited folder

Did `git checkout -- .` in parent folder of the .git dumped above

Got:

```
???(kali?kali)-[~/dump]
??$ ls
app        composer.json  database     resources   storage
artisan    composer.lock  phpunit.xml  routes
bootstrap  config         public       server.php
```

Found in database seed a password

Used that to login into PHP site on port 80 in Admin

There's a usage of preg_replace in PHP which we can modify

Modified it to spawn a shell by replacing a with a system() calling a bash revshell

[PREG_REPLACE_PHP](https://captainnoob.medium.com/command-execution-preg-replace-php-function-exploit-62d6f746bda4)

