# Network Discovery Detection & Scan Analysis

Training Environment: TryHackMe - Network Discovery Detection
Analysis Environment: Ubuntu VM, Elastic/Kibana, and exported Zeek/SIEM connection logs

## Investigation Overview

This investigation focused on identifying and classifying network discovery activity within firewall and Zeek connection logs.

The goal was to move from raw network telemetry to an analyst-level understanding of:

* Internal vs. external scanning
* Horizontal and vertical port scanning
* ICMP host discovery
* TCP SYN scanning
* Targeted service discovery
* Whether the available telemetry showed evidence of UDP scanning

The investigation was performed using command-line analysis with `awk`, `grep`, `cut`, `sort`, and `uniq`, with additional validation in Elastic/Kibana.

---

## Data Source

**Log source:** CSV connection logs exported from a SIEM environment

**Primary file analyzed:**

```text
log-session-2.csv
```

The logs contained fields including:

```text
@timestamp
source.ip
source.port
destination.ip
destination.port
network.protocol
message
event.dataset
```

The connection data also contained Zeek connection state information, which was useful for identifying scanning behavior.

---

# 1. Internal vs. External Scanning

I first compared the source and destination IP addresses in the available log files to determine whether scanning activity originated internally or externally.

### Internal scanning

`log-session-2.csv` contained internal-to-internal activity.

The primary scanning source was:

```text
192.168.230.127
```

The source and destination addresses were private/internal IP addresses, indicating that the activity originated from within the network.

I used the following command to examine the repeated source value:

```bash
cat log-session-2.csv | cut -d "," -f3 | uniq -c
```

The output showed:

```text
1 "source.port"
2276 "192.168.230.127"
```

This indicated **2,276 logged entries associated with 192.168.230.127** in the dataset.

Because this was internal-to-internal scanning, it warranted more attention than an external reconnaissance attempt. If the source were an unauthorized system, the activity could indicate that an attacker had already obtained a foothold and was performing internal discovery.

### External scanning

The external scanning activity originated from:

```text
203.0.113.25
```

The traffic showed an external source communicating with an internal destination, including:

```text
203.0.113.25 → 192.168.230.145
```

This is consistent with reconnaissance against an exposed internal asset rather than evidence of an already-established internal foothold.

---

# 2. Horizontal Scan Detection

### Finding

**Scanned network:**

```text
203.0.113.0/24
```

I extracted the unique destination IP addresses from `log-session-2.csv` using a CSV-aware `awk` expression:

```bash
awk -v FPAT='([^,]+)|("[^"]+")' 'NR>1 { gsub(/"/,"",$4); print $4 }' log-session-2.csv | sort | uniq
```

The output contained a large number of addresses sharing the same first three octets, including addresses such as:

```text
203.0.113.10
203.0.113.100
203.0.113.101
203.0.113.102
203.0.113.103
...
203.0.113.254
```

The important observation was not an individual IP address, but the **pattern**:

```text
203.0.113.X
```

The first three octets remained consistent while the host portion changed across many destinations.

A `/24` network contains 24 network bits and 8 host bits:

```text
203 . 0 . 113 . X
 8     8     8   8
 └────── 24 ──────┘
```

Therefore, the group of addresses belongs to:

```text
203.0.113.0/24
```

### Analyst interpretation

A horizontal scan occurs when one source probes the same or similar service across many destination hosts.

The large number of `203.0.113.X` destinations indicated that the activity was distributed across a network range rather than focused on a single host.

**Finding:**

> Network discovery activity was observed across the `203.0.113.0/24` range, consistent with horizontal scanning.

---

# 3. Vertical Scan Detection

I then looked for the opposite pattern: one source communicating with a single destination across many different ports.

I grouped the logs by source IP and destination IP and counted unique destination ports:

```bash
awk -v FPAT='([^,]+)|("[^"]+")' 'NR>1{gsub(/"/,"",$2);gsub(/"/,"",$4);gsub(/"/,"",$5);key=$2","$4;ports[key][$5]=1}END{for(k in ports){n=0;for(p in ports[k])n++;if(n>1){split(k,a,",");print "Vertical scan: source="a[1]" target="a[2]" ports="n}}}' log-session-2.csv
```

The output identified:

```text
Vertical scan: source=192.168.230.127 target=192.168.230.145 ports=1001
```

This was the strongest vertical scanning finding in the dataset.

### Analyst interpretation

The same source:

```text
192.168.230.127
```

was communicating with the same destination:

```text
192.168.230.145
```

across **1,001 distinct destination ports**.

That is a strong indication of vertical port scanning because the scanner is maintaining the same destination while changing the port being probed.

**Finding:**

> `192.168.230.127` performed extensive vertical scanning against `192.168.230.145`, probing 1,001 distinct ports.

---

# 4. Targeted Service Discovery

A second host showed a much smaller set of scanned ports.

The destination was:

```text
192.168.230.1
```

I inspected the connection records for this host:

```bash
grep '192.168.230.1' log-session-2.csv | head
```

The logs showed connections to:

```text
80
445
3389
```

I then sorted these into ascending order:

```text
80, 445, 3389
```

### Service context

| Port | Common service |
| ---: | -------------- |
|   80 | HTTP           |
|  445 | SMB            |
| 3389 | RDP            |

The activity therefore appeared more targeted than the 1,001-port sweep against `192.168.230.145`.

### Analyst interpretation

The combination of HTTP, SMB, and RDP probes suggests an attempt to identify commonly exposed services on the host.

These services can represent meaningful attack paths depending on configuration, exposure, authentication controls, and vulnerabilities.

