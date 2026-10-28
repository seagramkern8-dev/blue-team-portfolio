# Blue Team Portfolio

**Seagram Kern** · SF Bay Area · ISC2 Certified in Cybersecurity (CC) · [LinkedIn](https://www.linkedin.com/in/seagramkern/)

## About me

I work in physical security, and I'm building toward a blue team role in cybersecurity. The jobs share a core: watch for anything unusual, respond fast, and document what happened. I do that on shift with CCTV monitoring, dispatch calls, and activity reports every 30 minutes. This portfolio shows the same work on the cyber side: hunting through logs, tracing attacks, and writing incident reports. I'm also studying for an Associate of Applied Science in AI Software Engineering.

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
- **Threat intel and phishing:** IOC lookups with VirusTotal, phishing and email header analysis
- **Scripting:** Python, PowerShell basics
- **Tools:** KC7, Airtable (IOC and case tracking), Burp Suite, Obsidian, GitHub
- **Physical security operations:** CCTV monitoring, access control and visitor management, dispatch, incident reporting

## Education and certifications

- **Associate of Applied Science, AI Software Engineering** · Maestro College · in progress, continuing to a BS
- **Associate in Arts, Social Sciences** · Sacramento City College · May 2014
- **ISC2 Certified in Cybersecurity (CC)**
- **Google IT Support Professional Certificate**
- **Thinkful Technical Project Management Certificate**
- **KC7 Cyber Security Analyst I** · completed Oct 2026
- **KC7 Cyber Security Analyst II** · in progress
