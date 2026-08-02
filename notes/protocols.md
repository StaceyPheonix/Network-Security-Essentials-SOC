# Network Protocols for SOC Analysts

## Overview

Network protocols define how systems communicate and exchange information. For SOC analysts, understanding protocols helps with alert triage, traffic analysis, threat hunting, and identifying suspicious activity.

---

# DNS (Domain Name System)

## Purpose

DNS translates domain names into IP addresses so systems can locate services across networks.

Example:
www.example.com → 93.184.216.34

## Common Security Uses

SOC analysts review DNS activity to identify:

- Malware communication
- Command-and-control (C2) infrastructure
- Suspicious domains
- DNS tunneling
- Newly registered domains

## Investigation Questions

- Is the domain reputation known?
- Is the domain newly created?
- Are multiple endpoints querying the same suspicious domain?
- Are DNS requests unusually frequent?

---

# HTTP / HTTPS

## Purpose

HTTP and HTTPS are protocols used for web communication.

HTTP sends data without encryption, while HTTPS uses TLS encryption to protect communication.

## Security Relevance

SOC analysts may investigate:

- Suspicious web traffic
- Malicious downloads
- Phishing links
- Command-and-control traffic

## Investigation Questions

- What domains are users accessing?
- Are there unusual user agents?
- Are connections going to known malicious infrastructure?
- Are there large outbound transfers?

---

# SSH (Secure Shell)

## Purpose

SSH provides secure remote access to systems.

## Security Relevance

SSH is commonly targeted through:

- Brute-force attacks
- Credential attacks
- Unauthorized remote access

## Investigation Questions

- Are there repeated failed login attempts?
- Is the source IP expected?
- Is access occurring outside normal hours?

---

# SMB (Server Message Block)

## Purpose

SMB allows Windows systems to share files, printers, and resources.

## Security Relevance

SMB is important in investigations involving:

- Lateral movement
- Credential abuse
- Ransomware activity

## Investigation Questions

- Is SMB traffic occurring between unusual systems?
- Are privileged accounts accessing multiple machines?
- Are there signs of remote file access?

---

# RDP (Remote Desktop Protocol)

## Purpose

RDP allows remote graphical access to Windows systems.

## Security Relevance

Attackers often abuse RDP for:

- Unauthorized access
- Credential attacks
- Persistence

## Investigation Questions

- Who initiated the connection?
- Was the login successful?
- Is the source location expected?

---

# DHCP (Dynamic Host Configuration Protocol)

## Purpose

DHCP automatically assigns network configuration information such as IP addresses.

## Security Relevance

DHCP data can help analysts:

- Identify devices
- Track IP ownership
- Investigate suspicious hosts

---

# Key Takeaway

Understanding protocols allows SOC analysts to interpret network activity, recognize abnormal behavior, and investigate potential security incidents.
