Formely known as Bro, btw. Read [[Zeek Fundamentals]] to learn more.

## Example 1 - Beaconing Malware

Beaconing is a process by which malware communicates with its C2 server to receive instrauctions or exfiltrate data. By analyzing connection logs, we can look for patterns in outbound traffic.
Anomalies can be further explored using Zeek scripts specifically designed to spot beaconing patterns.

```shell-session
/usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/psempire.pcap
cat conn.log
```

If we look carefully we will notice connections being made to 51.15.197.127:80, approximately every 5 seconds, indicative of malware beaconing. This is common behavior of PowerShell Empire.

## Example 2 - DNS Exfiltration

Since data exfiltration often mimics normal network traffic, it can be difficult to be detected. Zeek allows us to analyze our network traffic at a deeper level.

files.log can be used to identify large amounts of data being sent to an unusual external destination; http.log and dns.log can also be utilized to identify potential covert exfiltration channels, such as DNS tunneling or HTTP POST requests to a suspicious domain.

```shell-session
/usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/dnsexfil.pcapng
cat dns.log
```

Let's focus on the requested (sub)domains by using zeek-cut.
```shell-session
cat dns.log | /usr/local/zeek/bin/zeek-cut query | cut -d . -f1-7
```

It becomes clear that "letsgohunt.online" possesses a significant number of subdomains, similar to cloud providers.

## Example 3 - TLS Exfiltration

Now, detecting data exfiltration over TLS.

```shell-session
/usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/tlsexfil.pcap
cat conn.log
```

Once again, let's use zeek-cut.
```shell-session
cat conn.log | /usr/local/zeek/bin/zeek-cut id.orig_h id.resp_h orig_bytes | sort | grep -v -e '^$' | grep -v '-' | datamash -g 1,2 sum 3 | sort -k 3 -rn | head -10
```

Let's analyze the command above.
- `/usr/local/zeek/bin/zeek-cut id.orig_h id.resp_h orig_bytes`: In this case, we're extracting the `id.orig_h`, `id.resp_h`, and `orig_bytes` (number of bytes sent by the originating host) fields.
- `grep -v -e '^$'`: This command filters out any empty lines.
- `datamash -g 1,2 sum 3`: `datamash` is a command-line tool that performs basic numeric, textual, and statistical operations. The `-g 1,2` option groups the output by the first two fields (the IP addresses of the originating and responding hosts), and `sum 3` computes the sum of the third field (the number of bytes sent) for each group.
- `sort -k 3 -rn`: This command sorts the output of the previous command in descending order (`-r`) based on the numerical value (`-n`) of the third field (`-k 3`), which is the sum of `orig_bytes` for each pair of IP addresses.

We notice a lot of data being sent to 192.168.151.181.

## Example 4 - PsExec

PsExec is frequently used for remote administration within Active Directory environments. With no surprise, it is a cool tool for adversaries to carry out their remote code execution attacks.

In the following, it is illustrated a typical attack sequence where an attacker transfers a binary file to a target machine using the ADMIN$ share via SMB. Following this, the attacker remotely launches this file as a temporary service by utilizing the IPC$ share.

```shell-session
/usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/psexec_add_user.pcap
cat smb_files.log
cat dce_rpc.log
cat smb_mapping.log
```

The last 2 logs show the temporary service creation.
