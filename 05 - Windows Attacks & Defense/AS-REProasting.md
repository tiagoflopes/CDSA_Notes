## Description

The AS-REProasting attack is similar to the [[Kerberoasting]] attack; we can obtain crackable hashes for user accounts that have the property _Do not require Kerberos preauthentication_ enabled.

## Attack

We will use Rubeus again, but using the asreproast action.

![[asreproasting_rubeus.webp]]

To crack it, we need to edit the hash, by adding "23\$" after "krb5asrep\$". Now we can use hashcat with the hash-mode 18200 for AS-REPRoastable hashes.
```shell-session
sudo hashcat -m 18200 -a 0 asrep.txt passwords.txt --outfile asrepcrack.txt --force
```

## Prevention

The success of this attack depends on the strength of the password of users with _Do not require Kerberos preauthentication_ configured
First and foremost, we should only use this property if needed. And for users requiring this configured, we should assign a separate password policy.

## Detection

When we execute Rubeus, an Event with ID 4768 was generated, signaling that a Kerberos Authentication ticket was generated. (Different from Kerberoasting that generates a Kerberos Service ticket, event ID 4769).
AD generated this event for every user that authenticates with Kerberos to any device. It is possible to scrutinize the particular VLAN and alert on anything outside it.

## Honeypot

This is an excellent detection option to configure in AD environments.
See [[Kerberoasting#Honeypot]] for some needed considerations.
