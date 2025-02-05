## Description

DCSync is an attack that threat agents utilize to impersonate a Domain Controller and perform replication with a targeted Domain Controller to extract password hashes from AD.
This can be performed by anyone with the necessary permissions:
- Replicating Directory Changes
- Replicating Directory Changes All

## Attack

We will utilize the user Rocky:Slavi123 to showcase the attack. He has the needed permissions.
First, start a new command shell as Rocky: `runas /user:eagle\rocky cmd.exe`.
Then, run Mimikatz: `lsadump::dcsync /domain:eagle.local /user:Administrator`. We can also use `/all` to run it on all users.
We can then perform pass-the-hash with the obtained hash and authenticate against any Domain Controller.

## Prevention

Replications happen between Domain Controllers all the time. Therefore, preventing DCSync is not an option.
The only prevention technique against this attack is using solutions such as the [RPC Firewall](https://github.com/zeronetworks/rpcfirewall) that can block or allow specific RPC calls with robust granularity.
For example, using it, we can only allow replications from Domain Controllers.

## Detection

It's easy to detect since a replication generates an event with ID 4662.
We can pick up abnormal activity by checking whether the initiator account is a Domain Controller.
This events are generated all the time however, so to avoid false positives ensure that:
- Either the property `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` or `1131f6ad-9c07-11d1-f79f-00c04fc2dcd2` is [present in the event](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/1522b774-6464-41a3-87a5-1e5633c3fbbb).
- Whitelisting systems/accounts with a valid business reason for replicating, such as Azure AD Connect.
