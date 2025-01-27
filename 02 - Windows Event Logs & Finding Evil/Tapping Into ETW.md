## Detecting Strange Parent-Child Relationships

In Windows, certain processes never call or spawn others. For example, it is unlikely to see "calc.exe" spawning "cmd.exe". Here are a few examples.
![[parent_child_processes.jpg]]

It is possible to see these relationships using Process Hacker (sort by dropdowns).
To showcase a weird relationship, we will utilize an attacking technique called Parent PID Spoofing, executed through the [psgetsystem project](https://github.com/decoder-it/psgetsystem) this way:
```powershell
powershell -ep bypass
Import-Module .\psgetsys.ps1
[MyProcess]::CreateProcessFromParent([Process ID of spoolsv.exe], "C:\Windows\System32\cmd.exe", "")
```

In Sysmon we get this:
![[parent_pid_spoofing.png]]
The event incorrectly displays spoolsv.exe as the parent of cmd.exe (it is actually powershell.exe that created cmd.exe).
We will collect data from the Microsoft-Windows-Kernel-Process provider, using [SilkETW](https://github.com/mandiant/SilkETW) (the provider can be identified using `logman`).
Then run `SilkETW.exe -t user -pn Microsoft-Windows-Kernel-Process -ot file -p C:\windows\temp\etw.json`.
If we search for the process ID of spoolsv.exe in the json, we find out that powershell.exe actually created the cmd.exe process.

## Detecting Malicious .NET Assembly Loading

Traditionally, adversaries employed a strategy known as "Living of the land", which means they would execute legitimate system tools to carry out their malicious operations. This reduces the risk of detection but there are already countermeasures against this strategy.
There is now a new approach: "Bring your own land". Instead of relying on tools already present on a victim's system, threat actors emulating these tactics employ .NET assemblies executed entirely in memory.
The ["execute-assembly"](https://www.cobaltstrike.com/blog/cobalt-strike-3-11-the-snake-that-eats-its-tail/) is a powerful illustration.
Let's demonstrate this by emulating a malicious .NET assembly load by executing a precompiled version of [Seatbelt](https://github.com/GhostPack/Seatbelt) that resides on disk.
`.\Seatbelt.exe TokenPrivileges` should be detected on Sysmon (Event ID 7), because it triggers the loading of key .NET-related DLLs.
But using Sysmon to detect this can be challenging, so let's use the Microsoft-Windows-DotNETRuntime provider.
`SilkETW.exe -t user -pn Microsoft-Windows-DotNETRuntime -uk 0x2038 -ot file -p C:\Windows\temp\etw.json`
To learn more about *0x2038* see [this](https://medium.com/threat-hunters-forge/threat-hunting-with-etw-events-and-helk-part-1-installing-silketw-6eb74815e4a0).
