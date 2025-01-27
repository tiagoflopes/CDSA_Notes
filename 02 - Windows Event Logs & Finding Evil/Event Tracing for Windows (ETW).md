## What is ETW?

It's a general-purpose, high-speed tracing facility provided by the operating system. It is implemented in the kernel, so it provides a tracing mechanism for events raised by kernel-mode device drivers.
ETW captures a diverse set of events, spanning system calls, process creation and termination, network activity, file and registry modifications, and numerous other dimensions.

## ETW Architecture & Components

![[etw_arch.webp]]

- Controllers - Assume control over all aspects related to ETW operations. Initiating and termination trace sessions, enabling or disabling providers...
At the core, the architecture is the publish-subscribe model.
- Providers (publishers) - Generate events and write them to the designated ETW sessions. There are 4 types:
	- MOF - Based on Managed Object Format and are capable of generating events according to predefined MOF schemas.
	- WPP - Windows Software Trace Preprocessor providers leverage specialized macros and annotations within the app's source code to generate events.
	- Manifest-based - More contemporary form of providers that rely on XML manifest files that define the structure and characteristics of events.
	- TraceLogging - A simplified and efficient approach to event generation, leveraging the TraceLogging API, in recent Windows versions.
- Consumers (subscribers) - Subscribe to specific events of interest and receive those events for further processing or analysis.
- Channels - Facilitate efficient event collection and consumption. Act as logical containers for organizing and filtering events based in their characteristics and importance.
- ETL files - ETW provides specialized support for writing events to disk through the use of event trace log files.

## Interacting With ETW

*Logman* is a utility for managing ETW and Event Tracing Sessions. This tool is invaluable for creating, initiating, halting and investigating tracing sessions.
![[example_logman.png]]

The following command provides more information on the specified set.
```powershell
logman.exe query "EventLog-System" -ets
```

And this one is for listing all providers.
```powershell
logman.exe query providers
```
Windows 10 includes more than 1000 built-in providers.

The *Performance Monitor* is the GUI alternative.

## Useful Providers

- `Microsoft-Windows-Kernel-Process`: This ETW provider is instrumental in monitoring process-related activity within the Windows kernel. It can aid in detecting unusual process behaviors such as process injection, process hollowing, and other tactics commonly used by malware and advanced persistent threats (APTs).
- `Microsoft-Windows-Kernel-File`: As the name suggests, this provider focuses on file-related operations. It can be employed for detection scenarios involving unauthorized file access, changes to critical system files, or suspicious file operations indicative of exfiltration or ransomware activity.
- `Microsoft-Windows-Kernel-Network`: This ETW provider offers visibility into network-related activity at the kernel level. It's especially useful in detecting network-based attacks such as data exfiltration, unauthorized network connections, and potential signs of command and control (C2) communication.
- `Microsoft-Windows-SMBClient/SMBServer`: These providers monitor Server Message Block (SMB) client and server activity, providing insights into file sharing and network communication. They can be used to detect unusual SMB traffic patterns, potentially indicating lateral movement or data exfiltration.
- `Microsoft-Windows-DotNETRuntime`: This provider focuses on .NET runtime events, making it ideal for identifying anomalies in .NET application execution, potential exploitation of .NET vulnerabilities, or malicious .NET assembly loading.
- `OpenSSH`: Monitoring the OpenSSH ETW provider can provide important insights into Secure Shell (SSH) connection attempts, successful and failed authentications, and potential brute force attacks.
- `Microsoft-Windows-VPN-Client`: This provider enables tracking of Virtual Private Network (VPN) client events. It can be useful for identifying unauthorized or suspicious VPN connections.
- `Microsoft-Windows-PowerShell`: This ETW provider tracks PowerShell execution and command activity, making it invaluable for detecting suspicious PowerShell usage, script block logging, and potential misuse or exploitation.
- `Microsoft-Windows-Kernel-Registry`: This provider monitors registry operations, making it useful for detection scenarios related to changes in registry keys, often associated with persistence mechanisms, malware installation, or system configuration changes.
- `Microsoft-Windows-CodeIntegrity`: This provider monitors code and driver integrity checks, which can be key in identifying attempts to load unsigned or malicious drivers or code.
- `Microsoft-Antimalware-Service`: This ETW provider can be employed to detect potential issues with the antimalware service, including disabled services, configuration changes, or potential evasion techniques employed by malware.
- `WinRM`: Monitoring the Windows Remote Management (WinRM) provider can reveal unauthorized or suspicious remote management activity, often indicative of lateral movement or remote command execution.
- `Microsoft-Windows-TerminalServices-LocalSessionManager`: This provider tracks local Terminal Services sessions, making it useful for detecting unauthorized or suspicious remote desktop activity.
- `Microsoft-Windows-Security-Mitigations`: This provider keeps tabs on the effectiveness and operations of security mitigations in place. It's essential for identifying potential bypass attempts of these security controls.
- `Microsoft-Windows-DNS-Client`: This ETW provider gives visibility into DNS client activity, which is crucial for detecting DNS-based attacks, including DNS tunneling or unusual DNS requests that may indicate C2 communication.
- `Microsoft-Antimalware-Protection`: This provider monitors the operations of antimalware protection mechanisms. It can be used to detect any issues with these mechanisms, such as disabled protection features, configuration changes, or signs of evasion techniques employed by malicious actors.

## Restricted Providers

This providers offer valuable telemetry but are only accessible to processes that carry the requisite permissions.
According to Elastic: _To be able to run as a PPL, an anti-malware vendor must apply to Microsoft, prove their identity, sign binding legal documents, implement an Early Launch Anti-Malware (ELAM) driver, run it through a test suite, and submit it to Microsoft for a special Authenticode signature. It is not a trivial process. Once this process is complete, the vendor can use this ELAM driver to have Windows protect their anti-malware service by running it as a PPL._ With that said, [workarounds to access the Microsoft-Windows-Threat-Intelligence provider exist](https://posts.specterops.io/uncovering-windows-events-b4b9db7eac54).

## References

- https://nasbench.medium.com/a-primer-on-event-tracing-for-windows-etw-997725c082bf
- https://web.archive.org/web/20230222121234/https://bmcder.com/blog/a-begginers-all-inclusive-guide-to-etw