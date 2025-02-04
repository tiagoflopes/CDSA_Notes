## Description

A Group Policy Object (GPO) is a virtual collection of policy settings that has a unique name. They are the most widely used configuration management tool in AD. Each contains zero or more policy settings. They are linked to an Organizational Unit and they can be restricted to which objects they apply by specifying, for example, an AD group, or a WMI filter.
When a GPO is created, only Domain admins can modify it. However, within environments, we will encounter different delegation that allow less privileged accounts to perform edits on the GPOs.
This access will allow an adversary to compromise all computer objects in the OUs that the vulnerable GPOs are linked to.

## Attack

No attack walkthrough, since it's a simple GPO edit or file replacement.

## Prevention

One way to prevent this attack is to lock down the GPO permissions to be modified by a particular group of users only or by a specific account. Similarly, never deploy files stored in network locations so that many users can modify the share permissions.
We should also review the permissions of GPOs actively and regularly, with the option of automating a task that runs hourly and alerts if any deviations from the expected permissions are detected.

## Detection

Fortunately it is straightforward to detect when a GPO is modified, since it generates an event with ID 5136. If a user who is not expected to have the right to modify a GPO appears here, then a red flag should be raised. 

## Honeypot

A common thought  is that because of the easy detection methods of these attacks, it is worth having a misconfigured GPO in the environment. This will quickly turn out to be our weakest point if an escalation path is discovered via some GPO modification. However, when implementing a honeypot, consider the following:
- GPO is linked to non-critical servers only.
- Continuous automation is in place for monitoring modifications of GPO (If it is modified, we disable the user immediately).
- The GPO should be automatically unlinked from all locations if a modification is detected.

The following PowerShell script will detect and send an alert when the honeypot GPO is modified.
```powershell
# Define filter for the last 15 minutes
$TimeSpan = (Get-Date) - (New-TimeSpan -Minutes 15)

# Search for event ID 5136 (GPO modified) in the past 15 minutes
$Logs = Get-WinEvent -FilterHashtable @{LogName='Security';id=5136;StartTime=$TimeSpan} -ErrorAction SilentlyContinue |`
Where-Object {$_.Properties[8].Value -match "CN={73C66DBB-81DA-44D8-BDEF-20BA2C27056D},CN=POLICIES,CN=SYSTEM,DC=EAGLE,DC=LOCAL"}


if($Logs){
    $emailBody = "Honeypot GPO '73C66DBB-81DA-44D8-BDEF-20BA2C27056D' was modified`r`n"
    $disabledUsers = @()
    ForEach($log in $logs){
        If(((Get-ADUser -identity $log.Properties[3].Value).Enabled -eq $true) -and ($log.Properties[3].Value -notin $disabledUsers)){
            Disable-ADAccount -Identity $log.Properties[3].Value
            $emailBody = $emailBody + "Disabled user " + $log.Properties[3].Value + "`r`n"
            $disabledUsers += $log.Properties[3].Value
        }
    }
    # Send an alert via email - complete the command below
    # Send-MailMessage
    $emailBody
}
```

If a user is disabled, it will generate an event with ID 4725
