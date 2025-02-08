It's Tcpdump with a GUI. It captures live data from WiFi, USB and Bluetooth (and others) and saves the traffic to several formats. It also possesses decryption capabilities.

## Wireshark GUI Walkthrough

![[wireshark-interface.webp]]

1. Packet List: `Orange`
    - In this window, we see a summary line of each packet that includes the fields listed below by default. We can add or remove columns to change what information is presented.
        - Number- Order the packet arrived in Wireshark
        - Time- Unix time format
        - Source- Source IP
        - Destination- Destination IP
        - Protocol- The protocol used (TCP, UDP, DNS, ETC.)
        - Information- Information about the packet. This field can vary based on the type of protocol used within. It will show, for example, what type of query It is for a DNS packet.

2. Packet Details: `Blue`
    - The Packet Details window allows us to drill down into the packet to inspect the protocols with greater detail. It will break it down into chunks that we would expect following the typical OSI Model reference. `The packet is dissected into different encapsulation layers for inspection.`
    - Keep in mind, Wireshark will show this encapsulation in reverse order with lower layer encapsulation at the top of the window and higher levels at the bottom.

3. Packet Bytes: `Green`
    - The Packet Bytes window allows us to look at the packet contents in ASCII or hex output. As we select a field from the windows above, it will be highlighted in the Packet Bytes window and show us where that bit or byte falls within the overall packet.
    - This is a great way to validate that what we see in the Details pane is accurate and the interpretation Wireshark made matches the packet output.
    - Each line in the output contains the data offset, sixteen hexadecimal bytes, and sixteen ASCII bytes. Non-printable bytes are replaced with a period in the ASCII format.

## Pre-capture and Post-capture Processing and Filtering

##### Capture Filters
![[wireshark_capture_filters.png]]

A lot of text would go here:
