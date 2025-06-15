I have 2 PCAP files. I need to inspect them and find which attack was performed in each.
I have some options for each PCAP:
- DNS Flooding - high rate of DNS request packets (UDP port 53), same source IP, short intervals,  absent responses.
- DNS Amplification - DNS responses from many servers to a single IP, larger responses, ANY type, no request by the victim.
- DNS Tunneling - long DNS query names, frequent to a single domain, consistent packet sizes and patterns.
- ICMP Flooding - high rate of echo requests (type 8).
- ICMP Tunneling - unusual payload sizes and frequent back-and-forth traffic.
- ICMP SMURF Attack - victim receives numerous echo replies without asking for it.
