Serves as both an IDS and IPS, but can also function as a packet logger or sniffer. It was created to operate efficiently on both general-purpose and custom hardware.

## Operation Modes

- **Inline IDS/IPS**
- **Passive IDS**
- **Network-based IDS**
- **Host-based IDS (not ideal)**
###### Passive mode
Gives Snort the ability to observe and detect traffic on a network interface, but it prevents outright blocking of traffic.
###### Inline mode
Gives the ability to block traffic if a particular packet warrants such an event.

## Architecture

To transit from a simple packet sniffer to a robust IDS, several key components were added:

- The **packet sniffer** (which includes the **Packet Decoder**) extracts network traffic, recognizing the structure of each packet. These are forwarded to the Preprocessors.

- **Preprocessors** within Snort identify the type or behavior of the forwarded packets. It contains an array of preprocessor plugins. The configuration of those can be found in _snort.lua_ Then, information is passed to the Detection Engine.

- The **Detection Engine** compares each packet with a predefined set of Snort rules. If there's a match, information is forwarded to the Logging and Alerting System.

- The **Logging and Alerting System** and **Output modules** are in charge of recording or triggering alerts as determined by each rule action. The Output modules are configured within Snort configuration file.

## Configuration & Validation

Snort offers a wide range of configuration options, and pre-configured files to facilitate a quick start. These default configuration files, _snort.lua_ and _snort_defaults.lua_, serve as the foundation for setting it up and getting it operational.

The _snort.lua_ file serves as the principal configuration file for Snort. It contains the sections: ****
- **Network variables**
- **Decoder configuration**
- **Base detection engine configuration**
- **Dynamic library configuration**
- **Preprocessor configuration**
- **Output plugin configuration**
- **Rule set customization**
- **Preprocessor and decoder rule set customization**
- **Shared object rule set customization**

To explore and get a brief description of all modules, run `snort --help-modules`.
To view default settings for a module, use `snort --help-config arp_spoof`.
To validate config files, run `snort -c <snort.lua> --daq-dir <daq>`. "daq" should be under _/usr/local/lib/daq_. It is not required for this command though.
DAQ (Data Acquisition Library) is an abstraction layer used by modules to communicate with both hardware and software network data sources.

## Inputs

The option -r reads a pcap file.
```shell-session
sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -r /home/htb-student/pcaps/icmp.pcap
```

The option -i listens on the specified network interfaces.
```shell-session
sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -i ens160
```

## Rules

Snort rules are composed of a header and an option. They resemble Suricata rules.
We can have a file with rules be imported to the configuration file.
```shell-session
---SNIP---
ips =
{
    -- use this to enable decoder and inspector alerts
    --enable_builtin_rules = true,

    -- use include for rules files; be sure to set your path
    -- note that rules files can include other rules files
    -- (see also related path vars at the top of snort_defaults.lua)

    { variables = default_variables, include = '/home/htb-student/local.rules' }
}
---SNIP---
```

We can also incorporate rules directly from the command line:
- For single rules file, use -R;
- To include an entire directory, use --rule-path;

## Outputs

We may encounter a significant amount of data. Let's see the key aspects:
- Basic Statistics - Upon shutdown, it generates various counts based on the config and processed traffic, like:
	- Packet Statistics - DAQ, decoders, number of received packets and UDP packets.
	- Module Statistics - Frequency of observed or performed actions.
	- File Statistics - File types, bytes and signatures.
	- Summary Statistics - Total runtime for packet processing, packets per second and profiling data.
- Alerts - (enabled using -A) shows details of detection events. Types of alert outputs:
	- -A cmg - (-A fast -d -e) displays alert information along with packet headers and payload.
	- -A u2 - logs events and triggering packers in a binary file (Unified2).
	- -A csv - CSV format.
- Performance Statistics - Additional data can be obtained, by configuring the _perf_monitor_ module. This data can be fed into an external program to monitor Snort's activity without interrupting its operation. The _profiler_ module allows tracking of time and space usage by modules and rules. It appears under Summary Statistics.

## Key Features

- Deep packet inspection, packet capture, and logging
- Intrusion detection
- Network Security Monitoring
- Anomaly detection
- Support for multiple tenants
- Both IPv6 and IPv4 are supported
