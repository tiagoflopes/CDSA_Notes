Logs from Windows Event and Sysmon are way too much, so we need tools to analyze these en masse. One of these tools is the [Get-WinEvent cmdlet](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.3) in PowerShell.

## Using Get-WinEvent

This provides us with the capability to retrieve different types of event logs. To quickly identify the available logs, we can leverage the *-ListLog* parameter to retrieve all logs. Then we can display some essential properties.
```powershell
Get-WinEvent -ListLog * | Select-Object LogName, RecordCount, IsClassicLog, IsEnabled, LogMode, LogType | Format-Table -AutoSize
```

We can also explore the event log providers
```powershell
Get-WinEvent -ListProvider * | Format-Table -AutoSize
```

Here are some examples demonstrating how to retrieve events from various logs.
##### 1. **Retrieving from the System log**
This retrieves the first 50 events from the System log.
```powershell
Get-WinEvent -LogName 'System' -MaxEvents 50 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

##### 2. **Retrieving from Microsoft-Windows-WinRM/Operational**
Events are retrieved from Microsoft-Windows-WinRM/Operational log.
```powershell
Get-WinEvent -LogName 'Microsoft-Windows-WinRM/Operational' -MaxEvents 30 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

To retrieve the oldest events, instead of sorting we can use *-Oldest*.

##### 3. **Retrieving from .evtx Files**
If you have exported .evtx file, you can read and query those logs.
```powershell
Get-WinEvent -Path 'C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Execution\exec_sysmon_1_lolbin_pcalua.evtx' -MaxEvents 5 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

##### 4. **Filtering with FilterHashtable**
This enables us to define specific conditions for the logs we want to retrieve.
```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1,3} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

For exported events, the equivalent command is the following.
```powershell
Get-WinEvent -FilterHashtable @{Path='C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Execution\sysmon_mshta_sharpshooter_stageless_meterpreter.evtx'; ID=1,3} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

And based on a data range, can be done as follows.
```powershell
$startDate = (Get-Date -Year 2023 -Month 5 -Day 28).Date
$endDate = (Get-Date -Year 2023 -Month 6 -Day 3).Date
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1,3; StartTime=$startDate; EndTime=$endDate} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```
Note: The start date is inclusive and the end date is exclusive.

##### 5. **Filtering with FilterHashtable & XML**
Consider an intrusion detection scenario, where there is a suspicious connection to a particular IP.
```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=3} |
`ForEach-Object {
$xml = [xml]$_.ToXml()
$eventData = $xml.Event.EventData.Data
New-Object PSObject -Property @{
    SourceIP = $eventData | Where-Object {$_.Name -eq "SourceIp"} | Select-Object -ExpandProperty '#text'
    DestinationIP = $eventData | Where-Object {$_.Name -eq "DestinationIp"} | Select-Object -ExpandProperty '#text'
    ProcessGuid = $eventData | Where-Object {$_.Name -eq "ProcessGuid"} | Select-Object -ExpandProperty '#text'
    ProcessId = $eventData | Where-Object {$_.Name -eq "ProcessId"} | Select-Object -ExpandProperty '#text'
}
}  | Where-Object {$_.DestinationIP -eq "52.113.194.132"}
```
The Windows XML EventLog (EVTX) format is [here](https://github.com/libyal/libevtx/blob/main/documentation/Windows%20XML%20Event%20Log%20(EVTX).asciidoc).

In the "Tapping into ETW" section, we were looking for anomalous clr.dll and mscoree.dll loading activity.
```powershell
$Query = @"
	<QueryList>
		<Query Id="0">
			<Select Path="Microsoft-Windows-Sysmon/Operational">*[System[(EventID=7)]] and *[EventData[Data='mscoree.dll']] or *[EventData[Data='clr.dll']]
			</Select>
		</Query>
	</QueryList>
	"@
Get-WinEvent -FilterXml $Query | ForEach-Object {Write-Host $_.Message `n}
```

##### 6. **Filtering with FilterXPath**
If we want to get Process Creation (Sysmon Event ID 1) events, we can utilize the command below.
```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[EventData[Data[@Name='Image']='C:\Windows\System32\reg.exe']] and *[EventData[Data[@Name='CommandLine']='`"C:\Windows\system32\reg.exe`" ADD HKCU\Software\Sysinternals /v EulaAccepted /t REG_DWORD /d 1 /f']]" | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

Let's suppose we want to investigate any network connections to a particular IP that Sysmon has logged.
```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[System[EventID=3] and EventData[Data[@Name='DestinationIp']='52.113.194.132']]"
```

##### 7. **Filtering based on property values**
This presents us with all properties of Sysmon event ID 1 logs.
```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} -MaxEvents 1 | Select-Object -Property *
```

Now a command that retrieves *Process Create* events from the *Microsoft-Windows-Sysmon/Operational* log.
```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} | Where-Object {$_.Properties[21].Value -like "*-enc*"} | Format-List
```
The index 21 of Properties of each object contains the *ParentCommandLine* property of the event

## Practical Exercise

I used the following command to retrieve all logs from the directory and search for the ID that indicates that a share was added.
```powershell
$logPath = "C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Lateral Movement"
$logFiles = Get-ChildItem -Path $logPath -Filter *.evtx
Get-WinEvent -Path $logFiles.FullName | Where-Object { $\_.Id -eq 5142 }
```