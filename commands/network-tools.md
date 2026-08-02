# Network Tools Reference for SOC Analysts

## Overview

This document covers common networking tools used during security investigations, troubleshooting, and threat analysis.

SOC analysts use these tools to gather information about hosts, validate network activity, identify suspicious behavior, and support incident response investigations.

---

# Ping

## Purpose

Ping tests connectivity between systems using ICMP packets.

## Example

```
ping 8.8.8.8
```

## SOC Investigation Use

A ping response alone does not indicate whether a system is safe or malicious. Analysts use it as an initial validation step before performing deeper investigation.

---

# Traceroute / Tracert

## Purpose

Shows the network path packets take between a source and destination.

## Commands

Windows:

```
tracert example.com
```

Linux:

```
traceroute example.com
```

## SOC Investigation Use

Useful for:

- Understanding network paths
- Identifying routing issues
- Investigating unexpected destinations

---

# Nslookup

## Purpose

Queries DNS servers to resolve domain names and IP addresses.

## Example

```
nslookup example.com
```

## SOC Investigation Use

Analysts use DNS lookups to investigate:

- Suspicious domains
- Domain resolution changes
- Potential command-and-control infrastructure

## Investigation Questions

- Does the domain resolve to a suspicious IP?
- Is the domain newly registered?
- Is the domain associated with malicious activity?

---

# Dig

## Purpose

Provides detailed DNS information.

## Example

```
dig example.com
```

## SOC Investigation Use

Useful for reviewing:

- DNS records
- Mail exchange records
- Name server information
- Suspicious DNS configurations

---

# Netstat

## Purpose

Displays active network connections and listening services.

## Examples

Windows:

```
netstat -ano
```

Linux:

```
netstat -tulpn
```

## SOC Investigation Use

Analysts can review:

- Active connections
- Listening ports
- Unexpected services
- Suspicious outbound communication

## Investigation Questions

- Is this connection expected?
- What process owns this connection?
- Is the destination suspicious?

---

# IPconfig / Ifconfig

## Purpose

Displays network interface configuration.

## Commands

Windows:

```
ipconfig
```

Linux:

```
ifconfig
```

## SOC Investigation Use

Useful for identifying:

- Host IP addresses
- Network interfaces
- DNS configuration
- Default gateways

---

# ARP

## Purpose

Displays local IP-to-MAC address mappings.

## Example

```
arp -a
```

## SOC Investigation Use

Can help identify:

- Devices on the local network
- Unexpected MAC addresses
- Local network communication issues

---

# Curl

## Purpose

Transfers data from URLs and interacts with web services.

## Example

```
curl -I https://example.com
```

## SOC Investigation Use

Useful for:

- Reviewing HTTP headers
- Testing web connectivity
- Investigating suspicious URLs

---

# Whois

## Purpose

Provides domain registration information.

## Example

```
whois example.com
```

## SOC Investigation Use

Analysts may investigate:

- Domain ownership
- Registration dates
- Suspicious infrastructure
- Potential threat actor infrastructure

---

# Wireshark

## Purpose

Wireshark is a network protocol analyzer used for packet capture and traffic analysis.

## SOC Investigation Use

Used for:

- Inspecting packets
- Analyzing suspicious communications
- Identifying protocols
- Investigating PCAP files
- Extracting network indicators

## Examples of Analysis

- DNS queries
- HTTP traffic
- TCP handshakes
- Suspicious connections
- Data transfers

---

# NetworkMiner

## Purpose

NetworkMiner is a network forensic analysis tool used to extract artifacts from packet captures.

## SOC Investigation Use

Used for:

- Extracting files
- Identifying hosts
- Reviewing sessions
- Gathering network evidence

---

# Investigation Workflow Example

1. Alert identifies suspicious network activity.
2. Use SIEM logs to identify the affected host.
3. Validate network information using tools such as:
   - nslookup
   - netstat
   - whois
4. Analyze traffic using:
   - Wireshark
   - NetworkMiner
5. Enrich findings with threat intelligence.
6. Document findings and determine response actions.

---

# Key Takeaway

Networking tools provide visibility into how systems communicate. SOC analysts use these tools together with SIEM platforms, endpoint security solutions, and threat intelligence to investigate suspicious activity and support incident response.
