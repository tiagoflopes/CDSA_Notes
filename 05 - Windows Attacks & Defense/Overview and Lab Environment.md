We will dive into several different attacks. The objective of each it to:
1. Describe it.
2. Provide a walkthrough of how we can carry out the attack.
3. Provide preventive techniques and compensating controls.
4. Discuss detection capabilities.
5. Discuss the 'honeypot' approach of detecting the attack, if applicable.

The following is a complete list of all attacks described in this module:
- `Kerberoasting`
- `AS-REProasting`
- `GPP Passwords`
- `Misconfigured GPO Permissions (or GPO-deployed files)`
- `Credentials in Network Shares`
- `Credentials in User Attributes`
- `DCSync`
- `Kerberos Golden Ticket`
- `Kerberos Constrained Delegation attack`
- `Print Spooler & NTLM Relaying`
- `Coercing attacks & Kerberos Unconstrained Delegation`
- `Object ACLs`
- `PKI Misconfigurations` - `ESC1`
- `PKI Misconfigurations` - `ESC8` (`Coercing` + `Certificates`)

## Lab Environment

The assumption is that an attacker has already gained remote code execution on the Windows 10 (WS001) machine. The user is Bob, a regular user in AD with no special permissions assigned.
The environment consists of the following:
- DC1 - 172.16.18.3
- DC2 - 172.16.18.4
- Server01 - 172.16.18.10
- PKI - 172.16.18.15
- WS001 - DHCP or 172.16.18.25
- Kali Linux - DHCP or 172.16.18.20

## Connection

##### WS001 via RDP
```shell-session
xfreerdp /u:eagle\\bob /p:Slavi123 /v:TARGET_IP /dynamic-resolution
```

##### Kali via SSH
```shell-session
ssh kali@TARGET_IP
```
Password: kali

Sometimes it will be recommended to rdp into kali.
```shell-session
xfreerdp /v:TARGET_IP /u:kali /p:kali /dynamic-resolution
```

## Moving files between WS001 and the Attacking Machine

There is a shared folder on WS001, accessible via SMB.
On kali, use smbclient.
```shell-session
smbclient \\\\TARGET_IP\\Share -U eagle/administrator%Slavi123
```
