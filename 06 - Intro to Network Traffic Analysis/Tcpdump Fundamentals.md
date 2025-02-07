Tcpdump is a command-line packet sniffer that can directly capture and interpret data frames from a files or network interface.

## Traffic Captures

Here are some examples of basic Tcpdump switch usage:
##### Listing Available Interfaces
`sudo tcpdump -D`

##### Choosing an Interface to Capture From
`sudo tcpdump -i eth0`
If we use the switch `-nn`, it will refrain from resolving IP addresses and port numbers to their hostnames and common port names.

##### Display the Ethernet Header
`sudo tcpsump -i eth0 -e`

##### Include ASCII and Hex Output
`sudo tcpdump -i eth0 -X`
To get that view you get when using Wireshark when you analyze a packet.

##### Tcpdump Switch Combination
`sudo tcpdump -i wth0 -nnvXX`
No idea what this does.

##### Basic Capture Options Table
![[tcpdump_switch_table.png]]

## Tcpdump Output

Running through all those switches has shown us several different views.
Here's an example:
![[tcpdump_output_explained.png]]

Theoretically, we can use `tcpdump` to create an IDS/IPS system by having a Bash script analyze the intercepted packets according to a specific pattern.

## File Input/Output

##### Saving our PCAP Output to a File
`sudo tcpdump -i eth0 -w output.pcap`
The output will not scroll our terminal as usual. Instead, all output gets redirected to the specified file.

##### Reading Output From a File
`sudo tcpdump -r output.pcap`
And we go back to a basic view. We can still apply out switches.
