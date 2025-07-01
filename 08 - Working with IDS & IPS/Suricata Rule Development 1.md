A rule serves as a directive, instructing the engine to actively watch for certain markers in the network traffic. When this happens, we get notified.
These rules aren't exclusively focused on the detection of nefarious activities.
It's worth noting that each rule we deploy consumes a portion of the host's CPU and memory resources.

## Rule Anatomy

Sample rule: `action protocol from_ip port -> to_ip port (msg:"Known malicious behavior, possible X malware infection"; content:"some thing"; content:"some other thing"; sid:10000001; rev:1;)`.

- **Header** - encapsulates the intended action of the rule, along with the protocol where the rule is expected to be applied, the IP addresses, port information an an arrow indicating traffic directionality.
	- _action_ - instructs it on what steps to take if the contents match. Generating an alert, logging the traffic, ignoring the packet, dropping the packet, or sending RST packets.
	- _protocol_ - tcp, udp, icmp, ip, http, tls, smb or dns.
	- _direction_ - uses rules host variables and rule direction. Examples:
		- Outbound - $HOME_NET any -> $EXTERNAL_NET 9090
		- Inbound - $EXTERNAL_NET any -> $HOME_NET any
		- Bidirectional - $EXTERNAL_NET any <> $HOME_NET any
	- _rule ports_ - define the ports at which the traffic will be evaluated. You can also define a variable with ports in suricata.yaml.

- **Rule message & content** - contains the message we wish to be displayed to the analysts when an activity is detected.
	- _rule message (msg)_ - arbitrary text displayed when the rule is triggered. They should contain details about malware architecture, family and action.
		- flow - identifies the originator and responder.
		- dsize - matches the payload size of the packet (only the TCP segment length).
	- _rule content_ - comprises unique values that help identify specific network traffic or activities.
		- rule buffers - so that we don't have to search the entire packet for every content match.
		- rule options - additional modifiers to aid detection.
			- nocase - ensures rules are not bypassed through case changes.
			- offset - informs about the start position inside the packet for matching.
			- distance - tells it to look for the specified content n bytes relative to previous match.
- **Rule metadata** - metadata.
	- _reference_ - gives as a lead/trail that takes us back to the original source of information that inspired the creation of the rule.
	- _sid_ - unique numeric identifier so that the rule writer can manage and distinguish between rules.
	- _revision_ - insights into the rule's version. Highlights modifications and enhancements made.

PCRE - Pearl Compatible Regular Expression - helps crafting rules.

Example: `alert http any any -> $HOME_NET any (msg: "ATTACK [PTsecurity] Apache Continuum <= v1.4.2 CMD Injection"; content: "POST"; http_method; content: "/continuum/saveInstallation.action"; offset: 0; depth: 34; http_uri; content: "installation.varValue="; nocase; http_client_body; pcre: !"/^\$?[\sa-z\\_0-9.-]*(\&|$)/iRP"; flow: to_server, established;sid: 10000048; rev: 1;)`.

- Firstly, the rule triggers on HTTP traffic (`alert http`) from any source and destination to any port on the home network (`any any -> $HOME_NET any`).
- The msg field gives a human-readable description of what the alert is for, namely `ATTACK [PTsecurity] Apache Continuum <= v1.4.2 CMD Injection`.
- Next, the rule checks for the `POST` string in the HTTP method using the `content` and `http_method` keywords. The rule will match if the HTTP method used is a POST request.
- The `content` keyword is then used with `http_uri` to match the URI `/continuum/saveInstallation.action`, starting at `offset 0` and going to a `depth` of `34` bytes. This specifies the targeted endpoint, which in this case is the `saveInstallation` action of the Apache Continuum application.
- Following this, another content keyword searches for `installation.varValue=` in the HTTP client body, case insensitively (`nocase`). This string may be part of the command injection payload that the attacker is trying to deliver.
- Next, we see a `pcre` keyword, which is used to implement Perl Compatible Regular Expressions.
    - `^` marks the start of the line.
    - `\$?` checks for an optional dollar sign at the start.
    - `[\sa-z\\_0-9.-]*` matches zero or more (`*`) of the characters in the set. The set includes:
        - `\s` a space
        - `a-z` any lowercase letter
        - `\\` a backslash
        - `_` an underscore
        - `0-9` any digit
        - `.` a period
        - `-` a hyphen
	- `(\&|$)` checks for either an ampersand or the end of the line.
	- `/iRP` at the end indicates this is an inverted match (meaning the rule triggers when the match does not occur), case insensitive (`i`), and relative to the buffer position (`RP`).
