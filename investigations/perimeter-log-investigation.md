# Perimeter Log Investigation: Reconnaissance, VPN Compromise, C2, and Exfiltration

## Investigation Overview

**Source:** TryHackMe SOC Level 1 - Network Security Essentials Lab

This investigation documents the analysis of simulated enterprise network telemetry to identify attacker activity across multiple security data sources.

The objective was to investigate a suspected external threat actor, determine the attack progression, identify compromised systems, and document indicators of compromise.

## Evidence Reviewed

The investigation analyzed:

- Firewall logs
- VPN authentication logs
- IDS alerts

## Investigation Summary

A suspicious external IP address was identified performing reconnaissance against the network perimeter.

Further analysis revealed:

1. Network scanning activity against internal assets
2. Repeated VPN authentication attempts
3. Successful access using a compromised service account
4. Lateral movement attempts against internal systems
5. Command-and-control communication
6. Potential data exfiltration activity

---

# Investigation Findings

## 1. External Reconnaissance Identified

### Evidence

Firewall logs were reviewed to identify the source generating the highest number of blocked requests.

Command used:

```bash
grep "BLOCK" firewall.log | cut -d' ' -f5 | cut -d: -f1 | sort | uniq -c | sort -nr | head
```

### Finding

The IP address responsible for the most reconnaissance activity was:

```
203.0.113.45
```

### Analyst Assessment

The volume of blocked connection attempts suggests automated scanning activity against the organization's perimeter.

---

# 2. Targeted Internal Host Identified

### Evidence

The suspicious external IP was searched within firewall logs.

Command:

```bash
grep "203.0.113.45" firewall.log
```

### Finding

The targeted internal host was:

```
10.0.0.20
```

### Analyst Assessment

The attacker was actively probing internal resources after identifying available targets.

---

# 3. VPN Credential Attack Investigation

### Evidence

VPN authentication logs were reviewed for failed login attempts.

Command:

```bash
cat vpn_auth.log | grep FAIL
```

The highest volume of failed authentication attempts originated from:

```
203.0.113.45
```

Further investigation identified repeated attempts against:

```
svc_backup
```

Example:

```
2025-09-03 02:18:40 203.0.113.45 svc_backup FAIL
2025-09-03 02:19:40 203.0.113.45 svc_backup SUCCESS assigned_ip=10.8.0.23
```

### Analyst Assessment

The attacker successfully authenticated after multiple failed attempts, indicating a possible brute-force attack or compromised credentials.

---

# 4. Unauthorized VPN Access Confirmed

### Finding

The successful VPN login assigned the attacker:

```
10.8.0.23
```

### Analyst Assessment

The attacker obtained internal network access and could potentially interact with internal systems.

---

# 5. Lateral Movement Activity Observed

### Evidence

Firewall logs were reviewed for internal scanning and service access attempts.

Potential lateral movement services identified:

| Service | Port |
|---|---|
| SMB | TCP 445 |
| RDP | TCP 3389 |
| SSH | TCP 22 |

### Finding

The attacker attempted SMB-based lateral movement:

```
TCP 445
```

### Analyst Assessment

SMB activity after unauthorized access is suspicious because attackers commonly use it for internal discovery and lateral movement.

---

# 6. Command-and-Control Communication Detected

### Evidence

IDS alerts were reviewed for C2 activity.

Command:

```bash
cat ids_alerts.log | grep C2 | head
```

Alert:

```
ET TROJAN Possible C2 Beaconing

10.0.0.60:30000 -> 198.51.100.77:4444
```

### Finding

Compromised host:

```
10.0.0.60
```

Communicated with:

```
198.51.100.77
```

### Analyst Assessment

The repeated beaconing pattern indicates possible malware communication with external command-and-control infrastructure.

---

# 7. Potential Data Exfiltration Identified

### Evidence

Firewall logs were filtered for communication with the identified C2 destination.

Command:

```bash
cat firewall.log | grep "198.51.100.77"
```

Observed communication:

```
10.0.0.51 -> 198.51.100.77
```

### Finding

The host associated with potential exfiltration activity:

```
10.0.0.51
```

### Analyst Assessment

Repeated outbound communication to an external destination may indicate unauthorized data transfer and requires further investigation.

---

# Attack Timeline

| Phase | Evidence | Finding |
|---|---|---|
| Reconnaissance | Firewall BLOCK events | 203.0.113.45 scanning activity |
| Credential Attack | VPN failures | svc_backup targeted |
| Initial Access | VPN SUCCESS event | Internal IP 10.8.0.23 assigned |
| Lateral Movement | Firewall activity | SMB TCP 445 |
| Command and Control | IDS alerts | 10.0.0.60 communicating with 198.51.100.77 |
| Exfiltration | Firewall traffic | 10.0.0.51 outbound communication |

---

# Investigation Lessons Learned

This investigation demonstrated how SOC analysts correlate multiple telemetry sources to reconstruct attacker activity.

Key skills practiced:

- Firewall log analysis
- VPN authentication analysis
- IDS alert investigation
- Network traffic investigation
- Command-line filtering
- Indicator identification
- Attack chain reconstruction

A single alert rarely tells the complete story. Effective SOC investigations require connecting individual events across different security tools to understand attacker behavior and determine impact.
