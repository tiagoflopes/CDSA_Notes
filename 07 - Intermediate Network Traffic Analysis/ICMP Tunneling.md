This is a technique employed by adversaries in order to exfiltrate data from one location to another. There are many different kinds of tunneling, and each used a different protocol. Commonly, attackers may use proxies to bypass our network controls, or protocols that our systems and controls allow.

## Basics of Tunneling

We might notice tunneling through the attacker possessing some C2 over one of our machines. One of the more common types is SSH tunneling, but proxy-based, HTTP, HTTPS, DNS and other types can be observed in similar ways.
![[basic-tunnel-1.webp]]

## ICMP Tunneling

In the case of this type, an attacker will append data they want to exfiltrate in the data field in an ICMP request.

##### Finding ICMP Tunneling
We can find it by looking at the contents of data per request and reply.
Suppose we noticed fragmentation occurring within our ICMP traffic. This would indicate a large amount of data being transferred via ICMP.
A normal ICMP request will contain some reasonable value like 48 bytes for the data field. However, a suspicious request might have a large data length like 38000 bytes.
Then, we can just see the content of that data, that might be encoded or encrypted.
Any value larger than 48 bytes for the data length should be analysed.

## Preventing ICMP Tunneling

1. Block ICMP Requests
2. Inspect ICMP Requests and Replies for Data
