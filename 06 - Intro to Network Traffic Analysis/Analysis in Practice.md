This section breaks down a workflow for performing traffic analysis.

## Descriptive Analysis

This serves to describe a data set based on individual characteristics. It helps to detect possible error in data collection and/or outliers in the data set.

1. What is the issue?
	- Suspected breach? Networking issue?
2. Define our scope and the goal.
	- Target - Multiple hosts potentially downloading a malicious file from bad.example.com
	- When - Within the last 48 hours + 2 hours from now.
	- Supporting info - Filename/types 'superbad.exe' and 'new-crypto-miner.exe'.
3. Define our target(s).
	- Scope - 192.168.100.0/24 network, protocols used were HTTP and FTP.

## Diagnostic Analysis

This analysis clarifies the causes, effects and interactions of conditions. In doing do, it provides insights that are obtained through correlations and interpretation.

4. Capture network traffic
	- Plug into a link with access to the network to capture live traffic to try and grab one of the executables in transfer.
5. Identification of required network traffic components
	- Filter out any packets not needed for this investigation to include. For example, HTTP and FTP from the subnet, anything transferring or containing a GET request for the suspected executables.
6. An understanding of captured network traffic
	- Dig for our targets-filter on things like _ftp-data_ to find any files transferred and reconstruct them. For HTTP, we can filter on _http.request.method == "GET"_.

## Predictive Analysis

This creates a predictive model for future probabilities, by evaluating historical and current data. This method makes it possible to identify trends, detect deviations from expected values at an early stage and predict future occurrences as accurately as possible.

7. Note-taking and mins mapping of the found results
	- Annotating everything we do, see, or find throughout the investigation is crucial. Timeframes we captures, suspicious hosts within the network, conversations containing the files in question...
8. Summary of the analysis
	- Finally, summarize what we have found explaining the relevant details so that superiors can decide to quarantine the affected hosts or perform more significant incident response.
	- Our analysis will affect decisions made, so it is essential to be as clear and concise as possible.

## Prescriptive Analysis

What we just talked about. Let the 8 steps be a template for the future.
This process is usually cyclic and we will need to rerun steps based on our analysis of the original capture to build a bigger picture.

## Key Components of an Effective Analysis

##### 1. Know your environment
There are several key components to perform traffic analysis effectively. First, know the environment. If we are unsure if a host belongs in the network, how can we determine if it is rogue or not?
Keeping asset inventories and network maps is vital.

##### 2. Placement is Key
Closed to the source of the issue is the ideal placement of our capturing tool. If the traffic is coming form the internet, listening to the inbound links is a great way to see the complete picture.
If the problem seems to be isolated to one host on our internal network, try placing the capture tools in the same segment as the problem host and see what traffic is happening within the segment.

##### 3. Persistence
The issue will not always be easy to spot. It may not even be a frequent event on the network. For example, an attacker's C2 server reaching out to the victim may only happen once every several hours, or days.
Don't lose the drive to find the problem. It could mean the difference between stopping the attacker and a full-scale breach like a ransomware attack.

## Analysis Approach

Let's take a second to discuss easy wins when looking at traffic and finding problems.
Start with standard protocols and work our way into the austere and specific only to the organization. Most attacks will come from the internet, so it has to access the internal net somehow.
Then, check protocols that allow for communications between networks.
Look for patterns. Is a specific host or set of hosts checking something on the internet at the same time daily? This is a clear C2 profile setup.
Check anything host-to-host within our network. In a standard setup, hosts will rarely ever talk to each other, so be suspicious of any traffic that looks like this.
Look for unique events. The changing of patterns, like number of visits of a website by a host, or the User-Agent string not matching our applications, a random port only being bound once or twice.
In large environments, patterns are expected, so anything sticking our warrants a look.
**Ask for help.** Best advice I guess.
