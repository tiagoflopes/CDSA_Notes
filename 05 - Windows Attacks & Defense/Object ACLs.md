## Description

In AD, Access Control Lists (ACLs) are tables, or simple lists, that define the trustees who have access to a specific object and their access type.
Each Access Control Entry (ACE) defines the trustee and the type of access they have.
In real-life AD environments, we will often encounter cases such as:
- All Domain users added as Administrators to all Servers.
- Everyone can modify all objects.
- All Domain users have access to the computer's extended properties containing the LAPS passwords.

## Attack

To identify abusable ACLs, we will use [BloodHound](https://github.com/BloodHoundAD/BloodHound) to graph the relationships between the objects and [SharpHound](https://github.com/BloodHoundAD/SharpHound) to scan the environment.
```powershell
.\SharpHound.exe -c All
```

The output zip file can be visualized in BloodHound. Instead of looking for every misconfigured ACL in the environment, we will focus on Bob. Bob has _GenericAll_ permissions over Anni and Server01.
From there, you can do whatever.

## Prevention

To prevent this:
- Begin continuous assessment to detect if this is a problem in the AD environment.
- Educate employees with high privileges to avoid doing this.
- Automate as much as possible from the access management process and only assign privileged access to administrative accounts.

## Detection

There are several ways to detect if AD objects are modified. However, the events generated from that are incomplete.
Those events are still useful, because we can tell if a non-privileged user performs privileged actions on another user. 

## Honeypot

There are 2 ways to approach this:
1. Assign relatively high ACLs to a user account used as a honeypot. For example, we have a user whose fake credentials are exposed in the description field. Having ACLs assigned to the account may provoke an adversary to attempt and verify if the account's exposed password is valid as it holds high potential.
2. Have an account that everyone or many users can modify. Any changes on this honeypot are malicious since there is no valid reason for anyone to perform any actions on it.
