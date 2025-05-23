Strange behavior like one host generating excessive traffic with HTTP or HTTPs might be seen in our web servers. Attackers like to abuse the transport layer many times, as the apps running on our servers might be vulnerable to different attacks. Usually, we can detect and identify fuzzing attempts through the following:
1. Excessive HTTP/HTTPs traffic from one host.
2. Referencing our web server's access logs for the same behavior.

Attackers will gather information before attempting to launch an attack. We might already have a Web Application Firewall to prevent this.

## Finding Directory Fuzzing

Directory fuzzing is used to find all possible web pages and locations in our web applications.
Using Wireshark, we can filter to 'http' or 'http.request'. It is quite simple to detect:
1. A host repeatedly attempts to access files on our web server which do not exist.
2. A host sends these in rapid succession.

For Apache, we can check our access.log.

## Finding Other Fuzzing Techniques

Other types of fuzzing such as fuzzing dynamic or static elements, like id fields, can also be detected.
However, sometimes, attackers will do the following to prevent detection:
- Stagger these responses across a longer period of time.
- Send these responses from multiple hosts or source addresses.

## Preventing Fuzzing Attempts

1. Maintain our virtualhost or web access configurations to return the proper response codes to throw off these scanners.
2. Establish rules to prohibit these IP addresses from accessing our server through our web application firewall.
