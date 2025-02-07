NTA is a dynamic process that can change depending on the tools we have on hand, permissions given to us by the organization and our network's visibility.
It's a detailed examination of an event or process, determining its origin and impact, which can be used to trigger specific precautions and/or actions to support or prevent future occurrences.
In more advanced implementations for NTA that include other tools like IDS/IPS, firewalls, host and network logs, and additional information being fed into Tools like Splunk or ELK Stack, having the ability to monitor traffic is invaluable. The tools help us quickly alert on malicious actions happening. Many defensive tools have signatures built for most of the common attacks and toolkits.
Lastly, this is a dynamic skill, and using automated tools to aid us is perfectly fine. Just do not rely on them solely. The human eye is still our best resource for finding the bad.

## Analysis Dependencies

Traffic capturing can be performed actively or passively. The table below lays out the dependencies for each.
![[traffic_capture_dependecies.png]]
The last one is more of a recommendation than a requirement
