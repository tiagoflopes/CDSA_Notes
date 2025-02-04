## Description

Credentials exposed in shares are the most encountered misconfiguration in AD to date. We often find them within scripts and configuration files.
The main difference between the storage of credentials on shares and machines is that the former poses a significantly higher risk, as it may be accessible by every user.

## Attack

First, we need to identify the shares in the domain. We will use PowerView's [Invoke-ShareFinder](https://github.com/darkoperator/Veil-PowerView/blob/master/PowerView/functions/Invoke-ShareFinder.ps1). The output contains a list of non-default shares that the current user account has at least read access to.
```powershell
Invoke-ShareFinder -domain eagle.local -ExcludeStandard -CheckShareAccess
```

A few automated tools exist, such as [SauronEye](https://github.com/vivami/SauronEye), which can parse a collection of files and pick up matching words. However, because there are few shares in the playground, we will take a more manual approach (Living Off the Land) and use the built-in command _findstr_ for this attack. Attractive targets for this search would be .bat, .cmd, .ps1, .conf, .config and .ini files.
```powershell
cd \\Server01.eagle.local\dev$
findstr /m /s /i "pass" *.bat
findstr /m /s /i "pass" *.cmd
findstr /m /s /i "pass" *.ini
findstr /m /s /i "pass" *.config
```
Instead of "pass" we can also search for "pw" and the NETBIOS name of the domain ("eagle").
By removing the /m argument, we get the exact line where it appears.

Note that running _findstr_ with the same arguments is picked up by Windows Defender as suspicious behavior.

## Prevention

The best practice is to lock down every share in the domain so there are no loose permissions.
There is no way to prevent what users leave behind in scripts or other exposed files, so performing regular scans to identify any new open shares or credentials exposed in older ones is necessary.

## Detection

Understanding and analyzing users' behavior is the best detection technique for abusing discovered credentials in shares. An event with ID 4624, 4625 or 4768 representing a logon for the Administrator account performed from a weird source IP address is suspicious.
Another detection technique is discovering the one-to-many connections. It would be abnormal for a workstation to connect to 100s or even 1000s of other devices simusltaneously.

## Honeypot

Here is another excellent reason for leaving a honeypot user in AD environments, like in [[GPP Passwords#Honeypot]].
