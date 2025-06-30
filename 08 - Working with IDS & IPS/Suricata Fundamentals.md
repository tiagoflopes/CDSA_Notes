The objective of Suricata is to dissect every iota of network traffic, seeking potential signs of malicious activities. Suricata's efficiency is second to none.

## Operation Modes

It operates in 4 distinct modes:
1. **Intrusion Detection System (IDS) mode** - acts as a silent observer. It meticulously examines traffic, flagging potential attacks but refraining from any form of intervention.
2. **Intrusion Prevention System (IPS) mode** - adopts a proactive stance. All network traffic passes through stringent checks and is only granted access to the internal network upon its approval. The inspection process may introduce latency.
3. **Intrusion Detection Prevention System (IDPS) mode** - brings the best of both. It passively monitors traffic, while possessing the ability to actively transmit RST packets in response to abnormal activity. Low latency, crucial for seamless network operations.
4. **Network Security Monitoring (NSM) mode** - transitions into a dedicated logging mechanism.

## Inputs

There are 2 main categories of inputs:
1. **Offline Input** - This involves reading PCAP files for precessing priviously captured packets. Useful to experiment with various rule sets and configurations.
2. **Live Input** - Packets are read directly from network interfaces.

There are other less common or more advanced inputs available.

## Outputs

It creates logs, alerts and additional network-related data, such as DNS requests and network flows. *EVE*, a JSON formatted log that records a wide range of event types including alerts, HTTP, DNS, TLS metadata, drop, SMTP metadata, flow, netflow and more. Tools such as Logstach can easily consume this output. *Unified2*, a Snort binary alert format.

## Configuring Suricata & Custom Rules

Get all the rules files: `ls -lah /etc/suricata/rules`.
They can be inspected to understand their functionality.
Rules can be commented out, meaning they aren't loaded.
Variables such as HOME_NET and EXTERNAL_NET represent from and to IP addresses, which can be defined in the suricata.yaml configuration file.
In that same file we can add a path to a new rule in "rule-files".

## Hands-on With Inputs

Let's experiment with both offline and live input.

##### Offline Input
`suricata -r suspicious.pcap`
This created carious logs (eve.json, fast.log and stats.log). This command can be ran with -k flag to bypass checksums and -l to log in a different directory.

##### Live Input
With LibPCAP mode: `sudo suricata --pcap=<interface> -vv`.
With Inline (NFQ) mode, execute `sudo iptables -I FORWARD -j NFQUEUE` and then `sudo suricata -q 0`.
To see it dealing with live traffic, use tcpreplay to replay a pcap file: `sudo tcpreplay -i <interface> <pcap>`. These logs are available in `/var/log/suricata`.

## Hands-on With Outputs

Let's delve into each log file provided.

##### eve.json
To filter out only alert events, for example, use jq as follows.
```sh
cat /var/log/suricata/eve.json | jq -c 'select(.event_type == "alert")'
```

Or to observe the earliest DNS event.
```sh
cat /var/log/suricata/eve.json | jq -c 'select(.event_type == "dns")' | head -1 | jq .
```

*flow_id* is a unique identifier assigned by Suricata to each network flow. A flow is a set of IP packets passing through a network interface in a specific direction and between a given pair of source and destination endpoints.

*pcap_cnt* is a counter that Suricata increments for each packet it processes form the network traffic or from a PCAP file.

##### fast.log
This is a text-based log that records alerts only.

##### stats.log
Human-readable statistics log, particularly useful while debugging Suricata deployments.

There are options to create a more focused output strategy.

## Hands-on with File Extraction

This feature allows us to capture and store files transferred over a number of different protocols, providing invaluable data fro threat hunting, forensics, or simply data analysis.
This can be enabled in the configuration file, in the section named "file-store".
```yaml
file-store:
  version: 2
  enabled: yes
  force-filestore:yes
```

File extraction isn't an automatic process. It needs explicit instructions. The simplest rules we can add to our local rule to experiment with is the following.
`alert http any any -> any any (msg:"FILE store all"; filestore; sid:2; rev:1;)`

This will then create a directory in the specified directory with files, sorted by named. To inspect them, we can use the xxd tool.
`xxd ./21/21742fc621f83041db2e47b0899f5aea6caa00a4b67dbff0aae823e6817c5433 | head`

## Live Rule Reloading Feature & Updating Rulesets

This allows us to update our ruleset without interrupting ongoing traffic inspection. Again, this can be configured in the config file.
```yaml
detect-engine:
  - reload: true
```

Then execute kill command, which will signal the Suricata process to refresh its rule set without necessitating a complete restart.
`sudo kill -usr2 $(pidof suricata)`

To actually update Suricata's ruleset, use `suricata-update` tool.

## Validating Configuration

To validate the configuration, we can use the -T option, which runs a test to check if the config file is valid and all files referenced there are accessible.

## Documentation

Documentation: [Suricata's documentation](https://docs.suricata.io/).

## Key Features

Key features that bolster its effectiveness include:
- Deep packet inspection and packet capture logging
- Anomaly detection and NSM
- IDS, IPS and hybrid IDPS
- Lua scripting
- Geographic IP identification (GeoIP)
- Full IPv4 and IPv6 support
- IP reputation
- File extration
- Advanced protocol inspection
- Multitenancy
