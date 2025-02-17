The IP layer functions in its ability to transfer packets from one hop to another.
However, it is essential to note that this layer has no mechanism to identify when packets are lost, dropped or tampered with.
To dissect these packets, we can explore some of their fields:
1. Length - Contains the overall length of the IP header.
2. Total Length - Specifies the entire length of the IP packet, including any relevant data.
3. Fragment Offset - Provides instructions on how to reassemble the packet upon delivery to the destination host.
4. Source and Destination IP Addresses - Of the 2 communicating hosts.

## Abuse of Fragmentation

Fragmentation serves as a means for our legitimate hosts to communicate large data sets to one another by splitting the packets and reassembling them upon delivery. This is achieved by setting a maximum transmission unit (MTU), that divides these large packets into equal sizes (except for the last one) to accommodate the entire transmission.
Attackers abuse this field for:
- IPS/IDS Evasion - If our defense system doesn't reassemble fragmented packets, the attacker could split their enumeration techniques to be fragmented and bypass these controls.
- Firewall Evasion - Likewise, if the firewall does not reassemble them, the attacker's enumeration attempt might succeed.
- Resource Exhaustion - Suppose an attacker were to craft their attack to fragment packets to a very small MTU, the network control might not reassemble these packets due to resource constraints.
- Denial of Service - For old hosts, an attacker might send IP packets exceeding 65535 bytes. By doing so, the destination host will reassemble this malicious packer and experience countless different issues.

If our network mechanism were to perform correctly, it should do the following:
- Delayed Reassembly - The IDS/IPS/Firewall should act the same as the destination host, in the sense that it waits for all fragments to arrive to reconstruct the transmission to perform packet inspection.

## Finding Irregularities in Fragment Offsets

We will look into the pcapng. we notice several ICMP request, indicative of the starting requests from a traditional enumeration scan.
##### Attacker's Enumeration
`nmap <host>`

Secondarily, as attacker might define a maximum transmission unit size.
`nmap -f 10 <host>`
The wireshark scan will show each fragment and their reassemble.

All the RST flags that are sent to the attacker when the port is closed indicate this is a fragmentation attack.
