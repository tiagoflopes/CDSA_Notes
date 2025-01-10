As defined by NIST, the incident handling process consists of the following distinct stages:
![[handling_process.webp]]

Incident handlers spend most of their time in the first two stages. When a malicious event is detected, we then move to the next stage and respond to the event (there should always be resources operating on the first two stages though)
It is not linear, it's cyclical. It is vital to ensure that you don't skip steps in the process and that you complete a step before moving onto the next one. This can result in unpredictable consequences

There are however 2 main activities:
- Investigating - aims to discover the initial "patient zero" victim and create an incident timeline, determine what tools and malware the adversary used and document the compromised systems and what the adversary has done
- Recovering - following the investigation, this involves creating and implementing a recovery plan

When an incident is fully handled, a report is issued that details the cause and cost of the incident. *Lessons Learned*  activities are performed to understand how to prevent incident of similar type

These will be studied in separate notes:
- [[Preparation Stage]]
- [[Detection & Analysis Stage]]
- [[Containment, Eradication & Recovery Stage]]
- [[Post-Incident Activity Stage]]
## Questions
- True or False: Incident handling contains two main activities. These are investigating and reporting.
	- False