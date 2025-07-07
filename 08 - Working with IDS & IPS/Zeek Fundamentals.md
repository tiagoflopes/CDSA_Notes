It is a network traffic analyzer. It is also helpful for troubleshooting network issues and conducting a variety of measurements within a network.
What sets it apart is its highly capable scripting language, which allows users to customize and extend its functionalities, letting blue team members develop their own logic and strategies for network analysis and intrusion detection.
This is not another signature-based IDS. It's a platform that can facilitate semantic misuse detection, anomaly detection and behavioral analysis.

## Operation Modes

It operates in the following modes:
- Fully passive traffic analysis
- libpcap interface for packet capture
- Real-time and offline analysis
- Cluster support for large-scale deployments

## Architecture

It contains 2 primary components.

##### Event Engine
It takes an incoming packet stream and transforms it into a series of high-level events. These _events_ describe network activity in policy-neutral terms (they don't offer an interpretation or evaluation of it).

##### Script Interpreter
Executes a set of Zeek scripts that define actions to be taken upon the detection of certain events. This provides interpretation or analysis of those events gathered previously.

## Logs

Among the diverse array of logs Zeek produces, some familiar ones include:
- conn.log - provides details about IP, TCP, UDP and ICMP connections;
- dns.log
- http.log
- ftp.log
- smtp.log

Let's consider the http.log, which contains:
- host;
- uri;
- referrer;
- user_agent;
- status_code;

Zeek compresses log files every hour, which are then stored in a directory named in the YYYY-MM-DD format.

Zeek logs can be analyzed with Unix commands, but it provides the specialized tool ___zeek-cut___ for handling log files.

## Key Features

Key features that bolster Zeek's effectiveness include:
- Comprehensive logging of network activities
- Analysis of application-layer protocols (irrespective of the port, covering protocols like HTTP, DNS, FTP, SMTP, SSH, SSL, etc.)
- Ability to inspect file content exchanged over application-layer protocols
- IPv6 support
- Tunnel detection and analysis
- Capability to conduct sanity checks during protocol analysis
- IDS-like pattern matching
- Powerful, domain-aware scripting language that allows for expressing arbitrary analysis tasks and managing network state over time
- Interfacing that outputs to well-structured ASCII logs by default and offers alternative backends for ElasticSearch and DataSeries
- Real-time integration of external input into analyses
- External C library for sharing Zeek events with external programs
- Capability to trigger arbitrary external processes from within the scripting language
