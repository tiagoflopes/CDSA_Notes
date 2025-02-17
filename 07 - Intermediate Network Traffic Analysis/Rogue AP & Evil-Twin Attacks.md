In the realm of malevolent access points, rogue and evil-twin attacks invariably surface as significant concerns.
A rogue access point primarily serves as a tool to circumvent perimeter controls in place, to provide unauthorized access to restricted sections of a network. They are directly connected to the network.

## Evil-Twin

This poses many other different purposes. The key factor is that in most cases these APs aren't connected to our network.
Attackers might set these up to harvest wireless or domain passwords among other pieces of information. Commonly, these attacks might also encompass a hostile portal attack.

## Airodump-ng Detection

Through airodump-ng, if we select our target's ESSID, we will find an evil-twin with the same ESSID.
Deauthentication attempts are also indicative of enforcement measures from the attacker operating their AP.
We can also analyze the traffic on wireshark, using the filters: `wlan.fc.type == 0 and wlan.fc.type_subtype == 8`.
To differentiate them, we must look to the Robust Security Network (RSN) information (our organizations' contain information such as its authentication mechanism).

## Finding a Fallen User

Some users may fall prey to attacks like these. In the case of open network style evil-twin attacks, we can view most higher-level traffic in an unencrypted format.
Just view the traffic with the attacker's AP BSSID: `wlan.bssid == ...`.
If we see anything, record pertinent details about the client device to further our incident response efforts.
1. Its MAC Address.
2. Its Host Name.

## Finding Rogue APs

On the other hand, finding rogue APs can often be a simple task of checking our network device lists.
If we encounter an unrecognizable wireless network with a string signal, particularly if it lacks encryption, this could indicate that a user has established a rogue AP to navigate around our perimeter controls.