- Finally, the `flow` keyword is used to specify that the rule triggers on `established`, inbound traffic towards the server.

## IDS/IPS Rule Development Approaches

Basically, we can detect anomalies based on behavior or signatures.

## Example 1 - PowerShell Empire

```shell-session
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"ET MALWARE Possible PowerShell Empire Activity Outbound"; flow:established,to_server; content:"GET"; http_method; content:"/"; http_uri; depth:1; pcre:"/^(?:login\/process|admin\/get|news)\.php$/RU"; content:"session="; http_cookie; pcre:"/^(?:[A-Z0-9+/]{4})*(?:[A-Z0-9+/]{2}==|[A-Z0-9+/]{3}=|[A-Z0-9+/]{4})$/CRi"; content:"Mozilla|2f|5.0|20 28|Windows|20|NT|20|6.1"; http_user_agent; http_start; content:".php|20|HTTP|2f|1.1|0d 0a|Cookie|3a 20|session="; fast_pattern; http_header_names; content:!"Referer"; content:!"Cache"; content:!"Accept"; sid:2027512; rev:1;)
```

The Suricata rule above is designed to detect possible outbound activity from [PowerShell Empire](https://github.com/EmpireProject/Empire), a common post-exploitation framework used by attackers. Let's break down the important parts of this rule to understand its workings.

- `msg:"ET MALWARE Possible PowerShell Empire Activity Outbound"`: This is the message that will be included in the alert to describe what the rule is looking for.
- `flow:established,to_server`: This specifies the direction of the traffic. The rule is looking for established connections where data is flowing to the server.
- `content:"GET"; http_method;`: This matches the HTTP GET method in the HTTP request.
- `content:"/"; http_uri; depth:1;`: This matches the root directory ("/") in the URI.
- `pcre:"/^(?:login\/process|admin\/get|news)\.php$/RU";`: This PCRE is looking for URIs that end with login/process.php, admin/get.php, or news.php.
    - `PowerShell Empire` is an open-source Command and Control (C2) framework. Its agent can be explored via the following repository.[https://github.com/EmpireProject/Empire/blob/master/data/agent/agent.ps1#L78](https://github.com/EmpireProject/Empire/blob/master/data/agent/agent.ps1#L78)
- `content:"session="; http_cookie;`: This is looking for the string "session=" in the HTTP cookie.
- `pcre:"/^(?:[A-Z0-9+/]{4})*(?:[A-Z0-9+/]{2}==|[A-Z0-9+/]{3}=|[A-Z0-9+/]{4})$/CRi";`: This PCRE is checking for base64-encoded data in the Cookie.
    - A plethora of articles examining `PowerShell Empire` exist, here is one noting that the cookies utilized by PowerShell Empire adhere to the Base64 encoding standard. [https://www.keysight.com/blogs/tech/nwvs/2021/06/16/empire-c2-networking-into-the-dark-side](https://www.keysight.com/blogs/tech/nwvs/2021/06/16/empire-c2-networking-into-the-dark-side)
- `content:"Mozilla|2f|5.0|20 28|Windows|20|NT|20|6.1"; http_user_agent; http_start;`: This matches a specific User-Agent string that includes "Mozilla/5.0 (Windows NT 6.1".
    - [https://github.com/EmpireProject/Empire/blob/master/data/agent/agent.ps1#L78](https://github.com/EmpireProject/Empire/blob/master/data/agent/agent.ps1#L78)
- `content:".php|20|HTTP|2f|1.1|0d 0a|Cookie|3a 20|session="; fast_pattern; http_header_names;`: This matches a pattern in the HTTP headers that starts with ".php HTTP/1.1\r\nCookie: session=".
- `content:!"Referer"; content:!"Cache"; content:!"Accept";`: These are negative content matches. The rule will only trigger if the HTTP headers do not contain "Referer", "Cache", and "Accept".

This Suricata rule triggers an alert when it detects an established HTTP GET request from our network to an external network, with a specific pattern in the URI, cookie, and user-agent fields, and excluding certain headers.

## Example 2 - Covenant

```shell-session
alert tcp any any -> $HOME_NET any (msg:"detected by body"; content:"<title>Hello World!</title>"; detection_filter: track by_src, count 4 , seconds 10; priority:1; sid:3000011;)
```

**Rule source**: [Signature-based IDS for Encrypted C2 Traffic Detection - Eduardo Macedo](https://repositorio-aberto.up.pt/bitstream/10216/142718/2/572020.pdf)

The (inefficient) Suricata rule above is designed to detect certain variations of [Covenant](https://github.com/cobbr/Covenant), another common post-exploitation framework used by attackers. Let's break down the important parts of this rule to understand its workings.

- `content:"<title>Hello World!</title>";`: This instructs Suricata to look for the string `<title>Hello World!</title>` in the TCP payload.
    - `Covenant` is an open-source Command and Control (C2) framework. Its underpinnings can be explored via the following repository.[https://github.com/cobbr/Covenant/blob/master/Covenant/Data/Profiles/DefaultHttpProfile.yaml#L35](https://github.com/cobbr/Covenant/blob/master/Covenant/Data/Profiles/DefaultHttpProfile.yaml#L35)
- `detection_filter: track by_src, count 4, seconds 10;`: This is a post-detection filter. It specifies that the rule should track the source IP address (`by_src`) and will only trigger an alert if this same detection happens at least 4 times (`count 4`) within a 10-second window (`seconds 10`).

This Suricata rule is designed to generate a high-priority alert if it detects at least four instances of TCP traffic within ten seconds that contain the string `<title>Hello World!</title>` in the payload, originating from the same source IP and headed towards any host on our internal network.

## Example 3 - Covenant using Analytics

```shell-session
alert tcp $HOME_NET any -> any any (msg:"detected by size and counter"; dsize:312; detection_filter: track by_src, count 3 , seconds 10; priority:1; sid:3000001;)
```

- `dsize:312;`: This instructs Suricata to look for TCP traffic with a data payload size of exactly 312 bytes.

This Suricata rule is designed to generate a high-priority alert if it detects at least three instances of TCP traffic within ten seconds that each contain a data payload of exactly 312 bytes, all originating from the same source IP within our network and headed anywhere.

## Example 4 - Sliver

```shell-session
alert tcp any any -> any any (msg:"Sliver C2 Implant Detected"; content:"POST"; pcre:"/\/(php|api|upload|actions|rest|v1|oauth2callback|authenticate|oauth2|oauth|auth|database|db|namespaces)(.*?)((login|signin|api|samples|rpc|index|admin|register|sign-up)\.php)\?[a-z_]{1,2}=[a-z0-9]{1,10}/i"; sid:1000007; rev:1;)
```

**Rule source**: [https://www.bilibili.com/read/cv19510951/](https://www.bilibili.com/read/cv19510951/)

The Suricata rule above is designed to detect certain variations of [Sliver](https://github.com/BishopFox/sliver), yet another common post-exploitation framework used by attackers. Let's break down the important parts of this rule to understand its workings.

- `content:"POST";`: This option instructs Suricata to look for TCP traffic containing the string "POST".
- `pcre:"/\/(php|api|upload|actions|rest|v1|oauth2callback|authenticate|oauth2|oauth|auth|database|db|namespaces)(.*?)((login|signin|api|samples|rpc|index|admin|register|sign-up)\.php)\?[a-z_]{1,2}=[a-z0-9]{1,10}/i";`: This regular expression is utilized to identify specific URI patterns in the traffic. It will match URIs that contain particular directory names followed by file names ending with a PHP extension.
    - `Sliver` is an open-source Command and Control (C2) framework. Its underpinnings can be explored via the following repository.[https://github.com/BishopFox/sliver/blob/master/server/configs/http-c2.go#L294](https://github.com/BishopFox/sliver/blob/master/server/configs/http-c2.go#L294)

And another rule for this:
```shell-session
alert tcp any any -> any any (msg:"Sliver C2 Implant Detected - Cookie"; content:"Set-Cookie"; pcre:"/(PHPSESSID|SID|SSID|APISID|csrf-state|AWSALBCORS)\=[a-z0-9]{32}\;/"; sid:1000003; rev:1;)
```

Let's break down the important parts of this rule to understand its workings.

- `content:"Set-Cookie";`: This option instructs Suricata to look for TCP traffic containing the string `Set-Cookie`.
- `pcre:"/(PHPSESSID|SID|SSID|APISID|csrf-state|AWSALBCORS)\=[a-z0-9]{32}\;/";`: This is a regular expression used to identify specific cookie-setting patterns in the traffic. It matches the `Set-Cookie` header when it's setting specific cookie names (PHPSESSID, SID, SSID, APISID, csrf-state, AWSALBCORS) with a value that's a 32-character alphanumeric string.
