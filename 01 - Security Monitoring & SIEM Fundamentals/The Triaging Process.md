## What Is Alert Triaging?

Performed by a SOC analyst, is the process of evaluating and prioritizing security alerts generated by various monitoring and detection systems to determine their level of threat and potential impact on a organization.
**Escalation** is a process that involves notifying supervisors, incident response teams or designated individuals within the organization who have the authority to make decisions and coordinate the response effort. The analyst provides detailed information, including its severity, potential impact and any relevant finding from the initial investigation.

## What Is The Ideal Triaging Process?

1. **Initial Alert Review** 
	- Thoroughly review the initial alert, including metadata, timestamp, source IP, destination IP, affected systems and triggering rule/signature.
	- Analyse associated logs to understand the alert's context.
2. **Alert Classification**
	- Classify the alert based on severity, impact and urgency using the predefined classification system. 
3. **Alert Correlation**
	- Cross-reference the alert with related alerts, events or incidents to identify patterns, similarities or potential IOCs.
	- Query the SIEM/log management system to gather relevant log data.
	- Leverage threat intelligence feeds to check for known attack patters or malware signatures.
4. **Enrichment of Alert Data**
	- Gather additional information to enrich the alert data and gain context (network packet captures, memory dumps, reconnaissance).
5. **Risk Assessment**
	- Evaluate the potential risk and impact to critical assets, data or infrastructure (consider regulatory implications).
6. **Contextual Analysis**
	- The analyst considers the context surrounding the alert, including the affected assets, their criticality and the sensitivity of the data they handle.
	- Evaluate the security controls in place, such as firewalls, IDS/IPS and EDR.
	- Assess the relevant compliance requirements.
7. **Incident Response Planning**
	- Initiate an IRP if the alert is significant.
8. **Consultation With IT Operations**
	- Assess the need for additional context or missing information by consulting with IP operations or relevant departments.
9. **Response Execution**
	- Determine the appropriate response action based on the alert review, risk assessment and consultation. If the alert still indicates potential security concerns, proceed with the IR actions.
10. **Escalation**
	- Identify triggers for escalation based on organization's policies and alert severity.
	- Assess the alert against escalation triggers, considering potential consequences if not escalated.
	- Follow internal escalation process.
	- Provide comprehensive alert summary, severity, potential impact, enrichment data and risk assessment.
	- Document all communication related to escalation.
	- In some cases, escalate to external entities, based on legal/regulatory requirements.
11. **Continuous Monitoring**
	- Continuously monitor the situation and incident response progress.
	- Maintain open communication with escalated teams, providing updates on developments, findings or changes in severity/impact.
	- Collaborate closely with escalated teams for a coordinated response.
12. **De-escalation**
	- Evaluate the need for de-escalation as the IR progresses and the situation is under control.
	- De-escalate when the risk is mitigated, incident is contained and further escalation is unnecessary.
	- Notify relevant parties, providing a summary of actions taken, outcomes and lesson learned.
