## Description

SYSVOL is a network share on all Domain Controllers, containing logon scripts, group policy data and other required domain-wide data. AD stores all group policies in \\\\\<DOMAIN>\SYSVOL\\\<DOMAIN>\Policies\.
Group Policy Preferences (GPP) introduced the ability to store and use credentials in several scenarios, all of which AD stores in the policies directory.
There might be scheduled tasks and scripts executed under a particular user and contain the username and an encrypted version of the password in XML policy files. The encryption key was released on Microsoft Docs, meaning anyone can decrypt the credentials.
For reference, this is an example XML file containing an encrypted password:
![[GPPcPass.webp]]

## Attack

To abuse that, we will use the [Get-GPPPassword](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1) function from PowerSploit, which automatically parses XML files in the policies folder, picking up those with the cpassword property and decrypting them once detected.
```powershell
Import-Module .\Get-GPPPassword.ps1
Get-GPPPassword
```

## Prevention

Microsoft released a patch to prevent caching credentials in GPP. It is still highly recommended to continuously assess and review the environment to ensure that no credentials are exposed.
If an organization built its AD environment before 2014, it is likely that its credentials are still cached because the patch doesn't clear existing stored credentials.

## Detection

There are 2 detection techniques:
- As demonstrated by the tool, it parses all of the XML files in the policies folder. Then we can generate an event whenever a user reads a dummy XML file that isn't associated with any GPO (Event ID 4663).
- Logon attempts of the user whose credentials are exposed (Event IDs 4624, 4625 and 4768). In case of a service account, the IP address may help identifying the device and discover if it is legit or suspicious behavior.

## Honeypot

Another excellent opportunity for setting up a trap: we can use a semi-privileged user with a wrong password. Service accounts provide a more realistic opportunity because:
- The password is usually expected ti be old.
- It is easy to ensure that the last password change is older than when the GPP XML file was last modified.
- Schedule the user to perform any dummy task to ensure that there are recent logon attempts.
So, we are expecting failed logon attempts (Event IDs 4625, 4771 and 4776).
