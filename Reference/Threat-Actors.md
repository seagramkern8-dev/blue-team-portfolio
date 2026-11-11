---
tags: [reference, threat-intel]
---

# Threat Actor Reference Sheet

Who attacks, why, and what that means for the defender. Built from the KC7 module *The Blame Game: Why Hackers Be Hacking* (Security Analyst II).

**Core idea:** motivation tells you more about an attacker than any single technical indicator. It shapes what they target, how patient they are, and how much time you have.

## The four categories

| Category | Main motivation | Typical behavior | Example |
|---|---|---|---|
| **Nation-state** (often called an APT) | Espionage, sabotage, geopolitical advantage; sometimes revenue for the state | Government-funded teams, custom tools and zero-days, long quiet dwell times (historically 200+ days) | APT28 / Fancy Bear (Russia), APT41 (China), Lazarus Group (North Korea) |
| **Cybercriminal** | Money | Run like businesses: Ransomware as a Service (RaaS), affiliates keeping 60–80%, Initial Access Brokers, double extortion | DarkSide (Colonial Pipeline, 2021), LockBit, Conti |
| **Insider threat** | Malicious (money, spite, recruited by others) or negligent (mistakes) | Uses legitimate access, so it looks like normal work. Caught with behavior analytics, DLP and access logs | Fake job offers and LinkedIn recruitment by foreign intelligence |
| **Hacktivist** | Ideology and attention | Fast, loud, public: defacements, leaks, DDoS, with public statements | Anonymous |

**Capability is not motivation.** "Script kiddie" describes skill, not why someone attacks. A low-skill attacker can be a criminal, a hacktivist or just curious.

## Why the lines blur

- **States acting like criminals:** North Korea's Lazarus Group steals from banks and crypto exchanges to fund the regime.
- **Hacktivists as possible state proxies:** KillNet called itself patriotic volunteers; researchers assessed it as at least tolerated by Russian state interests.
- **False flags:** Olympic Destroyer (2018 Winter Olympics) carried planted North Korean and Chinese code fragments; it was traced to Russia's Sandworm.
- **Manufactured insiders:** nation-state operatives get hired under fake identities (see FAMOUS CHOLLIMA below).

## Five groups to know

| Group | Category | What they do | Why it matters |
|---|---|---|---|
| **Volt Typhoon** | Nation-state (China) | Quietly compromised US critical infrastructure (water, energy, transportation, communications) using only built-in Windows tools ("living off the land"). Some access lasted up to five years. | Pre-positioning for sabotage, not data theft. Look for admin tools (PowerShell, wmic, netsh) used in unusual ways. |
| **Salt Typhoon** | Nation-state (China) | Compromised at least nine US telecom providers and their lawful-intercept systems, collecting call metadata on more than a million users. | Same sponsor as Volt Typhoon, different mission (counterintelligence). States run specialized teams. |
| **Scattered Spider** (Octo Tempest) | Cybercriminal (ransomware affiliates) | Young English-speaking members who call IT helpdesks, impersonate staff, SIM-swap and spam MFA prompts, then deploy partner ransomware. MGM lost over $100M; Caesars paid $15M. | The helpdesk is an attack surface. Verify identity before resetting credentials or MFA. |
| **FAMOUS CHOLLIMA** (Jasper Sleet) | Nation-state (North Korea) **and** insider threat | Operatives use stolen or AI-generated identities and deepfakes to get hired as remote IT workers, collect salaries for the regime, and steal data. US "laptop farms" helped hide them. | No clean box. Hiring and onboarding checks are security controls. |
| **SiegedSec** | Hacktivist | Leaked documents and defaced US state government sites over anti-transgender legislation (2022–2024). No financial demands, no known state sponsor; announced it was disbanding in 2024. | Clear ideology and public claims are the hacktivist tell. GhostSec shows a group's focus can shift over time. |

## How I use this during an incident

1. **Contain first, attribute later.** The priority order is contain, eject the attacker, restore, then investigate. Naming the attacker is mostly an intelligence and policy job.
2. **Read the motive from the evidence.** Data staged for theft and a ransom note point to crime. Long quiet access with no payoff points to a state. Public claims point to hacktivists.
3. **Hold attribution loosely.** One indicator is not attribution. Use several independent sources and state a confidence level.
4. **Let motive set urgency.** Ransomware crews move from access to encryption in days; state actors may wait months; hacktivists want visibility now.

## Mapping my KC7 cases

| Case | Actor type (from the story) | Tell |
|---|---|---|
| [Rap Beef](../KC7/Rap-Beef/IR.md) | Hacker-for-hire | Rival label paid a hacker to dig up dirt |
| [CloutHaus](../KC7/CloutHaus/IR.md) | Cybercriminal | Financial and identity documents forwarded out |
| [A Scandal in Valdoria](../KC7/Valdoria/IR.md) | Hacktivist | "Hacktivist manifesto" script and a fake political article |
| [Jojo's Hospital](../KC7/Jojos-Hospital/IR.md) | Access broker + ransomware group | One group sold access, another encrypted and extorted |
| [Whiskermania](../KC7/Whiskermania/IR.md) | Cybercriminal | Targeted staff with access to the donation system |

*Sources: KC7 Cyber module material, plus public reporting from CISA, Microsoft, CrowdStrike and Mandiant as summarized in the module.*
