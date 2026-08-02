# TCP vs UDP for SOC Analysts

## Overview

TCP and UDP are transport layer protocols responsible for moving data across networks. Understanding their differences helps SOC analysts interpret network traffic and identify suspicious behavior.

---

# TCP (Transmission Control Protocol)

## Characteristics

TCP is:

- Connection-oriented
- Reliable
- Ordered
- Error-checked

TCP establishes communication using a three-way handshake:

Client → SYN

Server → SYN/ACK

Client → ACK

## Common TCP Applications

- HTTP/HTTPS
- SSH
- SMB
- RDP
- FTP

## SOC Relevance

TCP analysis can help identify:

- Failed connections
- Port scanning
- Suspicious remote access
- Data transfers

## Investigation Questions

- Are connections completing successfully?
- Are many ports being scanned?
- Is communication occurring with unusual destinations?

---

# UDP (User Datagram Protocol)

## Characteristics

UDP is:

- Connectionless
- Faster
- Does not guarantee delivery
- Does not establish a handshake

## Common UDP Applications

- DNS
- DHCP
- VoIP
- Streaming
- NTP

## SOC Relevance

UDP is commonly reviewed during investigations involving:

- DNS abuse
- Network scanning
- Data exfiltration
- Suspicious communication patterns

## Investigation Questions

- Is there unusual UDP volume?
- Are random external destinations being contacted?
- Is traffic occurring on unexpected ports?

---

# TCP vs UDP Comparison

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable delivery | Best effort |
| Speed | Slower | Faster |
| Ordering | Maintains order | No guarantee |
| Handshake | Required | Not required |
| Common Uses | Web, SSH, SMB | DNS, DHCP, streaming |

---

# SOC Analyst Takeaway

Neither TCP nor UDP is automatically malicious. Analysts investigate whether the communication pattern, destination, timing, and behavior are expected or suspicious.
