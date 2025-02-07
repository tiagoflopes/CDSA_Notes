NTA can be described as the act of examining network traffic to characterize common ports and protocols utilized, established a baseline for our environment, monitor and respond to threats and ensure the greatest possible insight into our organization's network.
It can also facilitate the process of meeting security guidelines. Everyday use cases of NTA include:
- Collecting real-time traffic within the network to analyze upcoming threats.
- Setting a baseline for day-to-day network communications.
- Identifying and analyzing traffic from non-standard ports, suspicious hosts and issues with networking protocols such as HTTP errors, problems with TCP or other misconfigurations.
- Detecting malware in the wire, such as ransomware, exploits and non-standard interactions.

## Environment and Equipment

Each of the following provides a different way to capture or dissect the traffic.
![[common_traffic_analysis_tools.png]]

## Performing Network Traffic Analysis

Performing analysis can be as simple as watching live traffic roll by in our consoles, or as complex as capturing data with a tap, sending it back to a SIEM for ingestion, and analyzing the pcap data for signatures and alerts related to common TTPs.

#### NTA Workflow
Performing traffic analysis can distill down to a few basic tenants:
1. Ingest Traffic - Once we have decided on our placement, begin capturing traffic.
2. Reduce Noise by Filtering - Once we complete the initial capture, an attempt to filter out unnecessary traffic from our view can make analysis easier.
3. Analyze and Explore - Now is the time to start carving out data pertinent to the issue we are chasing down. The following questions will help us:
	1. Is the traffic encrypted or plain text? Should it be?
	2. Can we see users attempting to access resources to which they should not have access?
	3. Are different hosts talking to each other that typically do not?
4. Detect and Alert 
	1. Are we seeing any errors? Is a device not responding that should be?
	2. Use our analysis to decide if what we see is benign or potentially malicious.
	3. Other tools like IDS/IPS can come in handy at this point.
5. Fix and Monitor - This should be included in any workflow we perform. If we make a change or fix an issue, we should continue to monitor the source for a time to determine if the issue has been resolved.
