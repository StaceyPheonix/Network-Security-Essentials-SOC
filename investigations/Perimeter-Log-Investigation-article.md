# Perimeter Log Investigation: Reconnaissance, VPN Compromise, C2, and Potential Exfiltration

**Author:** Stacey Menley  
**Publication Date:** August 15, 2026  
**Publication Type:** Technical Article / Security Investigation  
**Training Source:** TryHackMe SOC Level 1 - Network Security Essentials Lab  

---

## Investigation Overview

Effective network security investigations rarely depend on a single log entry. A suspicious connection, failed VPN authentication, or IDS alert may appear insignificant when viewed independently. However, correlating events across multiple network telemetry sources can reveal a much larger attack sequence.

As part of the TryHackMe SOC Level 1 **Network Security Essentials** lab, I investigated simulated enterprise network telemetry to identify reconnaissance activity, a potential VPN credential compromise, internal network activity, command-and-control communication, and potential data exfiltration.

The investigation provided practical experience analyzing firewall logs, VPN authentication records, and IDS alerts while using command-line filtering to identify indicators and reconstruct the potential progression of an attack.

---

## Evidence Reviewed

The investigation analyzed multiple sources of network security telemetry:

- Firewall logs
- VPN authentication logs
- IDS alerts
- Source and destination IP addresses
- Network service ports
- Authentication activity
- Internal and external network communications

The objective was to identify suspicious activity, correlate related events, determine affected systems, and reconstruct the potential attack sequence.

---

## 1. External Reconnaissance Identified

I began by reviewing firewall logs for blocked connection attempts originating from external IP addresses. The objective was to identify a source generating an unusually high volume of perimeter activity.

I used command-line filtering to isolate blocked events and identify the most active source:

`grep "BLOCK" firewall.log | cut -d' ' -f5 | cut -d: -f1 | sort | uniq -c | sort -nr | head`

The analysis identified **203.0.113.45** as the external IP address responsible for the highest volume of blocked requests.

The volume and repetitive nature of the connection attempts were consistent with automated reconnaissance or network scanning.

I then searched the firewall logs for additional activity associated with this IP address:

`grep "203.0.113.45" firewall.log`

This demonstrated the importance of moving beyond simply identifying a suspicious source and determining which internal assets were being targeted.

---

## 2. VPN Credential Attack Investigation

I next examined VPN authentication logs for repeated failed authentication attempts.

`cat vpn_auth.log | grep FAIL`

The same external IP address, **203.0.113.45**, appeared repeatedly in the failed authentication activity.

The targeted account was:

`svc_backup`

A subsequent successful authentication appeared in the logs:

`2025-09-03 02:18:40 203.0.113.45 svc_backup FAIL`

`2025-09-03 02:19:40 203.0.113.45 svc_backup SUCCESS assigned_ip=10.8.0.23`

The transition from repeated failures to a successful authentication was a significant investigative finding.

The successful VPN connection assigned the source an internal VPN address:

`10.8.0.23`

This indicated that the external actor potentially obtained authenticated access to the internal network.

The activity was consistent with a possible brute-force attack or use of compromised credentials. Additional authentication and endpoint evidence would be required to determine the exact cause of the successful login.

---

## 3. Unauthorized VPN Access Confirmed

The successful VPN authentication provided an important pivot point for the investigation.

The external source:

`203.0.113.45`

successfully authenticated as:

`svc_backup`

and received:

`10.8.0.23`

as its internal VPN address.

This changed the investigation from an external reconnaissance event to a potential unauthorized internal network access event.

Once an external actor obtains authenticated VPN access, subsequent internal network activity becomes important for determining whether the account was used for discovery, lateral movement, command execution, or data access.

---

## 4. Lateral Movement Activity Observed

Following the successful VPN authentication, I reviewed network activity for services commonly associated with internal administration and lateral movement.

The investigation focused on:

| Service | TCP Port |
|---|---:|
| SSH | 22 |
| SMB | 445 |
| RDP | 3389 |

SMB activity over **TCP 445** was identified.

SMB is a legitimate enterprise protocol used for file sharing and Windows network services. However, SMB activity following suspicious authentication warrants additional investigation because attackers can use SMB for network discovery, remote access, file transfer, and lateral movement.

This finding demonstrates why network analysts must evaluate protocols in context rather than treating individual services as inherently malicious.

---

## 5. Command-and-Control Communication Detected

I then examined IDS alerts for evidence of command-and-control activity.

`cat ids_alerts.log | grep C2 | head`

An IDS alert identified:

**ET TROJAN Possible C2 Beaconing**

`10.0.0.60:30000 -> 198.51.100.77:4444`

The communication identified:

- **Potentially compromised host:** `10.0.0.60`
- **External destination:** `198.51.100.77`
- **Destination port:** `4444`

The repeated beaconing pattern was consistent with possible malware communication with external command-and-control infrastructure.

However, network indicators alone do not establish that a host is definitively compromised. Additional endpoint, DNS, packet-level, and process telemetry would normally be reviewed to validate the finding.

---

