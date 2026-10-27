# Cyber Analyst Portfolio

**Seagram Kern** · SF Bay Area · ISC2 Certified in Cybersecurity (CC) · [LinkedIn](https://www.linkedin.com/in/seagramkern/)

## About me

I'm a security officer moving into SOC analysis. On shift I monitor CCTV, handle dispatch calls, and write a timed activity report every 30 minutes. This portfolio builds the other half of the job: investigating logs, finding what happened, and writing it up clearly. I'm also studying for an Associate of Applied Science in AI Software Engineering.

## Investigations

Each report follows the same structure: TL;DR, timeline, evidence and key KQL queries, what I ruled out, Cyber Kill Chain and MITRE ATT&CK mapping, recommendations, open questions, and lessons learned.

> All investigations use simulated training data from [KC7 Cyber](https://kc7cyber.com). Attacker network indicators are defanged. Items marked *(story)* come from the module's storyline rather than the logs.

| Case | Attack type | Severity | Report |
|---|---|---|---|
| Rap Beef | Phishing, account takeover, data theft | High | [Incident report](KC7/Rap-Beef/IR.md) |
| CloutHaus | Phishing, account takeover, data theft | High | [Incident report](KC7/CloutHaus/IR.md) |
| A Scandal in Valdoria | Phishing, remote access, disinformation | High | [Incident report](KC7/Valdoria/IR.md) |
| Jojo's Hospital | Malvertising, access broker, ransomware | Critical | [Incident report](KC7/Jojos-Hospital/IR.md) |
| Whiskermania | Credential phishing, malware / C2 | High | [Incident report](KC7/Whiskermania/IR.md) |

## Reference sheets

Notes I keep and update as I learn:

- [KQL cheat sheet](Reference/KQL.md)
- [Ports, protocols and network basics](Reference/Ports-Protocols.md)
- [MITRE ATT&CK techniques I've mapped](Reference/MITRE-ATTACK.md)
- [Incident response process](Reference/IR-Process.md)

## Skills and tools

- **Log analysis:** KQL (Azure Data Explorer) across email, DNS, network flow, proxy, authentication, file, and process logs
- **Investigation:** pivoting between indicators, timeline building, IOC tracking, defanging
- **Frameworks:** Cyber Kill Chain, MITRE ATT&CK, NIST incident response
- **Tools:** KC7, Airtable (IOC and case tracking), Burp Suite, Obsidian, GitHub