**Finding:**

> `192.168.230.127` probed `192.168.230.1` on ports `80`, `445`, and `3389`, indicating targeted service discovery.

---

# 5. ICMP Ping Sweep

I also investigated whether the scanning source was attempting to identify live hosts across the internal network.

The source performing the ping sweep was:

```text
192.168.230.127
```

In Elastic/Kibana, I examined ICMP traffic using:

```text
network.protocol:"icmp"
```

The investigation showed repeated ICMP activity from `192.168.230.127` across multiple destination IP addresses.

### Analyst interpretation

A ping sweep differs from a port scan because the immediate objective is **host discovery** rather than service enumeration.

The behavior observed from `192.168.230.127` was consistent with identifying active hosts before or alongside additional scanning activity.

This was particularly significant because the same source was also responsible for the vertical and targeted port-scanning activity identified elsewhere in the investigation.

---

# 6. TCP SYN Scan

I investigated the connection state associated with traffic from:

```text
203.0.113.25
```

to:

```text
192.168.230.145
```

The dominant Zeek connection state was:

```text
S0
```

In Zeek connection logs, `S0` indicates that the originator sent a SYN but no response was observed.

The repeated presence of `S0` across connection attempts was consistent with a:

> **TCP SYN / half-open scan**

### Analyst interpretation

A SYN scan attempts to determine whether ports are accessible without completing a normal TCP connection.

The repeated SYN attempts and lack of completed connections provided additional evidence that the external activity was reconnaissance rather than normal application traffic.

**Finding:**

> `203.0.113.25` performed activity against `192.168.230.145` consistent with a TCP SYN/half-open scan.

---

# 7. UDP Scan Assessment

I also checked the logs for evidence of systematic UDP scanning.

The investigation did contain UDP traffic, including multicast/discovery traffic. However, **UDP traffic by itself does not establish that a UDP scan occurred**.

For a UDP scan, I would expect to see a pattern such as:

* One source probing many UDP ports on a single destination
* One source probing the same UDP port across many destinations
* Repeated ICMP port-unreachable responses associated with UDP probes
* Another clear pattern indicating systematic UDP reconnaissance

The available logs did not show that type of repeated scanning behavior.

Some UDP traffic appeared consistent with normal network discovery/multicast activity rather than a systematic port scan.

**Finding:**

> No clear evidence of a systematic UDP scanning attempt was identified in the available telemetry.

---

# Investigation Summary

The investigation identified multiple forms of network discovery activity originating from both internal and external sources.

| Activity                   | Source            | Target            | Finding                           |
| -------------------------- | ----------------- | ----------------- | --------------------------------- |
| Internal scanning          | `192.168.230.127` | Internal hosts    | 2,276 logged entries              |
| Horizontal scan            | Internal scanner  | `203.0.113.0/24`  | Many destination IPs              |
| Vertical scan              | `192.168.230.127` | `192.168.230.145` | 1,001 ports                       |
| Targeted service discovery | `192.168.230.127` | `192.168.230.1`   | Ports 80, 445, 3389               |
| Ping sweep                 | `192.168.230.127` | Internal subnet   | ICMP host discovery               |
| SYN scan                   | `203.0.113.25`    | `192.168.230.145` | Zeek `S0` pattern                 |
| UDP scan                   | —                 | —                 | No systematic UDP scan identified |

---

# SOC Analyst Takeaways

The most significant finding was the activity associated with:

```text
192.168.230.127
```

The same internal source was associated with multiple discovery behaviors, including:

* ICMP host discovery
* Broad destination scanning
* A 1,001-port vertical scan
* Targeted probing of HTTP, SMB, and RDP

Taken together, these behaviors provide a stronger signal than any individual connection would provide.

From a SOC perspective, I would treat the internal scanning source as a higher-priority investigation than the external reconnaissance because internal scanning can indicate that a system already inside the environment is performing discovery.

The next investigation steps would be to determine:

1. Whether `192.168.230.127` is an authorized scanner or security-management system
2. Which endpoint or user was associated with the source IP
3. Whether the scanning activity was scheduled or expected
4. Whether the source generated authentication, process, or endpoint-security alerts
5. Whether any of the discovered services were subsequently accessed
6. Whether the scanning activity was followed by lateral movement or exploitation

---

# Tools & Techniques Used

**Command line**

* `awk`
* `grep`
* `cut`
* `sort`
* `uniq`
* `head`

**Security tooling**

* Elastic/Kibana
* Zeek connection telemetry
* CSV firewall/SIEM logs

**Analysis techniques**

* Source/destination relationship analysis
* CIDR/network identification
* Unique destination counting
* Unique port counting
* ICMP sweep identification
* Zeek connection-state analysis
* Horizontal vs. vertical scan classification

---

## Key Skills Demonstrated

This investigation demonstrates practical SOC workflow rather than simply identifying answers from a training exercise:

> **Raw Telemetry → Filtering → Pattern Recognition → Scan Classification → Security Interpretation → Investigation Priorities**

The investigation reinforced the importance of analyzing network activity as a pattern rather than evaluating individual connections in isolation.

A single connection may be normal. A source repeatedly touching hundreds of hosts, thousands of ports, or an entire subnet provides a much stronger indication of reconnaissance activity.

## Environment & Attribution

This investigation was completed as part of the TryHackMe Network Discovery Detection training environment.

The analysis was performed independently within the provided Ubuntu VM using the supplied network telemetry, command-line tools, and Elastic/Kibana.

The purpose of this write-up is to document the investigation methodology, evidence, findings, and SOC analyst reasoning demonstrated during the exercise.