## 6. Potential Data Exfiltration Identified

Finally, I searched firewall telemetry for additional communication with the identified external C2 destination:

`cat firewall.log | grep "198.51.100.77"`

The investigation identified outbound communication from:

`10.0.0.51`

to:

`198.51.100.77`

This activity was assessed as **potential data exfiltration** requiring further investigation.

Network traffic alone does not establish that data was successfully exfiltrated.

Additional evidence that could be examined includes:

- Traffic volume
- Destination reputation
- DNS activity
- Protocol and port behavior
- Packet contents where available
- Endpoint processes
- File-access activity
- Data-transfer patterns
- Historical baseline behavior

This distinction is important because a SOC analyst should differentiate between a suspicious indicator, a working hypothesis, and a confirmed security finding.

---

## Attack Timeline

| Attack Phase | Evidence | Finding |
|---|---|---|
| Reconnaissance | Firewall BLOCK events | `203.0.113.45` scanning activity |
| Credential Attack | VPN failures | `svc_backup` targeted |
| Initial Access | VPN SUCCESS event | `10.8.0.23` assigned |
| Lateral Movement | Firewall activity | SMB / TCP 445 |
| Command and Control | IDS alert | `10.0.0.60 → 198.51.100.77:4444` |
| Potential Exfiltration | Firewall traffic | `10.0.0.51 → 198.51.100.77` |

---

## Network Investigation Methodology

The investigation followed a structured network-analysis process:

1. Identify anomalous perimeter activity.
2. Determine the source IP generating the activity.
3. Identify targeted internal systems.
4. Correlate the source with authentication logs.
5. Investigate successful authentication following repeated failures.
6. Determine the internal address assigned through VPN access.
7. Examine subsequent internal network activity.
8. Identify suspicious service usage and potential lateral movement.
9. Review IDS telemetry for command-and-control indicators.
10. Search network logs for additional communications with identified infrastructure.
11. Assess potential data-transfer activity.
12. Correlate the evidence into an attack timeline.

This approach demonstrates the value of combining network telemetry sources rather than investigating each event independently.

---

## Network Security Skills Demonstrated

This investigation provided hands-on practice with:

- Firewall log analysis
- Network traffic investigation
- IP address analysis
- Internal and external network identification
- VPN authentication analysis
- Network reconnaissance detection
- TCP service identification
- SMB traffic analysis
- IDS alert investigation
- Command-and-control detection
- Potential exfiltration analysis
- Command-line log filtering
- Indicator identification
- Event correlation
- Attack-chain reconstruction
- Security investigation documentation

---

## Network+ Relevant Technical Concepts

The investigation reinforced several concepts relevant to network security and network operations, including:

- IPv4 addressing
- Internal versus external addressing
- Network perimeter monitoring
- TCP/IP communications
- TCP service ports
- VPN remote-access technology
- Firewall traffic filtering
- Network scanning and reconnaissance
- SMB network services
- RDP and SSH services
- Network intrusion detection
- Network traffic analysis
- Source and destination identification
- Authentication-related network events
- Command-and-control traffic
- Network-based indicators of compromise
- Data-transfer analysis
- Correlation of network telemetry

Understanding these concepts is important for identifying abnormal network behavior and determining how systems communicate across internal and external network boundaries.

---

## Lessons Learned

A major lesson from this investigation is that **a single alert rarely tells the complete story**.

The initial firewall activity identified an external source, but additional log sources were required to determine what happened next. By correlating firewall events with VPN authentication records and IDS alerts, it became possible to construct a potential sequence from reconnaissance through credential attack, unauthorized access, lateral movement, C2 communication, and potential exfiltration.

The investigation also reinforced the importance of understanding normal network behavior. Protocols such as SMB, VPN traffic, and outbound connections are not inherently malicious. Their significance depends on context, timing, source, destination, frequency, and relationship to other security events.

Another important lesson was the distinction between **detection and confirmation**. A suspicious connection or IDS alert should generate an investigative hypothesis, not an unsupported conclusion. Additional telemetry should be collected before declaring a system compromised or confirming data exfiltration.

Finally, the exercise demonstrated how simple command-line techniques can rapidly reduce large volumes of telemetry into actionable evidence. Filtering, sorting, grouping, and correlating events allowed me to move from individual log entries toward a reconstructed attack sequence.

---

## Conclusion

This investigation demonstrated a practical SOC workflow for analyzing network security telemetry and reconstructing attacker behavior.

By examining firewall logs, VPN authentication events, IDS alerts, IP addresses, and network services together, I was able to identify indicators associated with reconnaissance, possible credential compromise, unauthorized VPN access, lateral movement, command-and-control communication, and potential data exfiltration.

The investigation reinforced a fundamental security operations principle:

**Effective network analysis requires connecting individual events across multiple telemetry sources to understand attacker behavior, identify affected systems, assess potential impact, and determine the next investigative step.**

The exercise strengthened practical skills in network monitoring, traffic analysis, IP addressing, service identification, log analysis, event correlation, and security investigation-skills directly applicable to Security Operations Center (SOC) environments.
