## Description

Objects in AD have a plethora of different properties. When administrators create accounts, they fill in those properties.
A common practice in the past was to add the user's password in the _Description_ of _Info_ properties, which may be available for all domain users.

## Attack

A simple PowerShell script can query the entire domain by looking for specific terms/strings in those fields.
```powershell
Function SearchUserClearTextInformation
{
    Param (
        [Parameter(Mandatory=$true)]
        [Array] $Terms,

        [Parameter(Mandatory=$false)]
        [String] $Domain
    )

    if ([string]::IsNullOrEmpty($Domain)) {
        $dc = (Get-ADDomain).RIDMaster
    } else {
        $dc = (Get-ADDomain $Domain).RIDMaster
    }

    $list = @()

    foreach ($t in $Terms)
    {
        $list += "(`$_.Description -like `"*$t*`")"
        $list += "(`$_.Info -like `"*$t*`")"
    }

    Get-ADUser -Filter * -Server $dc -Properties Enabled,Description,Info,PasswordNeverExpires,PasswordLastSet |
        Where { Invoke-Expression ($list -join ' -OR ') } | 
        Select SamAccountName,Enabled,Description,Info,PasswordNeverExpires,PasswordLastSet | 
        fl
}
```

Then we can run the function with terms such as "pass".
```powershell
SearchUserClearTextInformation -Terms "pass"
```

## Prevention

We have many options to prevent this attack/misconfiguration:
- Perform continuous assessments to detect that.
- Educate employees to avoid that.
- Automate as much as possible the user creation process to ensure admins don't handle the account manually, reducing the risk of that happening.

## Detection

The best technique is to baseline users' behavior. Automated tools that monitor user behavior have shown increased success in detecting abnormal logons.
We would then expect events with IDs 4624, 4625 and 4768.
One thing to note is that the event ID 4738, that happens when a user object is modified, does not show the specific property that was altered, meaning we cannot use this to detect that.

## Honeypot

Storing credentials in properties of objects is an excellent honeypot technique. Just ensure the following:
- The password/credential is configured in the _Description_ field, as it's the easiest to pick up.
- The provided password is fake/incorrect.
- The account is enabled and has recent login attempts.
- Using a service account as a honeypot is better because these tend to be created by administrators manually.
- The account has the last password configured 2+ years ago.

The provided password is wrong, so we'll expect event IDs 4625, 4771 and 4776.
