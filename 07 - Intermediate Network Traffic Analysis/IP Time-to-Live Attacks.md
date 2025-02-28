These attacks are used as a means of evasion by attackers.
Basically it's to escape firewalls by only reaching the desired destination.

##### Finding Irregularities in IP TTL
Find these in scanning services (incremental port numbers).
For any packet, if we inspect it, we will find a low (1-3) TTL.

We can prevent/control this behavior by discarding or filtering packets that do not have a high enough TTL.
