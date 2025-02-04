## Description

In AD, a Service Principal Name (SPN) is a unique service instance identifier. Kerberos uses SPNs for authentication to associate a service instance with a service logon account. When a Kerberos TGS service ticket is asked for, it gets encrypted with the service's NTLM password hash.
**Kerberoasting** is a post-exploitation attack that attempts to exploit this behavior by obtaining a ticket and performing offline password cracking to open the ticket. Its success depends on the strength of the password.
Another factor that has some impact is the encryption algorithm used when the ticker is created, which will likely be either _AES_ (the slower to crack), _RC4_ (recommended disabling) or _DES_ (found in legacy apps, the worst).
However, attackers can force a downgrade back to RC4.

## Attack Path

To obtain crackable tickets, we can use Rubeus.

![[kerberoasting_rubeus.webp]]

Then we can move the file with the tickets to the attacker's machine for cracking.
We can use hashcat with the hash-mode 13100 for a _Kerberoastable TGS_.
```shell-session
hashcat -m 13100 -a 0 spn.txt passwords.txt --outfile="cracked.txt"
```

Once it finishes, we can read the output file to see the password in plain text.

Alternatively, the captured TGS hashes can be cracked with John The Ripper
```shell-session
sudo john spn.txt --fork=4 --format=krb5tgs --wordlist=passwords.txt --pot=results.pot
```

## Prevention

The success of this attack relies on the strength of the service account's password.
Limit the number of accounts with SPNs and disable those no longer used/needed. Ensure they have strong passwords (100+ characters).
There is also Group Managed Service Accounts (GMSA), which is a particular type of a service account that AD manages. This is the perfect solution because these accounts are bound to a specific server and no user can use them anywhere else. However, not all applications support these account, as they work mainly with Microsoft services (such as IIS and SQL). Utilize them everywhere possible.
When in doubt, do not assign SPNs to accounts that do not need them. Ensure regular clean-up of SPNs set to no longer valid services/servers.

## Detection

When a TGS ticket is requested, an event log with ID 4769 is generated, however, AD also generated the same event ID whenever a user attempts to connect to a service. Relying on it alone is virtually impossible to use as a detection method. If we happen to be in an environment where only AES tickets are generated, then it would be an excellent indicator.
This is the logged event when we demonstrated the attack.
![[kerberoasting_detection.webp]]

Despite the general volume of this event being quite heavy, it may be useful, since when Rubeus is ran, it extracts a ticket for each user in the environment with an SPN registered.
This means that we can match volume of events by source IP address or requester name within a short time.

## Honeypot

^ebf986

A honeypot user is a perfect detection option to configure. This must be a user with no real use/need, so that no service tickets are generated regularly. In this case, any attempts to generate a service ticket for this account is likely malicious and worth inspecting. There are a few things to ensure when using this account:
- The account must be a relatively old user (advanced threat actors won't request tickets for new accounts because they likely have strong passwords and the possibility of being a honeypot user).
- The password should not have been changed recently. The password must be strong enough that the threat agents cannot crack it.
- The account must have some privileges assigned to it, making it interesting to obtain a ticket from.
- The account must have an SPN registered, which appears legit. IIS and SQL account are good options because they are prevalent.
