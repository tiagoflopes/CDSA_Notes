## Threat Intelligence Report: Stuxbot

Stuxbot is a organized cybercrime collective. Their primary motivation appears to be espionage. They attack Windows users and can end up completely takeover the victim's computer or perform domain escalation.
This is their lifecycle.
![[lifecycle_stuxbot.webp]]

##### Initial Breach
There is a link to a OneNote file hosted on a file hosting service. This file masquerades as an invoice featuring a 'HIDDEN' button that triggers an embedded batch file, which fetches PowerShell scripts.
##### RAT Characteristics
The RAT deployed is modular, meaning it can be augmented with infinite capabilities.
##### Persistence
The mechanisms utilized have involved an EXE file deposited on the disk.
##### Lateral Movement
There are 2 methods that have been identified: leveraging the original Microsoft-signed PsExec, and using WinRM.
##### IoCs
OneNote File:
- https://transfer.sh/get/kNxU7/invoice.one
- https://mega.io/dl9o1Dz/invoice.one

Staging Entity (PowerShell Script):
- https://pastebin.com/raw/AvHtdKb2
- https://pastebin.com/raw/gj58DKz

Command and Control (C&C) Nodes:
- 91.90.213.14:443
- 103.248.70.64:443
- 141.98.6.59:443

Cryptographic Hashes of Involved Files (SHA256):
- 226A723FFB4A91D9950A8B266167C5B354AB0DB1DC225578494917FE53867EF2
- C346077DAD0342592DB753FE2AB36D2F9F1C76E55CF8556FE5CDA92897E99C7E
- 018D37CBD3878258C29DB3BC3F2988B6AE688843801B9ABC28E6151141AB66D4

## Hunting with The Elastic Stack

##### The Available Data
We have logs from:
- Windows audit (windows*)
- System Monitor (Sysmon, windows*)
- PowerShell (windows*)
- Zeek (zeek*)

##### The Environment
Small organization, with about 200 employees engaged in online marketing activities. Office applications are the primary software in use, Gmail serves as our standard email provider, Edge is the default browser, remote support is provided through TeamViewer and all company devices are managed via Active Directory Group Policy Objects (GPOs). Considering a transition to Microsoft Intune for endpoint management as part of an upcoming upgrade from W10 to W11.

##### The Task
Around a threat intelligence report concerning Stuxbot.

