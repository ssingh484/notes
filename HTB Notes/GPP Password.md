
A feature was previously available that allowed the deployment of custom local administrator accounts on a group of machines via Group Policy Preferences (GPP). However, this method had significant security flaws. Firstly, the Group Policy Objects (GPOs), stored as XML files in SYSVOL, could be accessed by any domain user. Secondly, the passwords within these GPPs, encrypted with AES256 using a publicly documented default key, could be decrypted by any authenticated user. This posed a serious risk, as it could allow users to gain elevated privileges.

To mitigate this risk, a function was developed to scan for locally cached GPP files containing a “cpassword” field that is not empty. Upon finding such a file, the function decrypts the password and returns a custom PowerShell object. This object includes details about the GPP and the file’s location, aiding in the identification and remediation of this security vulnerability.

Search in C:\ProgramData\Microsoft\Group Policy\history or in C:\Documents and Settings\All Users\Application Data\Microsoft\Group Policy\history (previous to W Vista) for these files:

Groups.xml
Services.xml
Scheduledtasks.xml
DataSources.xml
Printers.xml
Drives.xml
To decrypt the cPassword:

# To decrypt these passwords you can decrypt it using

gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw

Using crackmapexec to get the passwords:

crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin

Using crackmapexec or netexec to hunt for this password:

netexec smb 10.129.29.233 -u '<USER>' -p '<PASS>' -M gpp_password