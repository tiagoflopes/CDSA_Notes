We should always consider the following when analyzing these fields for our traffic analysis efforts.
1. The source IP address should always be from our subnet.
2. The source IP for outgoing traffic should always be from our subnet.

An attacker might conduct these packet crafting attacks towards the source and destination IP addresses for many different reasons or desired outcomes:
1. Decoy Scanning - Through changing the source to something within the same subnet as the target host, the attacker might succeed in firewall evasion.
2. Random Source Attack DDoS - An attacker might be able to send tons of traffic to the same port on the victim host.
3. LAND Attacks - These operate similarly to the above mentioned in the nature that the source address is set to the same as the destination hosts.
4. SMURF Attacks - Similar to those 2, these work through the attacker sending large amounts of ICMP packets to many different hosts, with the source address being the victim's machine.
5. Initialization Vector Generation - In older wireless networks, an attacker might capture, decrypt, craft and re-inject a packet with a modified source and destination in order to get IVs to build a decryption table.

## Finding Decoy Scanning Attempts

In the case of decoy scanning, we will notice:
1. Initial Fragmentation from a fake address.
2. Some TCP traffic from the legitimate source address.

To prevent these, we can:
1. Have our IDS/IPS/Firewall act as the destination host would.
2. Watch for connection started by one host, and taken over by another.

## Finding Random Source Attacks

Random source attacks can be done like the opposite of a SMURF attack, in which many hosts will ping one host which does not exist, and the pinged host will ping back all others and get no reply.
We should also consider that attackers might fragment these random hosts communications in order to draw out more resource exhaustion.
However in many cases, like LAND attacks, these attacks will be used by attackers to exhaust resources to one specific service on a port. Instead of spoofing them, the attacker might randomize them.
In this case, we have a few indicators of nefarious behavior:
1. Single Port Utilization from random hosts
2. Incremental Base Port with a lack of randomization.
3. Identical Length Fields

## Finding Smurf Attacks

Simply put, an attacker conducts these attacks like the following:
1. The attacker will send an ICMP request to live hosts with a spoofed address of the victim host.
2. The live hosts will respond to the legitimate victim host with an ICMP reply.
3. This may cause resource exhaustion on the victim host.

Fragmentation elevates the amount of noise.

## Finding LAND Attacks

These attacks operate through an attacker spoofing the source IP address to be the same as the destination. Quite easy to detect them.
