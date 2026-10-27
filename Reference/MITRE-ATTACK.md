---
tags: [reference, mitre]
---

# MITRE ATT&CK Techniques I've Tagged

Techniques used in my KC7 IOC trackers, grouped by tactic. Add the case name in the last column as I use each one.

Look up any technique at `https://attack.mitre.org/techniques/T1566/` (swap in the ID; sub-techniques use a slash, e.g. `T1566/002`).

## Reconnaissance

| ID | Technique | Cases |
|---|---|---|
| T1589.002 | Gather Victim Identity Information: Email Addresses | |
| T1594 | Search Victim-Owned Websites | |
| T1598.003 | Phishing for Information: Spearphishing Link | |

## Resource Development

| ID | Technique | Cases |
|---|---|---|
| T1583.001 | Acquire Infrastructure: Domains | Whiskermania |
| T1583.008 | Acquire Infrastructure: Malvertising | |
| T1585.002 | Establish Accounts: Email Accounts | |

## Initial Access

| ID | Technique | Cases |
|---|---|---|
| T1566 | Phishing | Whiskermania |
| T1566.002 | Phishing: Spearphishing Link | |
| T1078 | Valid Accounts | |

## Execution

| ID | Technique | Cases |
|---|---|---|
| T1204.001 | User Execution: Malicious Link | |
| T1204.002 | User Execution: Malicious File | |
| T1059.001 | Command and Scripting Interpreter: PowerShell | |
| T1053.005 | Scheduled Task/Job: Scheduled Task | |

## Defense Evasion

| ID | Technique | Cases |
|---|---|---|
| T1036 | Masquerading | |
| T1070.004 | Indicator Removal: File Deletion | |

## Credential Access

| ID | Technique | Cases |
|---|---|---|
| T1552.001 | Unsecured Credentials: Credentials In Files | |

## Discovery

| ID | Technique | Cases |
|---|---|---|
| T1016 | System Network Configuration Discovery | |
| T1018 | Remote System Discovery | |
| T1033 | System Owner/User Discovery | |
| T1046 | Network Service Discovery | |

## Lateral Movement

| ID | Technique | Cases |
|---|---|---|
| T1021.001 | Remote Services: Remote Desktop Protocol | |
| T1534 | Internal Spearphishing | |
| T1570 | Lateral Tool Transfer | |

## Collection

| ID | Technique | Cases |
|---|---|---|
| T1039 | Data from Network Shared Drive | |
| T1114.002 | Email Collection: Remote Email Collection | |
| T1560 | Archive Collected Data | |
| T1560.001 | Archive Collected Data: Archive via Utility | |

## Command and Control

| ID | Technique | Cases |
|---|---|---|
| T1071.001 | Application Layer Protocol: Web Protocols | Whiskermania |
| T1105 | Ingress Tool Transfer | |
| T1571 | Non-Standard Port | |
| T1572 | Protocol Tunneling | |

## Exfiltration

| ID | Technique | Cases |
|---|---|---|
| T1048 | Exfiltration Over Alternative Protocol | |
| T1048.002 | Exfiltration Over Asymmetric Encrypted Non-C2 Protocol | |

## Impact

| ID | Technique | Cases |
|---|---|---|
| T1486 | Data Encrypted for Impact | |
| T1565 | Data Manipulation | |

## Cyber Kill Chain (Lockheed Martin)

1. **Reconnaissance**: research the target
2. **Weaponization**: build the payload
3. **Delivery**: send it (phishing email, malicious link)
4. **Exploitation**: victim clicks or opens it
5. **Installation**: malware lands on the machine
6. **Command & Control**: malware calls home (beaconing)
7. **Actions on Objectives**: theft, encryption, sabotage
