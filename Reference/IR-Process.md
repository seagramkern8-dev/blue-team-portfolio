---
tags: [reference, incident-response]
---

# Incident Response Process

## The four phases (NIST SP 800-61)

1. **Preparation**: logging, tools, baselines, playbooks in place before anything happens
2. **Detection & Analysis**: spot it, confirm it, scope it
3. **Containment, Eradication & Recovery**: stop the spread, remove the attacker, restore systems
4. **Post-Incident Activity**: report, lessons learned, improvements

## My investigation workflow (KC7)

1. **Baseline first.** What's normal? DeviceInfo by VLAN, top ports in NetworkFlow, who's who in Employees.
2. **Start from the lead.** A story tip, alert, IP, or email.
3. **Pivot one step at a time.** IP → domain → machines → connections → URLs → emails → people.
4. **Log every pivot** in the Airtable tracker: IOC row, `Pivoted from`, query in `How found`.
5. **Separate proof from theory.** Only mark something confirmed if the logs or story show it. Timing that "fits" isn't proof.
6. **Build the timeline** from timestamps across tables.
7. **Write the IR** in the standard format, staying inside the story's scope. Anything beyond it goes in Open questions.

## Escalation triggers

Escalate to a lead or on-call (don't sit on it) when I see:

- Confirmed C2 / beaconing from any host
- More than one compromised machine
- Compromise of a critical asset (domain controller, internet-facing server)
- Signs of data leaving the network
- Valid accounts used by an attacker

## Severity guide

| Severity | Example |
|---|---|
| Low | Phishing received, nobody clicked |
| Medium | One user clicked; no malware or login seen |
| High | Malware or C2 on one or more hosts; credentials likely stolen |
| Critical | Domain controller compromise, ransomware, confirmed data theft |

## Indicator handling

- **Never store passwords.** Track *credential state* (Compromised, Reset required, Unknown) instead, and write `[REDACTED]`.
- **Defang** attacker IPs, domains, URLs and emails anywhere public: `1.2.3[.]4`, `evil[.]com`, `hxxps://`, `user@evil[.]com`.
- **Internal** hostnames and private IPs aren't defanged.
- **Confidence**: Low = incomplete or unconfirmed; Medium = strong but indirect; High = seen directly in logs or confirmed by the story.

## IR report sections (my standard)

About & TL;DR · Case details · Summary · Starting point · Timeline · Evidence & key queries · Ruled out · Kill chain / MITRE · Recommendations · Open questions · Lessons learned
