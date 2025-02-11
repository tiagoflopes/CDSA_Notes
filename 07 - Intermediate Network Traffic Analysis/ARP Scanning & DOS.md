Adversaries can also exploit ARP for information gathering.

## ARP Scanning Signs

Some red flags indicative of ARP scanning are:
1. Broadcast ARP requests sent to sequential IP addresses.
2. Broadcast ARP requests sent to non-existent hosts.
3. Potentially, an unusual volume of ARP traffic originating from a malicious or compromised host.

## Finding ARP Scanning

In the example given, we find ARP requests being propagated by a single host to all IP addresses in a sequential manner. This is symptomatic of ARP scanning and is a common feature of scanners such as Nmap.
![[ARP_Scanning_sequential.webp]]

## Identifying DOS

Upon acquiring a list of live hosts, the attacker might alter their strategy to deny services to all these machines. Essentially, they will strive to contaminate an entire subnet and manipulate as many ARP caches as possible.
This contamination is done by declaring new physical addresses for all live IP addresses.

## Responding To ARP Attacks

Upon identifying any of these anomalies, we can:
1. Trace and Identify - The attacker's machine is a physical entity located somewhere. If we manage to locate it, we could potentially halt its activities.
2. Contain - To stymie any further exfiltration of information by the attacker, we might contemplate disconnecting or isolating the impacted area at the switch or router level.
