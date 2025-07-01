Same, but for encrypted traffic. We can still look at other stuff.
SSL/TLS certificated contain a plethora of details that remain unencrypted.
We can also utilize the [JA3](https://github.com/salesforce/ja3) hash, a fingerprinting method that provides a unique representation for each SSL/TLS client. These hashes are a powerful tool in formulating detection rules for encrypted traffic.

## Example 5 - Dridex

```shell-session
alert tls $EXTERNAL_NET any -> $HOME_NET any (msg:"ET MALWARE ABUSE.CH SSL Blacklist Malicious SSL certificate detected (Dridex)"; flow:established,from_server; content:"|16|"; content:"|0b|"; within:8; byte_test:3,<,1200,0,relative; content:"|03 02 01 02 02 09 00|"; fast_pattern; content:"|30 09 06 03 55 04 06 13 02|"; distance:0; pcre:"/^[A-Z]{2}/R"; content:"|55 04 07|"; distance:0; content:"|55 04 0a|"; distance:0; pcre:"/^.{2}[A-Z][a-z]{3,}\s(?:[A-Z][a-z]{3,}\s)?(?:[A-Z](?:[A-Za-z]{0,4}?[A-Z]|(?:\.[A-Za-z]){1,3})|[A-Z]?[a-z]+|[a-z](?:\.[A-Za-z]){1,3})\.?[01]/Rs"; content:"|55 04 03|"; distance:0; byte_test:1,>,13,1,relative; content:!"www."; distance:2; within:4; pcre:"/^.{2}(?P<CN>(?:(?:\d?[A-Z]?|[A-Z]?\d?)(?:[a-z]{3,20}|[a-z]{3,6}[0-9_][a-z]{3,6})\.){0,2}?(?:\d?[A-Z]?|[A-Z]?\d?)[a-z]{3,}(?:[0-9_-][a-z]{3,})?\.(?!com|org|net|tv)[a-z]{2,9})[01].*?(?P=CN)[01]/Rs"; content:!"|2a 86 48 86 f7 0d 01 09 01|"; content:!"GoDaddy"; sid:2023476; rev:5;)
```

The rule above triggers an alert upon detecting a TLS session from the external network to the home network, where the payload of the session contains specific byte patterns and meets several conditions. These patterns and conditions correspond to SSL certificates that have been linked to certain variations of the Dridex trojan, as referenced by the SSL blacklist on `abuse.ch`.

No need to understand the rule in its entirety, but let's break down the important parts of it.

- `content:"|16|"; content:"|0b|"; within:8;`: The rule looks for the hex values 16 and 0b within the first 8 bytes of the payload. These represent the handshake message (0x16) and the certificate type (0x0b) in the TLS record.
- `content:"|03 02 01 02 02 09 00|"; fast_pattern;`: The rule looks for this specific pattern of bytes in the packet, which may be characteristic of the certificates used by Dridex.
- `content:"|30 09 06 03 55 04 06 13 02|"; distance:0; pcre:"/^[A-Z]{2}/R"`;: This checks for the 'countryName' field in the certificate's subject. The content match here corresponds to an ASN.1 sequence specifying an attribute type and value for 'countryName' (OID 2.5.4.6). The following PCRE checks that the value for 'countryName' begins with two uppercase letters, which is a standard format for country codes.
- `content:"|55 04 07|"; distance:0;`: This checks for the 'localityName' field in the certificate's subject (OID 2.5.4.7).
- `content:"|55 04 0a|"; distance:0;`: This checks for the `organizationName` field in the certificate's subject (OID 2.5.4.10).
- `content:"|55 04 03|"; distance:0; byte_test:1,>,13,1,relative;`: This checks for the `commonName` field in the certificate's subject (OID 2.5.4.3). The following byte_test checks that the length of the `commonName` field is more than 13.

The mentioned OIDs (Object Identifiers) are part of the X.509 standard for PKI and are used to uniquely identify the types of fields contained within certificates.

## Example 6 - Sliver

```shell-session
alert tls any any -> any any (msg:"Sliver C2 SSL"; ja3.hash; content:"473cd7cb9faa642487833865d516e578"; sid:1002; rev:1;)
```

The Suricata rule above is designed to detect certain variations of [Sliver](https://github.com/BishopFox/sliver) whenever it identifies a TLS connection with a specific JA3 hash.

The JA3 hash can be calculated as follows: `ja3 -a --json /home/htb-student/pcaps/sliverenc.pcap`.
