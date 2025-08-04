| off**SMB Version** | **Supported**                       | **Features**                                                           |
| ------------------ | ----------------------------------- | ---------------------------------------------------------------------- |
| CIFS               | Windows NT 4.0                      | Communication via NetBIOS interface                                    |
| SMB 1.0            | Windows 2000                        | Direct connection via TCP                                              |
| SMB 2.0            | Windows Vista, Windows Server 2008  | Performance upgrades, improved message signing, caching feature        |
| SMB 2.1            | Windows 7, Windows Server 2008 R2   | Locking mechanisms                                                     |
| SMB 3.0            | Windows 8, Windows Server 2012      | Multichannel connections, end-to-end encryption, remote storage access |
| SMB 3.0.2          | Windows 8.1, Windows Server 2012 R2 |                                                                        |
| SMB 3.1.1          | Windows 10, Windows Server 2016     | Integrity checking, AES-128 encryption                                 |

Connecting with null session (-N) and displaying all shares (-L):
```smbclient-null
smbclient -N -L //<target-ip>
```

SMBMap on host:
```smbmap-enum
smbmap -H <target-ip>
```

```smbclient-share
smbclient -N \\\\<target-ip>\\dev 
```

```nmap-smb-scripts
nmap --script=smb-* $TARGET -Pn 
```

```smbclient-anonymous-user
smbclient -U ''  -L \\\\<IP_ADDRESS>\\
```
