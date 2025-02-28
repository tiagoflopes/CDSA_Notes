When attackers are gaining information on our TCP services, we might notice a few odd behaviors during our traffic analysis efforts.
We already know how TCP connections are established. Let's keep in mind there are other flags to consider.
![[tcp_flags.png]]

As such, we can look for the following conditions:
- Too many flags of a kind or kinds.
- The usage of different and unusual flags.
- Solo host to multiple ports, or solo host to multiple hosts.

## Excessive SYN Flags

This is a prime example of nmap scanning.
There are however 2 primary scan types that use the SYN flag:
1. SYN Scans - the attacker will pre-emptively end the handshake with the RST flag.
2. SYN Stealth Scans - the attacker partially completes the TCP handshake.

## No Flags

The attacker may send no flags. This is a NULL scan. In this scan, TCP connections behave like this:
- If the port is open - the system will not respond at all since there is no flags.
- If the port is closed - the system responds with an RST packet.

## Too Many ACKs

On the other hand, the attacker might be employing an ACK scan, where we notice an excessive amount of acknowledgements between two hosts. The connection behaves like this:
- If the port is open - no response or RST packet.
- If the port is closed - RST packet.

## Excessive FINs

Another attack, a FIN scan. Behaves like this:
- If the port is open - no response.
- If the port is closed - RST packet.

## Just too many flags

"Throw spaghetti at the wall". Use a Xmas tree scan (all TCP flags). The response is:
- If the port is open - no response, or RST packet.
- If the port is closed - RST packet.

These are easy to spot.
