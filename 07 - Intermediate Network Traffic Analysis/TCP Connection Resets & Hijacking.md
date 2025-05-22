TCP does not provide the level of protection to prevent our hosts from having their connections terminated or hijacked by an attacker. As such, we might notice that a connection gets terminated by an RST packet, or hijacked.

## TCP Connection Termination

To cause DOS conditions within our network, attackers may employ a simple TCP RST Packet injection attack, a.k.a. TCP connection termination.
This combines a few conditions:
1. The attacker spoofs the source address to be the affected machine's.
2. The attacker modifies the TCP packet to contain the RST flag.
3. The attacker specifies the destination port to be the same as one currently in use by one of our machines.

It might look something like this.
![[tcp_termination.webp]]

This can be detected by checking the arp tables and seeing if an IP address is registered to a different MAC address that the one it is showing in the pcap. However, the attacker might spoof their MAC address too. But in this case, we would notice retransmissions and other issues as we saw in the ARP poisoning section.

## TCP Connection Hijacking

This is a more advanced attack, where the attacker actively monitors the target connection they want to hijack.
They conduct sequence number prediction in order to inject their malicious packets in the correct order. The source address is spoofed to be the same as our affected machine.
In order to continue the hijacking, they need to block ACKs from reaching the affected machine, by either delaying or blocking the ACK packets. As such, this attack is very commonly employed with ARP poisoning, and we might notice somethink like this.
![[tcp_hijacking.webp]]
