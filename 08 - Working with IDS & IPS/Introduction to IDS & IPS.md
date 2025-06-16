The purpose of Intrusion Detection/Prevention Systems is not only to identify potential threats, but also to mitigate their impact.

An **IDS** is a device or app that monitors network or system activities for malicious activities or policy violations and produces reports to a management station. They don't prevent an intrusion, but alert us when one occurs.

It operates in 2 main modes:
- signature-based detection - it recognizes bad patters, such as malware signatures and previously identified attack patterns. Limited to known threats.
- anomaly-based detection - it establishes a baseline of normal behavior and sends an alert when it detects deviations from this baseline. Susceptible to false positives.

An **IPS** sits directly behind the firewall and provides an additional layer of protection. It does not just passively monitor network traffic, but actively prevents any detected potential threats.

It also operates in a similar mode to IDS. Once a potential threat is detected, it takes actions such as dropping malicious packets, blocking network traffic and resetting the connection. The goal is to interrupt or halt activities that are deemed dangerous to our network or against our policy.

Both are typically integrated at different points, each having its optimal place depending on its function and the overall network design. They are generally placed behind the firewall, closer to the resources they protect. They can also be implemented on the host level, known as HIDS and HIPS. Their placement is an integral part of a defense-in-depth strategy.

## IDS/IPS Updates

To ensure these perform at their best, we need to consistently update them with the last threat signatures and fine-tune their anomaly detection algorithms.

## Coming Up

We will explore the fundamentals of **Suricata**, **Snort** and **Zeek**.
