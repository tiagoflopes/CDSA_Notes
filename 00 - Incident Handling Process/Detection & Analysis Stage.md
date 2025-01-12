Now, we have processes, procedures and guidelines on how to act upon security incidents.
This phase involves all aspects of detecting an incident, by utilizing:
- sensors
- logs
- trained personnel
- information and knowledge sharing
- context-based threat intelligence
- segmentation of the architecture
- clear understanding of and visibility within the network

It is highly recommended to create levels of detection:
- detection at the network perimeter (firewalls, internet-facing network IDS/IPS, DMZ)
- detection at the internal network level (local firewalls, host IDS/IPS)
- detection at the endpoint level (antivirus systems, EDR systems)
- detection at the application level (application logs, service logs)
## Initial Investigation

When a security incident is detected, conduct some initial investigation and establish context before assembling the team and calling an organization-wide incident response.
Aim to collect as much information as possible at this stage about the following:
- date/time when the incident was reported. additionally, who detected and/or who reported it
- how was it detected
- what was the incident (phishing, system unavailability, etc)
- assemble a list of impacted systems (if applicable)
- document who has accessed the impacted systems and actions have been taken
- physical location, OS, IP addresses and hostnames, system owner, system's purpose, current state of the system
- (if malware is involved) list of IP addresses, time and date of detection, type of malware, systems impacted, export of malicious files with forensic information on them

With all of this data gathered up, it is time to build an incident timeline, that will keep us organized throughout the event and provide an overall picture of what happened.
The timeline should contain the information described in the following columns:
![[timeline_columns.png]]
It focus primarily on attacker behavior.

## Incident Severity & Extent Questions

We should also try to answer the following questions to get an idea of the incident's severity and extent:
- What is the exploitation impact?
- What are the exploitation requirements?
- Can any business-critical systems be affected by the incident?
- Are there any suggested remediation steps?
- How many systems have been impacted?
- Is the exploit being used in the wild?
- Does the exploit have any worm-like capabilities?

The last two can possibly indicate the level of sophistication of an adversary.

## Incident Confidentiality & Communication

Incidents are very confidential topics and as such, all of the information gathered should be kept on a need-to-know basis, unless applicable laws or a management decision instruct us otherwise.
There are multiple reasons for this, like if the adversary is an employee of the company, or if there is a breach (which will be handled by the appointed person in accordance with the legal department).
When an investigation is launched, we will set some expectations and goals, like the type of incident that occurred, the sources of evidence and a rough estimation of how much time the team needs for the investigation.
It is important to keep everyone involved and the management informed about any advancements and expectations.

## The Investigation

The investigation starts based on the initially gathered (and limited) information that contain that we know about the incident so far.
With this initial data, we will begin a 3-step cyclic process that will iterate over and over again as the investigation evolves:
- creation and usage of indicators of compromise (IOC)
- identification of new leads and impacted systems
- data collection and analysis from the new leads and impacted systems

![[investigation_new.webp]]
### Initial Investigation Data
An investigation should be based on valid leads that have been discovered not only during this initial phase but throughout the entire investigation process.
Narrowing down to a specific activity often results in limited findings, premature conclusions and an incomplete understanding of the overall impact.
### Creation & Usage of IOCs
An IOC is a sign that an incident has occurred. These are documented in a structured manner which represents the artifacts of the compromise.
Examples of IOCs can be IP addresses, hash values of files and file names.
Special languages such as OpenIOC and Yara have been developed to document them and share them in a standard manner.
This means we may even obtain IOCs from third parties if the adversary or the attack is known.
To leverage them, we have to deploy an IOC-obtaining/IOC-searching tool. A common approach is to utilize WMI or PowerShell for IOC-related operations in Windows.
**Be careful**: the credentials of our highly privileged user(s) must not be cached when connecting to (potentially) compromised systems (or any systems, really).
### Identification Of New Leads & Impacted Systems
After searching for IOCs, we expect to have some hits that reveal other systems with the same signs of compromise.
These may not be directly related with the incident we are investigating. we need to identify and eliminate false positives.
We may also end up having a large number of hits, in which case we should prioritize the ones we will focus on, ideally those that can provide us with new leads after a potential forensic analysis.
### Data Collection & Analysis From The New Leads & Impacted Systems
Once identified, we want to collect and preserve the state of the systems that included our IOCs for further analysis.
Live response is the most common approach where we collect a predefined set of data that is usually rich in artifacts that may explain what happened to a system.
Shutting down a system may be bad because much of the artifacts will only live within the RAM memory of the machine.
Regardless, it is vital to ensure minimal interaction with the system occurs to avoid altering any evidence or artifacts.
Once it has been collected, it is time to analyze it. This is the most time-consuming process during an incident.
Malware analysis and disk forensics are the most common examination types.
Keep track of the chain of custody to ensure that the examined data is court-admissible if legal action is to be taken against an adversary.

## Questions
- True or False: Can a third party vendor be a source of detecting a compromise?
	- True

- During an investigation, we discovered a malicious file with an MD5 hash value of 'b40f6b2c167239519fcfb2028ab2524a'. How do we usually call such a hash value in investigations? Answer format: Abbreviation
	- IOC