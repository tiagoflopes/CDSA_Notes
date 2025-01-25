We will now delve into the realm of malicious activities and discover techniques for detection.

## Sysmon Basics

System Monitor (Sysmon) is a Windows system service and device driver that remains resident across system reboots to monitor and log system activity to the Windows event log.
Sysmon's primary components include:
- A Windows service for monitoring system activity.
- A device driver that assists in capturing the system activity data.
- An event log to display captured activity data.
The full list of Sysmon event IDs can be found [here](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon).
Sysmon uses an XML-based configuration file, that allows to include or exclude certain types of events based on different attributes like process names, IP addresses, etc.
For a comprehensive configuration, visit https://github.com/SwiftOnSecurity/sysmon-config, or https://github.com/olafhartong/sysmon-modular, for a modular approach.

## Detecting DLL Hijacking

To detect a DLL hijack, we need to focus on Event Type 7, which corresponds to module load events. For this, we need to change the config file *sysmonconfig-export.xml* (downloaded from sysmon-config github repository).
In case of detecting DLL hijacks, we change the "include" to "exclude" to ensure that nothing is excluded, allowing us to capture the necessary data.
![[sysmon_dll.webp]]
Now, go to Event Viewer - Applications and Services - Microsoft - Windows - Sysmon
To gain more insights into DLL hijacks, here is a [blog](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows) with an exhaustive list of various techniques. In this example, we will focus on calc.exe and WININET.dll.
Here is how to perform the ["hello world" reflective DLL](https://github.com/stephenfewer/ReflectiveDLLInjection/tree/master/bin). Rename the new DLL, copy calc.exe and WININET.dll to a writable directory and run it.
The output from Sysmon provides valuable insights. Let's explore some IOCs:
- "calc.exe" should not be found in a writable directory.
- "WININET.dll" should not be loaded outside of System32 by calc.exe.
- The original DLL is Microsoft-signed, while our injected DLL remains unsigned.

## Detecting Unmanaged PowerShell/C-Sharp Injection

C# is considered a "managed" language, meaning it requires a backend runtime to execute its code. The Common Language Runtime (CLR) serves as this runtime environment.
As defenders, we can detect unusual C# injections or executions within our environment, using [Process Hacker](https://processhacker.sourceforge.io/).
Processes highlighted in green are managed, like powershell.exe. Examine the module loads, by going into Properties - Modules.
clr.dll and clrjit.dll are used when C# code is ran as part of the runtime to execute the bytecode (Intermediate Language). This wouldn't be required, which suggests a potential [execute-assembly](https://www.cobaltstrike.com/blog/cobalt-strike-3-11-the-snake-that-eats-its-tail/) or [unmanaged PowerShell injection](https://www.youtube.com/watch?v=7tvfb9poTKg&ab_channel=RaphaelMudge) attack.
Let's inject an unmanaged PowerShell-like DLL into a random process (spollsv.exe) using [PSInject project](https://github.com/EmpireProject/PSInject).
```powershell
powershell -ep bypass
Import-Module .\Invoke-PSInject.ps1
Invoke-PSInject -ProcId [Process ID of spoolsv.exe] -PoshCode "V3JpdGUtSG9zdCAiSGVsbG8sIEd1cnU5OSEi"
```

After the injection, the binary transitions from an unmanaged to a managed state.

## Detecting Credential Dumping

One widely used tool for credential dumping is [Mimikatz](https://github.com/gentilkiwi/mimikatz), offering various methods for extracting Windows credentials.
The command *"sekurlsa::logonpasswords"* enables the dumping of password hashes or plaintext passwords by accessing LSASS.
To detect this activity we shift to Sysmon event ID 10, which represents "ProcessAccess" events.
![[example_mimikatz_process.webp]]
Here, we observe a random file "AgentEXE.exe" from a random folder "Downloads" attempting to access LSASS, which is unusual. Also the SourceUser being different form the TargetUser further emphasizes the abnormality. It's also worth noting that as part of the mimikatz-based credential dumping process, the user must request SeDebugPrivileges, which can be another IOC.