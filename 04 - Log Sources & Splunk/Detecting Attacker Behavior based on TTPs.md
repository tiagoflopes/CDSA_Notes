Effective threat detection often revolves around identifying patterns that either match known malicious behaviors or diverge significantly from expected norms. When crafting detection-related SPL searches, there are two main approaches:
- Leveraging our extensive knowledge of specific threats and attack vectors. If an entity behaves in a way that we recognize as characteristic of a particular threat, it draws our attention. *Spot the known*.
- Leaning heavily on statistical analysis and anomaly detection to identify abnormal behavior within the sea of normal activity. *Spot the unusual*.

## Crafting SPL Searches Based on Known TTPs

Below are some detection examples that follow the first approach.

1. Detection of reconnaissance activities leveraging native windows binaries
Attackers often leverage native Windows binaries such as net.exe to gain insights into the target environment, identify potential privilege escalation opportunities and perform lateral movement.
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 Image=*\\ipconfig.exe OR Image=*\\net.exe OR Image=*\\whoami.exe OR Image=*\\netstat.exe OR Image=*\\nbtstat.exe OR Image=*\\hostname.exe OR Image=*\\tasklist.exe | stats count by Image,CommandLine | sort - count
```

2. Detection of requesting malicious payloads/tools hosted on reputable/whitelisted domains (such as githubusercontent.com)
Attackers often use that website as a hosting platform for their payloads. This is due to the common whitelisting and permissibility of the domain by company proxies.
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=22  QueryName="*github*" | stats count by Image, QueryName
```

3. Detection of PsExec usage
This tool was conceived as a utility to aid system administrators in managing remote Windows systems. Several MITRE ATT&CK techniques have seen it in play. After some searching, we find out that event ids 13, 11, 17 and 18 may be useful in identifying its usage.

_Case 1 - Event ID 13_
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=13 Image="C:\\Windows\\system32\\services.exe" TargetObject="HKLM\\System\\CurrentControlSet\\Services\\*\\ImagePath" | rex field=Details "(?<reg_file_name>[^\\\]+)$" | eval reg_file_name = lower(reg_file_name), file_name = if(isnull(file_name),reg_file_name,lower(file_name)) | stats values(Image) AS Image, values(Details) AS RegistryDetails, values(_time) AS EventTimes, count by file_name, ComputerName
```
This query is looking for instances where the services.exe process has modified the ImagePath value of any service. The output will include the details of these modifications, including the name of the modified service, the new ImagePath value and the time of the modification.

_Case 2 - Event ID 11_
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image=System | stats count by TargetFilename
```

_Case 3 - Event ID 18_
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=18 Image=System | stats count by PipeName
```

4. Detection of utilizing archive files for transferring tools or data exfiltration
Attackers may employ **zip**, **rar** or **7z** files for transferring tools to a compromised host or exfiltrating data from it.
```shell-session
index="main" EventCode=11 (TargetFilename="*.zip" OR TargetFilename="*.rar" OR TargetFilename="*.7z") | stats count by ComputerName, User, TargetFilename | sort - count
```

5. Detection of utilizing PowerShell or MS Edge for downloading payloads/tools
The following searches examine files downloaded through PowerShell or Ms Edge.
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image="*powershell.exe*" |  stats count by Image, TargetFilename |  sort + count
```
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image="*msedge.exe" TargetFilename=*"Zone.Identifier" |  stats count by TargetFilename |  sort + count
```
The *\*Zone.Identifier* is indicative of a file downloaded from the internet and contains metadata about where the file was downloaded from and its security settings.

6. Detection of execution from atypical or suspicious locations
The following search is designed to identify process creations in a user's Downloads folder.
```shell-session
index="main" EventCode=1 | regex Image="C:\\\\Users\\\\.*\\\\Downloads\\\\.*" |  stats count by Image
```

7. Detection of executables of DLLs being created outside the Windows directory
```shell-session
index="main" EventCode=11 (TargetFilename="*.exe" OR TargetFilename="*.dll") TargetFilename!="*\\windows\\*" | stats count by User, TargetFilename | sort + count
```

8. Detection of misspelling legitimate binaries
```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (CommandLine="*psexe*.exe" NOT (CommandLine="*PSEXESVC.exe" OR CommandLine="*PsExec64.exe")) OR (ParentCommandLine="*psexe*.exe" NOT (ParentCommandLine="*PSEXESVC.exe" OR ParentCommandLine="*PsExec64.exe")) OR (ParentImage="*psexe*.exe" NOT (ParentImage="*PSEXESVC.exe" OR ParentImage="*PsExec64.exe")) OR (Image="*psexe*.exe" NOT (Image="*PSEXESVC.exe" OR Image="*PsExec64.exe")) |  table Image, CommandLine, ParentImage, ParentCommandLine
```

9. Detection of using non-standard ports for communications/transfers
```shell-session
index="main" EventCode=3 NOT (DestinationPort=80 OR DestinationPort=443 OR DestinationPort=22 OR DestinationPort=21) | stats count by SourceIp, DestinationIp, DestinationPort | sort - count
```
