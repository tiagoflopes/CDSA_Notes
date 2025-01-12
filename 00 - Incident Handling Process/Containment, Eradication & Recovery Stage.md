When the investigation is complete and we have understood the type of incident and the impact on the business, it is time to enter the containment stage to prevent the incident from causing more damage.
## Containment

We want to prevent the spread of the incident. We divide the actions into **short-term containment** and **long-term containment**.
It is important that containment actions are executed across all systems simultaneously.
##### Short-term containment
The actions taken leave a minimal footprint on the systems, like placing a system in a separate/isolated VLAN, pulling the network cable out of the system or modifying the attacker's C2 DNS name to a system under our control or to a non-existing one.
This actions contain the damage and provide time to develop a more concrete remediation strategy. We can also take forensic images and preserve evidence (a.k.a. backup substage of the containment stage).
If it is required to shut down a system, ensure that this is communicated to the business and appropriate permissions are granted.
##### Long-term containment
Focus on persistent actions and changes, like changing user passwords, applying firewall rules, inserting a host IDS, applying a system patch and shutting down systems.
Keep the business and relevant stakeholders updated.

## Eradication

Once it is contained, eradication is needed to eliminate both the root cause of the incident and what is left of it to ensure the adversary is out of the systems and network.
Some activities are removing the detected malware, rebuilding systems and restoring others from backup. We may also extend containment activities by applying additional patches.
Additional system-hardening activities are often performed during this phase (not just on impacted systems, but across the network in some cases).

## Recovery

In the recovery stage, we bring systems back to normal operations. For that, the business needs to verify that a system is in fact working as expected and that it contains all the necessary data.
All restored systems will be subject to heavy logging and monitoring, as compromised systems tend to be targets again is the attacker regains access. Typical suspicious events to monitor:
- unusual logons, like user or service accounts that have never logged in there before
- unusual processes
- changes to the registry in location that are usually modified by malware

This process may take months, since it is approached in phases. during the early phase, the focus is on increasing overall security, while on later phases you focus on permanent changes.

## Questions
- True or False: Patching a system is considered a short term containment.
	- False