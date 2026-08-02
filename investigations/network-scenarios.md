# Network Investigation Scenarios

## Overview

This document applies networking concepts to common SOC investigation scenarios. The goal is to demonstrate how network knowledge supports alert triage, threat detection, and incident response.

A protocol, port, or connection by itself does not indicate malicious activity. SOC analysts must evaluate context, behavior, and patterns to determine whether activity is expected or suspicious.

---

# Scenario 1: Suspicious DNS Activity

## Situation

A workstation generates hundreds of DNS requests to random-looking domains over a short period of time.

## Potential Concerns

- Malware beaconing
- Command-and-control (C2) communication
- DNS tunneling
- Compromised endpoint

## Investigation Steps

1. Identify the source device making the requests.
2. Review the queried domains.
3. Check domain reputation and age.
4. Look for repeated query patterns.
5. Compare activity against normal user behavior.
6. Determine whether other systems show similar activity.

## Evidence to Review

- DNS logs
- Firewall logs
- Endpoint alerts
- Threat intelligence results

## Analyst Questions

- Are the domains newly registered?
- Do domains have suspicious naming patterns?
- Are requests occurring at regular intervals?
- Is the endpoint communicating with known malicious infrastructure?

---

# Scenario 2: Unexpected RDP Connection

## Situation

A user account connects to a workstation through Remote Desktop Protocol (RDP) outside normal business hours.

## Potential Concerns

- Stolen credentials
- Unauthorized remote access
- Attacker persistence

## Investigation Steps

1. Identify the source IP address.
2. Verify the user account involved.
3. Review authentication logs.
4. Check whether the login location is expected.
5. Review activity after successful access.

## Evidence to Review

- Windows Event Logs
- VPN logs
- Authentication records
- Endpoint security alerts

## Analyst Questions

- Was this login expected?
- Is the source device known?
- Did the user perform unusual actions afterward?
- Were privileged accounts involved?

---

# Scenario 3: SMB Lateral Movement

## Situation

A workstation begins communicating with multiple internal systems over SMB (TCP 445).

## Potential Concerns

- Credential theft
- Lateral movement
- Malware propagation

## Investigation Steps

1. Identify the source and destination systems.
2. Determine whether SMB communication is normal.
3. Review user accounts involved.
4. Check for unusual file access.
5. Look for additional suspicious activity.

## Evidence to Review

- SMB logs
- Windows security events
- Endpoint detection alerts
- Network traffic data

## Analyst Questions

- Does the user normally access these systems?
- Are administrative shares being accessed?
- Are multiple hosts affected?
- Did the activity begin after a security alert?

---

# Scenario 4: Possible Port Scan Activity

## Situation

A host attempts connections to many ports across multiple systems.

## Potential Concerns

- Reconnaissance activity
- Vulnerability discovery
- Compromised internal host

## Investigation Steps

1. Identify the scanning source.
2. Determine whether the behavior is expected.
3. Review targeted systems and ports.
4. Look for successful connections after scanning.

## Evidence to Review

- Firewall logs
- IDS/IPS alerts
- Network flow data

## Analyst Questions

- Is this a security scanner?
- Is the source system authorized?
- Were vulnerable services discovered?
- Did follow-up activity occur?

---

# Scenario 5: Large Outbound Data Transfer

## Situation

A workstation sends an unusually large amount of data to an external destination.

## Potential Concerns

- Data exfiltration
- Compromised account
- Unauthorized file transfer

## Investigation Steps

1. Identify the destination.
2. Determine what data was transferred.
3. Review user activity.
4. Check whether the transfer matches normal behavior.

## Evidence to Review

- Firewall logs
- Proxy logs
- NetFlow data
- Endpoint activity logs

## Analyst Questions

- Is the destination trusted?
- Was the transfer expected?
- Did the user initiate the activity?
- Are sensitive systems involved?

---

# Security Tools and Investigation Workflow

Network activity is rarely investigated in isolation. SOC analysts correlate network findings with endpoint telemetry, SIEM data, and threat intelligence to determine whether activity is normal, suspicious, or requires response.

## SIEM Platforms

Examples:

- Microsoft Sentinel
- Elastic Security
- Splunk

Used for:

- Searching and correlating security events
- Investigating alerts
- Reviewing authentication and network activity
- Creating detection logic

---

## Endpoint Security

Examples:

- Microsoft Defender for Endpoint
- CrowdStrike Falcon

Used for:

- Investigating affected hosts
- Reviewing process activity
- Identifying suspicious endpoint behavior
- Supporting containment actions

---

## Threat Intelligence

Examples:

- VirusTotal
- AbuseIPDB
- AlienVault OTX

Used for:

- Investigating IP addresses
- Reviewing domain reputation
- Enriching indicators of compromise (IOCs)
- Adding context to investigations

---

# Example Investigation Workflow

1. Alert is generated from a SIEM platform.
2. Identify the affected user, device, and network activity.
3. Review endpoint telemetry for suspicious behavior.
4. Investigate related IP addresses, domains, and indicators.
5. Enrich findings using threat intelligence sources.
6. Determine severity and recommended response actions.

---

# Key Takeaway

Network security knowledge allows SOC analysts to move beyond identifying individual connections and understand the story behind network activity.

Effective investigations combine:

- Protocol knowledge
- Traffic patterns
- Logs and telemetry
- Endpoint visibility
- Threat intelligence
- Business context

The goal of a SOC investigation is to determine whether network activity is expected, suspicious, or requires further investigation by combining network data with security telemetry, threat intelligence, and organizational context.
