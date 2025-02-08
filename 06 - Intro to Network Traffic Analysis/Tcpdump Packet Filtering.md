## Filtering and Advanced Syntax Options

Utilizing more advanced filtering options will enable us to trim down what traffic is printed to output or sent to file. We can capture as widely as we wish, or be super specific only to capture packets from a particular host, or even with a particular bit in the TCP header set to on.

##### Helpful TCPDump Filters
![[tcpdump_advanced_filters_table.png]]

##### Host Filter - _host \[IP\]_
`sudo tcpdump -i eth0 host 172.16.146.2`
This is often used when we want to examine only a specific host or server.

##### Source/Destination Filter - _src/dest \[host|net|port\] \[IP|Network Range|Port\]_
`sudo tcpdump -i eth0 src host 172.16.146.2`
`sudo tcpsump -i eth0 dest net 172.16.146.0/24`
`sudo tcpdump -i eth0 tcp src port 80`

##### Protocol Filter - _\[proto]\[tcp/udp/icmp|protocol number\]_
`sudo tcpdump -i eth0 udp`
`sudo tcpdump -i eth0 proto 17`

##### Port Filter - _port \[port number]_
`sudo tcpdump -i eth0 tcp port 443`

##### Port Range Filter - _portrange \[portrange 0-65535]_
`sudo tcpdump -i eth0 portrange 0-1024`

##### Less/Greater Filter - _less/greater \[size in bytes]_
`sudo tcpdump -i eth0 less 64`
`sudo tcpdump -i eth0 greater 500`

##### AND, OR and NOT Filter - _and|or|not \[requirement]_
`sudo tcpdump -i eth0 host 192.168.0.1 and port 23`
`sudo tcpdump -r sus.pcap icmp or host 172.16.146.1`
`sudo tcpdump -r sus.pcap not icmp`

## Pre-Capture Filters VS Post-Capture Processing

By applying filters to the capture, it will drop any traffic not matching the filer. This reduces the amount of data in the captures and potentially clear out traffic we may need later.
When applying the filter to a file, it will save potential valuable data in the captures and won't change the capture file.
