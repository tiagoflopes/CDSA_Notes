## Introduction

Now we are going for a investigation on a much larger scale then before.
Among other stuff, we will try to keep eliminating false positives as much as possible.

## Ingesting Data Sources

The focus will be on personally created data, over 500000 events.
The goal isn't to identify every single attack or infection, but to understand how to begin to detect any sort of attack within such a vast data pool.

## Searching Effectively

As the blue team, our objective is to trace down TTPs and to craft alerts and hunting queries to cover as many potential threat vectors as we can.
First, we will identify all source types, to get the Sysmon data that we want.
`index=main | stats count by sourcetype`
This query returns each source type (including *WinEventLog:Sysmon*) and how may events it contains.
To get all data on Sysmon, run `index=main sourcetype="WinEventLog:Sysmon"`.
One key thing to notice is that targeted searches will return the results way faster, lessen resources consumption and allow others to keep using the SIEM with less disruption and impact.
This is truly important when creating alerts, and it cuts down a lot of irrelevant data. But what data do we want to focus on?

## Embracing the Mindset of Analysts, Threat Hunters & Detection Engineers

We'll try to spot any anomalies is our data. Since we are searching Sysmon events, let's count them by the Event Id.
`index=main sourcetype="WinEventLog:Sysmon" | stats count by EventCode` (EventCode is a predefined field for event.code)
We get 20 distinct EventCodes. Let's review a few:
- [Sysmon Event ID 1 - Process Creation](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90001): Useful for hunts targeting abnormal parent-child process hierarchies, as illustrated in the first lesson with Process Hacker. It's an event we can use later.
- [Sysmon Event ID 2 - A process changed a file creation time](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90002): Helpful in spotting "time stomp" attacks, where attackers alter file creation times. Bear in mind, not all such actions signal malicious intent.
- [Sysmon Event ID 3 - Network connection](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90003): A source of abundant noise since machines are perpetually establishing network connections. We may uncover anomalies, but let's consider other quieter areas first.
- [Sysmon Event ID 4 - Sysmon service state changed](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90004): Could be a useful hunt if attackers attempt to stop Sysmon, though the majority of these events are likely benign and informational, considering Sysmon's frequent legitimate starts and stops.
- [Sysmon Event ID 5 - Process terminated](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90005): This might aid us in detecting when attackers kill key processes or use sacrificial ones. For instance, Cobalt Strike often spawns temporary processes like werfault, the termination of which would be logged here, as well as the creation in ID 1.
- [Sysmon Event ID 6 - Driver loaded](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90006): A potential flag for BYOD (bring your own driver) attacks, though this is less common. Before diving deep into this, let's weed out more conspicuous threats first.
- [Sysmon Event ID 7 - Image loaded](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90007): Allows us to track dll loads, which is handy in detecting DLL hijacks.
- [Sysmon Event ID 8 - CreateRemoteThread](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90008): Potentially aids in identifying injected threads. While remote threads can be created legitimately, if an attacker misuses this API, we can potentially trace their rogue process and what they injected into.
- [Sysmon Event ID 10 - ProcessAccess](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90010): Useful for spotting remote code injection and memory dumping, as it records when handles on processes are made.
- [Sysmon Event ID 11 - FileCreate](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90011): With many files being created frequently due to updates, downloads, etc., it might be challenging to aim our hunt directly here. However, these events can be beneficial in correlating or identifying a file's origins later.
- [Sysmon Event ID 12 - RegistryEvent (Object create and delete)](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90012) & [Sysmon Event ID 13 - RegistryEvent (Value Set)](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90013): While numerous events take place here, many registry events can be malicious, and with a good idea of what to look for, hunting here can be fruitful.
- [Sysmon Event ID 15 - FileCreateStreamHash](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90015): Relates to file streams and the "Mark of the Web" pertaining to external downloads, but we'll leave this aside for now.
- [Sysmon Event ID 16 - Sysmon config state changed](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90016): Logs alterations in Sysmon configuration, useful for spotting tampering.
- [Sysmon Event ID 17 - Pipe created](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90017) & [Sysmon Event ID 18 - Pipe connected](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90018): Record pipe creations and connections. They can help observe malware's interprocess communication attempts, usage of [PsExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec), and SMB lateral movement.
- [Sysmon Event ID 22 - DNSEvent](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90022): Tracks DNS queries, which can be beneficial for monitoring beacon resolutions and DNS beacons.
- [Sysmon Event ID 23 - FileDelete](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90023): Monitors file deletions, which can provide insights into whether a threat actor cleaned up their malware, deleted crucial files, or possibly attempted a ransomware attack.
- [Sysmon Event ID 25 - ProcessTampering (Process image change)](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon): Alerts on behaviors such as process herpadering, acting as a mini AV alert filter.

