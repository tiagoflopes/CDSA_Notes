## Telnet

Telnet is a network protocol that allows a bidirectional interactive communication sessions between 2 devices over the network.
It may still be used by attackers for malicious purposes such as data exfiltration and tunneling.

## Finding Traditional Telnet Traffic Port 23

Traffic under port 23. It usually is decrypted and easily inspectable, but attackers may encrypt, encode or obfuscate the text.

## Unrecognized TCP Telnet in Wireshark

It can also happen on other ports, of course. 23 is just the default one.

## Telnet Protocol through IPv6

Unless our local network is configured to utilize IPv6, observing its traffic is an indicator of bad actions within our environment.

## Watching UDP Communications

As we already know, UDP is faster and connectionless. Excellent for exfiltrations.

## Common Uses of UDP

We might find legitimate traffic that uses UDP like the following:
1. Real-time Applications
2. DNS
3. DHCP
4. SNMP
5. TFTP
