## Threat Hunting Definition

Threat hunting is an active, human-led and often hypothesis-driven practice that systematically combs through network data to identify stealth, advanced threats that evade existing security solutions.
Its objective it to significantly reduce the dwell time (duration between the breach and its detection).
This process starts with the identification of assets that could be high-value targets. Next we analyze the TTPs adversaries are likely to employ. Then we strive to proactively detect, isolate and validate any artifacts related to the abovementioned TTPs.
We regular employ Threat Intelligence, a vital component that aids in formulating effective hunting hypotheses, developing counter-tactics and executing protective measures.
Key facets of threat hunting include:
- Proactive strategy that prioritizes threat anticipation over reaction, based on hypotheses, attacker TTPs and intelligence.
- Reactive response that searches across the network for artifacts to a verified incident, based on evidence and intelligence.
- A solid, practical comprehension of threat landscape, cyber threats, adversarial TTPs and the cyber kill chain.
- Cognitive empathy with the attacker.
- A profound knowledge of the organization's IT environment, network topology, digital assets and normal activity.
- Utilization of high-fidelity data and tactical analytics, and leveraging advanced threat hunting tools and platforms.

## The Relationship Between Incident Handling & Threat Hunting

How does it intersect with the various phases of Incident Handling?
- In the **Preparation** phase, a threat hunting team must set up robust, clear rules of engagement, operational protocols must be established, outlining when and how to intervene, the course of action in any scenario.
- During the **Detection & Analysis** phase, they can augment investigations, ascertain whether the observed IoCs truly signify an incident and further, their adversarial mindset can help uncover additional artifacts that might have been missed initially.
- In the **Containment, Eradication and Recovery** phase, some organizations might expect hunters to perform tasks within the phase, some don't.
- Regarding the **Post-Incident Activity** phase, their extensive expertise spanning various IT domains and IT Security can contribute significantly.
Whether these processes should be integrated or function independently is a strategic decision, contingent upon each organization's unique threat landscape, risk, etc.

## Team's Structure

It is crucial that each team member offers a unique set of competencies that when combined provide a holistic and comprehensive approach to identifying, mitigating and elimination threats.
The ideal threat hunting team composition typically includes:
- Threat Hunter - The core role within the team. They are cybersecurity professionals with a deep understanding of the threat landscape, adversarial's TTPs and sophisticated threat detection methodologies. They proactively search for IoCs and are proficient in using a variety of threat hunting tools and platforms.
- Threat Intelligence Analyst - These are responsible for gathering and analyzing data from a variety of sources, including open-source intelligence, dark web intelligence, industry reports and threat feeds. Their job is to understand the current threat landscape and predict future trends.
- Incident Responders - When threat hunters identify potential threats, these step in to manage the situation. They investigate the incident thoroughly and they are also responsible for containment, eradication and recovery actions.
- Forensics Experts - These delve deep into the technical details of an incident. They are proficient in digital forensics and incident response (DFIR), capable of analyzing malware, reverse engineering attacks and providing detailed incident reports.
- Data Analysts/Scientists - They examine large datasets to uncover patterns, correlation and trends that can lead to actionable insights for threat hunters.
- Security Engineers/Architects - These are responsible for the overall design of the organization's security infrastructure. They ensure everything is designed with security in mind and they often implement tools and techniques that facilitate threat hunting.
- Network  Security Analyst - These specialize in network behavior and traffic patterns. They understand the normal ebb and flow of network activity and can quickly identify anomalies indicative of a potential security breach.
- SOC Manager - They oversee the operations of the threat hunting team, ensuring smooth coordination among team members and effective communication with the rest of the organization.

## When Should We Hunt?

Threat hunting should not be seen as a sporadic or reactionary practice, but rather as a sustained, forward-thinking activity. Nevertheless, there are specific instances that call for an immediate and intense threat hunting operation. Here's a breakdown of these:
- *When new information on an adversary or vulnerability comes to light* - The landscape is always evolving, and it's imperative to decipher the adversary's modus operandi and scrutinize the new vulnerability.
- *When new indications are associated with a known adversary* - Intelligence sources often release new IoCs tied to specific adversaries.
- *When multiple network anomalies are detected* - Can be caused by system glitches or valid alterations, or a systemic issue or an orchestrated attack.
- *During an incident response activity* - Simultaneously conduct threat hunting across the network. Expose any connected threat that might not be readily visible.
- *Periodic proactive action* - Regular, proactive threat hunting exercises are key to discovering latent threat.

In a nutshell, the ideal time for threat hunting is always the present.

## The Relationship Between Risk Assessment & Threat Hunting

Risk assessment allows us to prioritizes our hunting activities and focus our efforts on the areas of greatest potential impact.
It involves a series of steps including asset identification, threat identification, risk determination and risk mitigation strategy formulation.
In threat hunting, the information gleaned from a through risk assessment can guide our activities:
- Prioritizing Hunting Efforts - By recognizing the most critical assets (a.k.a. 'crown jewels') and their risks.
- Understanding Threat Landscape - This understanding assists us in developing our hunting hypotheses.
- Highlighting Vulnerabilities - This enables us to look for exploitation indicators in these areas.
- Informing the Use of Threat Intelligence - By identifying the most likely threat actors and their preferred methods of attack.
- Refining Incident Response Plans - It helps us anticipate and plan for potential breaches, ensuring a swift and effective reponse.
- Enhancing Cybersecurty Controls - Further strengthening the organization's security posture.

The technicalities of employing risk assessment for threat hunting include the use of advanced tools and techniques such as automates vulnerability scanners, SIEM systems, etc.
In essence, risk assessment and threat hunting are deeply intertwined, each augmenting the other to create a more robust and resilient cybersecurity posture.
