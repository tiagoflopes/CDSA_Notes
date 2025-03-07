## Cyber Threat Intelligence Definition

Cyber Threat Intelligence (CTI) provides essential insights to fortify our defenses against cyberattacks. The primary objective of our CTI team is to transition our defense strategies from merely reactive measures to a more proactive anticipatory stance. They contribute crucial insights to our SOC.
Four fundamental principals make CTI an integral part of our cybersecurity strategy:
- Relevance - The true value of information lies in its relevance to our organization. If there is a reported vulnerability in a software that we don't use, the urgency to implement defensive measures diminishes.
- Timeliness - The value of information depreciates over time. Aged indicators lose their relevance.
- Actionability - Data under analysis should yield actionable insights for our defense team. Unactionable intelligence can lead to a self-perpetuating cycle of non-productive analysis, a.k.a. "self-licking ice cream cone".
- Accuracy - Incorrect indicators, misattributions or flawed TTPs can result in wastage of valuable time and resources.

When these 4 factors synergize, the intelligence gleaned allows us to:
- Gain insights into potential adversary operations and campaigns that might be targeting our organization.
- Enrich our data pool though analysis by CTI analysts and other network defenders.
- Uncover adversary TTPs, enabling the development of effective mitigation measures and enhancing our understanding of adversary behavior.
- Provide decision-makers within our organization with pertinent information for informed, impactful decision-making related to business operations.

## The Difference Between Threat Intelligence & Threat Hunting

These are 2 distinct yet intrinsically interconnected specialties within the realm of cybersecurity.
##### Threat Intelligence (Predictive)
The primary aim here is to anticipate the adversary's moves, ascertain their targets and discern their methods of information acquisition. Our mission is to predict:
- The location of the intended attack.
- The timing of the attack.
- The operational strategies the adversary will employ.
- The ultimate objectives of the adversary.
##### Threat Hunting (Reactive and Proactive)
An initiating event or incident prompts our team to launch an operation to ascertain whether an adversary is present in the network, or if one was present and evaded detection.

## Criteria Of Cyber Threat Intelligence

As the information gathered is compiled, it transforms into intelligence, which can then be classified into *Strategic*, *Operational* or *Tactical* Intelligence.

**Strategic Intelligence** is characterized by:
- Being consumed by C-suite executives, VPs, and other company leaders.
- Aiming to align intelligence directly with company risks to inform decisions.
- Providing an overview of the adversary's operations over time.
- Mapping TTPs and Modus Operandi (MO) of the adversary.
- Striving to answer the Who? and Why?

**Operational Intelligence** is characterized by:
- Also including TTPs of an adversary.
- Providing information on adversary campaigns.
- Offering more detail than what's found in strategic intelligence reports.
- Being produced for mid-level management personnel.
- Working towards answering the How? and Where?

**Tactical Intelligence** is characterized by:
- Delivering immediate actionable information.
- Being provided to network defenders for swift action.
- Including technical details on attacks that have occurred or could occur in the near future.

## How To Go Through A Tactical Threat Intelligence Report

Let's delve into a procedural, in-depth process using a theoretical scenario involving a threat intelligence report on an elaborate Emotet malware campaign:

- **Comprehending the Report's Scope and Narrative**: The initial phase of interpreting the report involves comprehending its broader context. Suppose our report elucidates an ongoing Emotet campaign directed towards businesses in our sector. The report may offer macro-level insights about the attackers' objectives and the types of entities in their crosshairs. By grasping the narrative, we can assess the pertinence of the threat to our own business.
    
- **Spotting and Classifying the IOCs**: Tactical intelligence typically encompasses a list of IOCs tied to the threat. In the context of our Emotet scenario, these might include IP addresses linked to C2 servers, file hashes of the Emotet payloads, email addresses or subject lines leveraged in phishing campaigns, URLs of deceptive websites, or distinct Registry alterations by the malware. We should partition these IOCs into categories for more comprehensible understanding and actionable results: Network-based IOCs (IPs, domains), Host-based IOCs (file hashes, registry keys), and Email-based IOCs (email addresses, subject lines). Furthermore, IOCs could also contain Mutex names generated by the malware, SSL certificate hashes, specific API calls enacted by the malware, or even patterns in network traffic (such as specific User-Agents, HTTP headers, or DNS request patterns). Moreover, IOCs can be augmented with supplementary data. For instance, IP addresses can be supplemented with geolocation data, WHOIS information, or associated domains.
    
- **Comprehending the Attack's Lifecycle**: The report will likely depict theTTPs deployed by the attackers, correspondingly mapped to the MITRE ATT&CK framework. For the Emotet campaign, it might commence with a spear-phishing email (Initial Access), proceed to execute the payload (Execution), establish persistence (Persistence), execute defense evasion tactics (Defense Evasion), and ultimately exfiltrate data or deploy secondary payloads (Command and Control). Comprehending this lifecycle aids us in forecasting the attacker's moves and formulating an effective response.
    
- **Analysis and Validation of IOCs**: Not all IOCs hold the same utility or accuracy. We need to authenticate them, typically by cross-referencing with additional threat intelligence sources or databases such as VirusTotal or AlienVault's OTX. We also need to contemplate the age of IOCs. Older ones may not be as pertinent if the attacker has modified their infrastructure or tactics. Moreover, contextualizing IOCs is critical for their correct interpretation. For example, an IP address employed as a C2 server may also host legitimate websites due to IP sharing in cloud environments. Analysts should also consider the source's reliability and whether the IOC has been whitelisted in the past. Ultimately, understanding the false positive rate is crucial to avoid alert fatigue.
    
- **Incorporating the IOCs into our Security Infrastructure**: Once authenticated, we can integrate these IOCs into our security solutions. This might involve updating firewall rules with malicious IP addresses or domains, incorporating file hashes into our EDR solution, or creating new IDS/IPS signatures. For email-based IOCs, we can update our email security gateway or anti-spam solution. When implementing IOCs, we should consider the potential impact on business operations. For example, blocking an IP address might affect a business-critical service. In such cases, alerting rather than blocking might be more appropriate. Additionally, all changes should be documented and approved following change management procedures to maintain system integrity and avoid unintentional disruptions.
    
- **Proactive Threat Hunting**: Equipped with insights from the report, we can proactively hunt for signs of the Emotet threat in our environment. This might involve searching logs for network connections to the C2 servers, scanning endpoints for the identified file hashes, or checking email logs for the phishing email indicators. Threat hunting shouldn't be limited to searching for IOCs. We should also look for broader signs of TTPs described in the report. For instance, Emotet often employs PowerShell for execution and evasion. Therefore, we might hunt for suspicious PowerShell activity, even if it doesn't directly match an IOC. This approach aids in detecting variants of the threat not covered by the specific IOCs in the report.
    
- **Continuous Monitoring and Learning**: After implementing the IOCs, we must continually monitor our environment for any hits. Any detection should trigger a predefined incident response process. Furthermore, we should utilize the information gleaned from the report to enhance our security posture. This could involve user education around the phishing tactics employed by the Emotet group or improving our detection rules to catch the specific evasion techniques employed by this malware. While we should unquestionably learn from each report, we should also contribute back to the threat intelligence community. If we discover new IOCs or TTPs, these should be shared with threat intelligence platforms and ISACs/ISAOs (Information Sharing and Analysis Centers/Organizations) to aid other organizations in defending against the threat.
