## Launch SMB Server

```bash
impacket-smbserver share . -smb2support -username user -password pass
```

### Access from Remote Machine

```bash
net use \\<local-ip>\share /u:user pass
```

### Transfer Files

```bash
# Remote to Local
cp .\example.txt \\<local-ip>\share\example.txt
```
