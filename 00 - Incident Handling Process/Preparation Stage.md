We have 2 separate objectives:
- the establishment of incident handling capability
- the ability to protect against and prevent IT security incidents by implementing appropriate protective measures
## Preparation Prerequisites

Ensure we have:
- skilled incident handling team members
- trained workforce, through security awareness activities or other means of training
- clear policies and documentation
- tools (software and hardware)

## Clear Policies & Documentation

They should contain an up-to-date version of the following:
- contact information and roles of the incident handling team members
- contact information for the legal and compliance department, management team, IT support, communications and media relations department, law enforcement, internet service providers, facility management and external incident response team
- incident response policy, plan and procedures
- incident information sharing policy and procedures
- baselines of systems and networks, out of a golden image and a clean state environment
- network diagrams
- organization-wide asset management database
- user accounts with excessive privileges that can be used on-demand by the team when necessary. These are usually enabled when an incident is confirmed during the initial investigation and then disable once it is over. Also requires a password reset
- ability to acquire hardware, software or an external resource without a complete procurement process (urgent purchase of up to a certain amount)
- forensic/investigative cheat sheets

Some of the non-severe cases may be handled relatively quickly, but a data breach involving customer data has to be reported to law enforcement withing a certain threshold in accordance with GDPR.
It is also important to keep documenting the incident as you investigate. Establish an effective reporting capability. These notes should contain timestamps, the activity performed, the result of it and who did it.

## Tools (Software & Hardware)

Ensure we have the right tools, which include but are not limited to:
- additional laptop or a forensic workstation for each team member to preserve disk images and log files, perform data analysis and investigate without any restrictions. These should not introduce risks to the organization
- digital forensic image acquisition and analysis tools
- memory capable and analysis tools
- live response captures and analysis
- log analysis tools
- network captures and analysis tools
- network cables and switches
- write blockers
- hard drives for forensic imaging
- power cables
- screwdrivers, tweezers and other relevant tools to repair or disassemble hardware devices
- Indicator of Compromise (IOC) creator and the ability to search for IOCs across the organization
- chain of custody forms
- encryption software
- ticket tracking system
- secure facility for storage and investigation
- incident handling system independent of your organization's infrastructure

Many of the tools mentioned will be part of a **jump bag**, that must be always ready to be picked up and leave immediately.
The documentation system must be completely independent from the organization's infrastructure and properly secured. Assume that your entire domain is compromised and that all systems can become unavailable.
Communications about an incident should be conducted through channels that are not part of the organization's systems.

## DMARC

[DMARC](https://dmarcly.com/blog/how-to-implement-dmarc-dkim-spf-to-stop-email-spoofing-phishing-the-definitive-guide#what-is-dmarc) is an email protection against phishing built on top of [SPF](https://dmarcly.com/blog/how-to-implement-dmarc-dkim-spf-to-stop-email-spoofing-phishing-the-definitive-guide#what-is-spf) and [DKIM](https://dmarcly.com/blog/how-to-implement-dmarc-dkim-spf-to-stop-email-spoofing-phishing-the-definitive-guide#what-is-dkim). The idea is to reject emails that "pretend" to originate from your organization.
It is easy and inexpensive to implement, but it requires testing; otherwise you risk blocking legitimate emails with no ability to recover them.

## Endpoint Hardening (& EDR)

Endpoint devices are the entry points for most of the attacks that we are facing on a daily basis.
There are a few recognized endpoint hardening standards , with CIS and Microsoft baselines begin the most popular. Some highly important actions to note and do something about are:
- disable LLMNR/NetBIOS
- implement LAPS and remove administrative privileges from regular users
- disable or configure PowerShell in "ContrainedLanguage" mode
- enable Attack Surface Reduction (ASR) rules if using Microsoft Defender
- implement whitelisting, though being nearly impossible to implement, but consider at least blocking execution from user-writable folders (Downloads, Desktop, AppData) as these are the locations where exploits and malicious payloads will initially find themselves. block script types such as .hta, .vbs, .cmd, .batm .js and similar (see [LOLBin](https://lolbas-project.github.io/))
- utilize host-based firewalls. as a bare minimum, block workstation-to-workstation communication and block outbound traffic to LOLBins
- deploy an EDR product. [AMSI](https://learn.microsoft.com/en-us/windows/win32/amsi/how-amsi-helps) provides great visibility into obfuscated scripts for antimalware products to inspect the content before it gets executed. only choose products that integrate with AMSI

When it comes to hardening, **Don't let perfect be the enemy of good**.

## Network Protection

Network segmentation is a powerful technique to avoid having a breach spread across the entire organization. Business-critical systems must be isolated and connections should be allowed only as the business requires. internal resources should not be facing the internet directly unless placed in a DMZ.
Also consider IDS/IPS systems. Their power shine when SSL/TLS interception is performed so that they can identify malicious traffic based on content on the wire and not based on reputation of IP addresses.
Additionally, ensure that only organization-approved devices can get on the network. 802.1x can be utilized to reduce the risk on BYOD or malicious devices connecting to the corporate network.

## Privilege Identity Management / MFA / Passwords

Stealing privileged user credentials is the most common escalation path in Active Directory environments. A common mistake is that admin users either have a weak password or a shared password with their regular user account, which can be obtained via multiple attack vectors such as keylogging.
It is important to teach employees to use pass phrases because they are hard to guess and difficult to brute force.
MFA is another identity-protecting solution that should be implemented at least for any type of administrative access to **ALL** applications and devices.

## Vulnerability Scanning

Perform continuous vulnerability scans of your environment and remediate at least the "high" and "critical" vulnerabilities that are discovered.
While the scanning can be automated, the fixes usually require manual involvement.
If you can't apply a patch, segment the systems that are vulnerable.

## User Awareness Training

Training users to recognize suspicious behavior and report it when discovered is a big win. This training are known to significantly reduce the number of successful compromises.
Periodic "surprise" testing should be part of this training, like monthly phishing emails, dropped USB sticks in the office building, etc.

## Active Directory Security Assessment

Look for security misconfigurations and exposed critical vulnerabilities from the perspective of an attacker.
Ensure there's no one-step escalation possibility to high privileges on the network. the more additional tools activity an attacker is generating, the higher likelihood of you detecting them so try to eliminate low-hanging fruits as much as possible.
Active Directory has a few known and unique escalation paths/bugs and new ones are quite often discovered. Security assessments are crucial for the security posture of the environment.
Don't assume that your system administrators are aware of all discovered or published bugs.

## Purple Team Exercises

Train incident handlers and keep them engaged. The best place to do it is inside an organization's own environment.
Purple team exercises are security assessment performed by a red team that either continuously or eventually inform the blue team about their actions, findings, any visibility/security shortcomings, etc.
Such exercises will help in identifying vulnerabilities in an organization, while testing the blue team's defensive capability in terms of logging, monitoring, detection and responsiveness.
If a threat goes unnoticed, there's a opportunity to improve.

## Questions
- What should we have prepared and always ready to "grab and go"?
	- jump bag

- True or False: Using baselines, we can discover deviations from the golden image, which aids us in discovering suspicious or unwanted changes to the configuration.
	- True

- What can we use to block phishing emails pretending to originate from our mail server?
	- DMARC

- True or False: "Summer2021!" is a complex password.
	- True