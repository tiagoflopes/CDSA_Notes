Our clients generate a lot of this, so abnormalities can sometimes get buried in the mass volume of it. However, understanding it is important.

## DNS Queries

![[dns_query.png]]

##### DNS Reverse Lookups/Queries
Same but opposite lmao :)

## DNS Record Types

![[dns_record_types.png]]

## Finding DNS Enumeration Attempts

Lot's of traffic. Concluding with **ANY** is a clear indication of DNS enumeration and possible subdomain enumeration from the attacker.

## Finding DNS Tunneling

Attackers have utilized DNS forward and reverse lookup queries to perform data exfiltration. They do so by appending the data as a part of the TXT field.

Attackers employ these tunnels for:
- Data Exfiltration
- C2
- Bypassing Firewalls and Proxies
- Domain Generation Algorithms (DGAs)

## The Interplanetary File System and DNS Tunneling

[Interplanetary File System](https://developers.cloudflare.com/web3/ipfs-gateway/concepts/ipfs/)
Attackers use it to store and pull malicious files.