Now we can perform preliminary queries. Let's find unusual parent-child trees, with the query `index=main sourcetype="WinEventLog:Sysmon" EventCode=1 | stats count by ParentImage, Image`.
We get over 5000 events, so let's target child processes known to be more problematic `index=main sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe") | stats count by ParentImage, Image`.
There's a notepad.exe to powershell.exe chain. Why? We need to validate if this is typical.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe") ParentImage="C:\\Windows\\System32\\notepad.exe"`
When we expand one event, we see that powershell is being executed to download a file from some unknown IP address.
From this we could trace what initiated the notepad.exe, or investigate other machines interacting with this IP. `index=main 10.0.0.229 | stats count by sourcetype`
Among the few options, most inform us that a connection has occurred, but not much more, except linux:syslog. `index=main 10.0.0.229 sourcetype="linux:syslog"`
It seems that it belongs to the host named waldo-virtual-machine. This sparks some concerns, hinting at the potential compromise of the Linux systems as well.
`index="main" 10.0.0.229 sourcetype="WinEventLog:sysmon" | stats count by CommandLine, host`
We see several downloads of known malicious binaries. It appears that the DCSync script was executed, indicating a likely DCSync attack. Let's confirm it.
`index=main EventCode=4662 Access_Mask=0x100 Account_Name!=*$`
That EventCode is triggered when an AD object is accessed. The access mask specifically requests Control Access typically needed for DCSync's high-level permissions. The account name is to search for users instead of accounts. To confirm this is a DCSync attack, let's check the properties field and search the GUIDs.
Upon researching, we find that it is linked to DS-Replication-Get-Changes-All, which allows the replication of secret domain data. It is confirmed. This signifies a full compromise in our network.
Now we should see how the attacker infiltrated and got admin privileges. We have previously observed detections for lsass dumping.
`index=main EventCode=10 lsass | stats count by SourceImage`
The most noticeable strange process accesses are notepad and rundll32. After searching for the notepad one sysmon suggests its purpose is for credential dumping, and the callstack refers to an UNKNOWN segment into ntdll. This implies that ANY API calls from this shellcode don't originate from any identifiable file on disk. These scenarios are limited to processes such as JIT processes.

## Creating Meaningful Alerts

We can now aim to create alerts from malicious malware based on API calls from UNKNOWN regions of memory.
We'll start by listing all the call stacks containing UNKNOWN during this lab period cased on event code to see which can yield the most meaningful data.
`index=main CallTrace="*UNKNOWN*" | stats count by EventCode`
Only the event code 10 shows anything related to that, so the alert will be tied to process access. There are lots of events so let's begin by grouping them based on SourceImage.
`index=main CallTrace="*UNKNOWN*" | stats count by SourceImage`
It shows all of those false positives mentioned, and they're all JIT as well.
To reduce those, first, we are not concerned when a process accesses itself: `... | where SourceImage!=TargetImage | ...`.
Then, we will exclude anything related to C# due to its JIT. `... SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll* | ...`.
Next, anything related to WOW64, because it often comprises regions of memory that are not backed by any specific file.`... CallTrace!=*wow64* | ...`.
We will also exclude Explorer.exe, considering its versatile nature. `... SourceImage!="C:\\Windows\\Explorer.EXE" | ...`.
Finally, we can see if it still makes sense to remove more, although by now it should be established a reasonable robust alert system.
To gain a more comprehensive understanding we could reintroduce some fields we moved earlier, like TargetImage and CallTrace to weed out any remaining false positives.`... | stats count by SourceImage, TargetImage, CallTrace`.
This is just a demonstration for this specific case. This can be bypassed easily. All a hacker could do is to load any random dll with "NI" appended to its name.
