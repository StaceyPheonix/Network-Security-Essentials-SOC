# Common Network Ports for SOC Analysts

## Overview

Ports identify specific services running on a system. During investigations, analysts use ports to understand what type of communication is occurring and determine whether activity is expected.

| Port | Protocol | Common Use | SOC Relevance |
|---|---|---|---|
| 20/21 | FTP | File Transfer | Unencrypted transfers, possible data exposure |
| 22 | SSH | Secure Remote Access | Brute-force attempts, unauthorized access |
| 23 | Telnet | Remote Access | Insecure legacy protocol |
| 25 | SMTP | Email | Email security investigations |
| 53 | DNS | Name Resolution | Malware communication, DNS tunneling |
| 67/68 | DHCP | IP Assignment | Device identification |
| 80 | HTTP | Web Traffic | Cleartext traffic analysis |
| 110 | POP3 | Email | Legacy email traffic |
| 123 | NTP | Time Synchronization | Log correlation |
| 135 | RPC | Windows Services | Windows communication |
| 139 | NetBIOS | File Sharing | Legacy Windows networking |
| 143 | IMAP | Email | Email investigations |
| 161 | SNMP | Network Management | Device monitoring |
| 389 | LDAP | Directory Services | Authentication investigations |
| 443 | HTTPS | Secure Web Traffic | Encrypted web communication |
| 445 | SMB | Windows File Sharing | Lateral movement, ransomware |
| 514 | Syslog | Log Collection | Security monitoring |
| 3389 | RDP | Remote Desktop | Unauthorized remote access |

---

# SOC Investigation Examples

## Unexpected RDP Connection

Questions:

- Was the login authorized?
- Is the source IP expected?
- Was MFA used?
- Did the account perform suspicious activity afterward?

---

## Unusual DNS Activity

Questions:

- Are domains suspicious?
- Is there high query volume?
- Are domains random-looking?
- Is there possible DNS tunneling?

---

## SMB Lateral Movement

Questions:

- Are systems communicating normally?
- Are privileged accounts involved?
- Are multiple systems being accessed?

---

# Key Takeaway

Ports provide context during investigations. A port number alone does not indicate malicious activity, but it helps analysts understand what behavior to investigate.
