The Address Resolution Protocol (ARP) has been a longstanding utility exploited by attackers to launch MITM and DOS attacks.
Many of these attacks are broadcasted, making them more readily detectable through our packet sniffing techniques.

## How Address Resolution Protocol Works

This protocol serves the purpose of finding the physical address (MAC address) of devices on the network.

## ARP Poisoning & Spoofing

Detecting these attacks can be challenging as they mimic the communication structure of standard ARP traffic. Yet, certain ARP requests and replies can reveal their nefarious nature.
![[ARP-spoofing-poisoning.webp]]

Detecting these attacks is one aspect, but averting them is a whole different challenge. We can potentially fend off these attacks with controls such as:
1. Static ARP Entries - This disallows easy rewrites and poisoning of the ARP cache. This, however, requires increased maintenance and oversight.
2. Switch and Router Port Security - Ensure that only authorized devices can connect to specific ports on our network devices.

## Finding ARP Spoofing

First, let's reduce the noisy by applying the _arp.opcode_ filter to focus solely on ARP requests and replies.
An anomaly that we instantly see is the fact that there's one host incessantly broadcasting ARP requests and replies.
To ascertain this, we can fine-tune our analysis to inspect solely the interactions among the attacker's machine, the victim's machine and the router (Opcode 1 is requests and 2 is replies).
When we focus on the requests, we see 2 packets, and an address duplication. This means that one IP address is mapped to two different MAC addresses. We would validate this by checking the machine's cache.
```shell
arp -a | grep 50:eb:f6:ec:0e:7f
arp -a | grep 08:00:27:53:0c:ba
```
Our cache would show both MAC addresses allocated to the same IP address.

## Identifying the Original IP Address

So what were the initial IP addresses of these devices? Knowing this will aid us in determining which device altered its IP address through MAC spoofing.
We can find that out using a filter: `(arp.opcode) && ((eth.src == 08:00:27:53:0c:ba) || (eth.dst == 08:00:27:53:0c:ba))`.
We see that the MAC address was first linked to 192.168.10.5, and then changed to 192.168.10.4. This transitions is indicative of a deliberate attempt at ARP spoofing or cache poisoning.