##### The Hunt
This is premised on the hypotheses of a successful phishing email delivering a malicious OneNote file.
The report indicates that initial compromises took place via "invoice.one" file. We will initiate a comprehensive search based on Sysmon Event ID 15 (FileCreateStreamHash), which represents a browser file download event.
`event.code:15 and file.name=*invoice.one`
We get 3 hits (Not confirmed that the file is the one mentioned above).
The process.name and process.executable indicate that Edge was used and it was stored in Bob's downloads folder, stated in file.path.
The timestamp to note is: *March 26, 2023 @ 22:05:47*.
We can corroborate this by examining Sysmon Event ID 11 (File create). The query `event.code:11 and file.name:invoice.one*` shows 1 hit.
The hostname of the machine is WS001 (host.hostname or host.name) and its IP address is 192.168.28.130 (`event.code:3 and host.hostname:WS001`, checking source.ip).
If we inspect network connections (Sysmon Event ID 3) around the time this file was downloaded, we'll find that Sysmon has no entries. This is common configuration to avoid capturing an overwhelming volume of logs of connections created by browsers. This is where Zeek logs prove invaluable.
We should filter and examine DNS queries from WS001 during the interval 22:05:00 to 22:05:48, when the file was downloaded. It is however required to do some filtering, to remove noise. This is the final configuration.
![[zeek_query_dns.webp]]
Then, we should add the dns.question.name column, so that we see it for each entry. This is the result.
![[zeek_dns_entries.png]]
The first 2 entries relate to the defender SmartScreen scanning a file, that kicks in when a file is downloaded in Edge, and the other 3 entries are the file hosting site. A little before that we can see mail.google.com, when Bob accessed his emails.
Expanding the log entry for file.io reveals the returned IP addresses: 34.197.10.85 and 3.213.216.16.
Searching for that IP as destination.ip during that timeframe it leads to connections to that IP, as expected. This corroborates that a user, Bob, successfully downloaded the file "invoice.one" from the hosting provider "file.io".
From now on, we can either cross-reference the data with the Threat Intel report to identify overlapping information within our environment, or conduct an IR-like investigation to trace the sequence of events post the file download. We choose to proceed with the latter approach.
Hypothetically, if the file was accessed, it would be opened with the OneNote app. The query  `event.code:1 and process.command_line:*invoice.one*` returns 1 hit, and the timeframe is about 6 seconds after the download. Now let's see the execution of something by onenote. Sysmon Event ID 1 (Process creation) is utilized.
`event.code:1 and process.parent.name:"ONENOTE.EXE"`
We get 3 entries. The last one is too old. The middle one executes ONENOTEM.EXE, which is a component that assists in launching files. And the top entry reveals "cmd.exe" in operation, executing a file named "invoice.bat".
The question now is, has this batch script instigated anything else? Let's see if a parent process with a command line argument pointing to the batch file has spawned any child processes, using `event.code:1 and process.parent.command_line:*invoice.bat*`. This returns a single result: a command to download and execute content from Pastebin. This was stated in the Threat Intelligence report on IoCs. Let's get the PID to get an overview of activities and add the event.code column. `process.pid:9944 and process.name:"powershell.exe"`
We get file creation, attempted network connections, and some DNS resolutions (Sysmon Event ID 22 DNSEvent). By adding some more tables, we get tons of new information:
![[hunt13.webp]]
Ngrok was likely employed as C2, to mask malicious traffic to a known domain, encrypted (because it points to port 443).
The dropped EXE is likely intended for persistence.
Let's review Zeek data for information on the destination IP address 18.158.249.75 we just discovered. The activity seems to have extended into the subsequent day. The reason for the termination of the activity is unclear.
Upon inspecting DNS queries for "ngrok.io" we find that the returned IP (dns.answers.data) has indeed altered. The newly discovered IP also indicates that connections continued consistently over the following days.
Now regarding the executable, let's query sysmon logs. It has been executed. The executable initiated DNS queries for Ngrok and established connections with the C2 IP addresses. It also uploaded 2 files "svchost.exe" and "SharpHound.exe". SharpHound is a tool for diagramming AD and identifying attack paths for escalation, but what about svchost.exe? Is it attempting to mimic the legitimate file which is part of Windows OS? If we scroll up there's further activity, including the uploading of "payload.exe", a VBS file, and repeated uploads of "svchost.exe".
Now did SharpHound execute? Since it was an on-disk executable file, let's execute the query `process.name:"SharpHound.exe"`. It has been executed twice.
It is worth mentioning that default.exe's file hash aligns with one found in the Threat Intel report. Has this been detected on other devices?`process.hash.sha256:018d37cbd3878258c29db3bc3f2988b6ae688843801b9abc28e6151141ab66d4`
By adding the host.hostname column, we can see that PKI has been breached too, and a backdoor file has been placed under the profile of user "svc-sql1" suggesting that this user's account is likely compromised. Expanding an instance of that executable on PKI reveals the parent process "PSEXECSVC", a component of PSExec from SysInternals, a tool utilized for lateral movement in AD breaches. The user.name field confirms that account has been compromised. The only plausible explanation is the earlier uploaded PowerShell script, designed for Password Bruteforcing. This was uploaded in WS001, so let'c check this machine's logins, excluding those from Bob, the owner.
`(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130` and excluding bob and WS001$ from user.name.
We get 2 failed logons under administrator and 4 successful logon under svc-sql1. So, first they tried to crack the admin's password but failed. However, 2 days later they manage to login as svc-sql1.
That's it for today i guess.
