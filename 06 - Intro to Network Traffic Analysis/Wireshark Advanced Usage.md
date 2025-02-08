Advanced capabilities ranging from tracking TCP conversations to cracking wireless credentials.

## Plugins

##### Statistics Tab
![[wireshark-statistics.webp]]

##### Analyze Tab
From this tab, we can utilize plugins that allow us to do things such as following TCP streams, filter on conversation types, prepare new packet filters and examine the expert info Wireshark generates about the traffic.

**Following TCP Streams**
Wireshark can stitch packets back together to recreate the entire stream in a readable format.

##### Filter for a Specific TCP Stream
`tcp-stream eq #`

##### FTP-Request-Command Filter
Shows any commands sent across the ftp-control channel (port 21).

##### FTP-Data Filter
Shows any data transferred over the data channel (port 20).
