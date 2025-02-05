## Description

Kerberos Delegation enables an application to access resources hosted on a different server (instead of giving the service account running other service).
We can configure 3 types of delegation in AD:
- Unconstrained Delegation - Allows an account to delegate to any service.
- Constrained Delegation - A user account will have its properties configured to specify which service(s) they can delegate.
- Resource-based Delegation - The configuration is within the computer object to whom delegation occurs. (Rare)

Any type of delegation is a possible security risk and we should avoid it unless necessary.

## Attack

We will showcase the abuse of constrained delegation.
When the account is trusted for delegation, the account sends a request to the KDC stating "gimme a Kerberos ticket for user YYYY because I am trusted to delegate this user to service ZZZZ", and a Kerberos ticket is generated for user YYYY. It's possible to delegate to another service, even if not configured in the user properties.
To demonstrate the attack, we assume that the user _web\_service_ is trusted for delegation and has been compromised.
First, we will use the `Get-NetUser` function from [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) to enumerate user accounts that are trusted for constrained delegation in the domain (use the PowerView-main.ps1).
```powershell
Get-NetUser -TrustedToAuth
```
The account is allowed to delegate to DC1 on HTTP service. The HTTP service provides the ability to execute PowerShell Remoting.

Now we need the NTLM hash of the web_service's password.
```powershell
.\Rubeus.exe hash /password:Slavi123
```

Then we can use Rubeus to get a ticket for the Administrator account.
```powershell
.\Rubeus.exe s4u /user:webservice /rc4:FCDC65703DD2B0BD789977F1F3EEAECF /domain:eagle.local /impersonateuser:Administrator /msdsspn:"http/dc1" /dc:dc1.eagle.local /ptt
```

With the ticket being available, we can connect to the DC impersonating the account Administrator.
```powershell
Enter-PSSession dc1
```

If this fails, do `klist purge`, obtain new tickets, and try again by rebooting the machine.

## Prevention

There are several protection mechanisms that aren't enabled by default. Here are 2 distinct ways to prevent a ticket from being issued for a user via delegation:
- Configure the property _Account is sensitive and cannot be delegated_ for all privileged users.
- Add privileged users to the Protected Users group. Do this if you understand the implications of being in that group.

## Detection

Correlating users' behavior is the best technique. See if any event with ID 4624 or 4625 generated is suspicious, based on the account and the source IP.
In some occasions, a successful logon attempt with a delegated ticket will contain information about the ticket's issuer under the _Transited Services_ attribute in the logs.
Service For User (S4U) is a Microsoft extension to the Kerberos protocol that allows an application service to obtain a Kerberos service ticket on behalf of a user.
