The attack lifecycle describes how attacks manifest themselves.
![[cyber_kill_chain.png]]
- Recon - the initial stage, where an attacker chooses their target and gather as much information as they can (passive or active)
- Weaponize - where the attacker develops and embeds the malware into some type of exploit or deliverable payload. the malware is crafted to be extremely lightweight and undetectable by the antivirus and detection tools (for this to be possible, the attacker must've gather information to identify those)
- Delivery - this is where the exploit/payload is delivered to the victim(s) either by phishing emails, smishing, vishing, physical interaction... It is rare to deliver a payload that requires the victim to do more than double-click an executable file
- Exploitation - when an exploit or payload is triggered. The attacker  typically attempts to execute code on the target system in order to gain access or control
- Installation - where the initial stager is executed and is running on the compromised machine. some common techniques are:
	- Droppers - small piece of code designed to install malware in the system and execute it. Via email attachments, malicious websites or social engineering tactics
	- Backdoors - designed to provide the attacker with ongoing access to the compromised system. Installed during exploitation or delivered via a dropper
	- Rootkits - designed to hide its presence on a compromised machine. to evade detection. Also installed during exploitation or delivered via a dropper
- Command and Control - the attacker establishes a remote access capability to the machine
- Action - this is the objective of an attack, which can vary. Exfiltrating confidential data, obtain the highest level of access to deploy a ransomware

This is not linear. After the installation stage, the logical next step is to initiate recon again to identify additional targets and find vulnerabilities to exploit.
Our objective is to **stop an attacker from progressing further up the kill chain**, ideally in one of the early stages.

## Questions
- In which stage of the cyber kill chain is malware developed?
	- Weaponize