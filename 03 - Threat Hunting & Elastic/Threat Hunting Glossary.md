- **Adversary** - Refers to an entity driven by shared objectives as your organizations, albeit unauthorized, seeking to infiltrate your business and satisfy their collection requirements, which may include financial gains, insider information or valuable intellectual property. These can be classified into *cyber criminals*, *insider threats*, *hacktivists* or *state-sponsored operators*.

- **Advanced Persistent Threat (APT)** - Associated with highly organized groups or nation-wide entities that possess extensive resources, thereby enabling them to carry out their malicious activities over prolonged periods. They show a marked preference for high-value targets (governmental organizations, healthcare infrastructure and defense systems). The 'Advanced' aspect refers to the sophisticates strategic planning and 'Persistent' alludes to their dogged persistence in achieving their objectives, backed by substantial resources, including financial banking, manpower and time.

- **Tactics, Techniques and Procedures (TTP)** - Borrowed from the military, symbolize the distinct operational patterns or 'signature' of an adversary.
	- *Tactics* - The strategic objectives and high-level concepts of operations employed by the adversary.
	- *Techniques* - The specific methods utilized by an adversary to accomplish their tactical objectives. 
	- *Procedures* - The granular, step-by-step instructions for the implementation of each technique.

- **Indicator** - Encompasses both technical data and contextual information.

- **Threat** - A multifaceted concept, consisting of three fundamental factors:
	- *Intent* - The underlying rationale driving adversaries to target and exploit your network infrastructure.
	- *Capability* - Denotes the tools, resources, financial banking and skill level that adversaries possess to carry out their operations successfully.
	- *Opportunity* - Refers to conditions or events that provide favorable circumstances for adversaries to execute their operations.

- **Campaign** - Refers to a collection of incidents that share similar TTPs and are believed to have comparable collection requirements. This type of intelligence necessitates substantial time and effort to aggregate and analyze.

- **Indicators of Compromise (IoCs)** - Digital traces or artifacts derived from active or past intrusions. They serve as 'signposts' of a specific adversary or malicious activity. These can include hashes of malicious files, suspicious IP addresses, URLs, domain names and names of malicious executables or scripts.

- **Pyramid of Pain** - A critical visualizations which presents a hierarchy of indicators that can support us in detecting adversaries. It also showcases the degree of difficulty in acquiring these specific indicators and the subsequent impact of gathering intelligence on them. The lower in the list you go, the harder it is to find, but the more intelligence you get.
	- *Hash Values* - Digital fingerprints of files. Malware binaries can be identified through their unique hash values. Obviously, one small change results in a completely different hash, so less reliable.
	- *IP Addresses* - Unique identifiers for devices on a network. Used to track the source of network traffic or a potential attack. IP spoofing, VPNs, proxies or TOR networks hide these though.
	- *Domain Names* - Used to identify 1+ IP addresses. Malicious actors often use domain generation algorithms (DGAs) to produce a large number of pseudo-random domain names to evade detection.
	- *Network/Host Artifacts*
		- Network Artifacts - Residual traces of an attacker's activities within the network infrastructure. Hard to modify without impacting the effectiveness or stealth of their operation.
		- Host Artifacts - Remnants of malicious activity left on individual systems or endpoints. Also hard to alter without affecting their intrusion campaign or revealing their presence.
	- *Tools* - Software used by adversaries to conduct their attacks. Sophisticated adversaries use custom tools or modify existing ones to evade detection.
	- *TTPs* - These are the most difficult for an adversary to change without significant cost and effort. Examples are spear-phishing, exploitation of software vulnerability and the steps to achieve that.

- **Diamond Model** - Conceptual framework designed to illustrate the fundamental aspects of a cyber intrusion.
![[diamond_model.webp]]
	- *Adversary* - The individual, group or organizations responsible for the cyber intrusion. It's important to understand their capabilities, motivations and intent to effectively defend against their attacks.
	- *Capability* - The TTPs that the adversary used to carry out the intrusion. Malware, exploits and other malicious tools, as well as the specific methods used to deploy these tools.
	- *Infrastructure* - The physical and virtual resources that the adversary used to facilitate the intrusion. Servers, domains, IP addresses and other, used to deliver malware, control compromised systems or exfiltrate data.
	- *Victim* - The target of the intrusion, individual, group or organizations. Understanding the victim's vulnerabilities, value of their assets and exposure is crucial for effective defense.

Overall, the Diamond Model provides a complementary perspective to the Cyber Kill Chain, offering a different lens through which to understand and respond to cyber threats.
