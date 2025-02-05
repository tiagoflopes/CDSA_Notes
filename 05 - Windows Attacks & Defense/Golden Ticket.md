## Description

The Kerberos Golden Ticket is an attack in which threat agents can create/generate tickets for any user in the Domain, therefore effectively acting as a Domain Controller.
When a Domain is created, the unique user account krbtgt is created. It is a disabled account that cannot be deleted, renames nor enabled. The KDC service will use its password to derive a key with which it signs all Kerberos tickets.
Any user possessing the password's hash of krbtgt can create valid Kerberos TGTs.
The attack allows us to escalate rights from any child domain to the parent in the same forest. It occurs after an adversary has gained Domain Admin (or similar) privileges.

## Attack

To perform it, we will use Mimikatz with the arguments:
- domain - The domain's name.
- sid - The domain's SID value.
- rc4 - The password's hash of krbtgt.
- user - The username for which the ticket will be issued (Windows 2019 blocks tickets to nonexistent users).
- id - Relative ID (last part of SID) for the user for whom the ticket will be issued.
- renewmax - The maximum number of days the ticket can be renewed.
- endin - End-of-life for the ticket.

First we need to get the NTLM hash of the user krbtgt, using DCSync: db0d0630064747072a7da3f7c3b4069e.
Then we need the SID value of the Domain, using Get-DomainSID from PowerView: S-1-5-21-1518138621-4282902758-752445584.
Now we got all the information needed.
```cmd-session
kerberos::golden /domain:eagle.local /sid:S-1-5-21-1518138621-4282902758-752445584 /rc4:db0d0630064747072a7da3f7c3b4069e /user:Administrator /id:500 /renewmax:7 /endin:8 /ptt
```
The argument `/ptt` makes Mimikatz pass the ticket into the current session.

If then we run `klist` we'll see the ticket cached.
To verify it, list the contents of the share C$ of DC1 in the current session.

## Prevention

It is hard to prevent it because the KDC generates valid tickets the same way.
There are some thing we can and should do:
- Block privileged users from authenticating to any device.
- Periodically reset the password of the krbtgt account. Use [KrbtgtKeys.ps1](https://github.com/microsoft/New-KrbtgtKeys.ps1).
- Enforce SIDHistory filtering between the domains in forests to prevent the escalation from a child domain to a parent domain. This can result in potential issues in migrating domains.

## Detection

Correlating users' behavior is the best technique. See if any event with ID 4624 or 4625 generated is suspicious, based on the account and the source IP.

## Note

If an AD forest has been compromised, we need to reset all users' passwords and revoke all certificates, and for krbtgt. we must reset its password twice in every domain, with at least 10 hours apart from each reset.
