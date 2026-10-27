---
tags: [reference, networking]
---

# Ports, Protocols & Network Basics

## Common ports

| Port | Protocol | What it's for | Security note |
|---|---|---|---|
| 22 | SSH | Remote command line | Brute-force target |
| 25 | SMTP | Sending email | Phishing arrives this way |
| 53 | DNS | Domain name lookups (mostly UDP) | Lookups show intent; heavy TCP 53 can mean tunneling |
| 80 | HTTP | Unencrypted web | |
| 110 / 995 | POP3 / POP3S | Receiving email | |
| 139 / 445 | NetBIOS / SMB | Windows file sharing | Lateral movement, ransomware spread |
| 143 / 993 | IMAP / IMAPS | Receiving email | |
| 443 | HTTPS | Encrypted web | Attackers hide C2 here because firewalls allow it |
| 587 | SMTP submission | Authenticated email sending | |
| 3389 | RDP | Windows Remote Desktop | Common way in when exposed |
| 8443 | Alt-HTTPS | Web apps / admin panels | |

Port ranges: **0–1023** well-known · **1024–49151** registered · **49152–65535** ephemeral (temporary client ports). Max port: **65535**.

## TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Full name | Transmission Control Protocol | User Datagram Protocol |
| Connection | Three-way handshake (SYN, SYN-ACK, ACK) | None ("connectionless") |
| Reliability | Confirms delivery, resends lost data | Fire and forget |
| Used for | Web, email, file transfer | DNS, streaming, gaming, VoIP |

## IP addresses

- **IPv4** = 4 octets, each 0–255, 32 bits total (e.g. `10.10.16.5`).
- **Private ranges (RFC 1918)**, internal only, not reachable from the internet:
  - `10.0.0.0` – `10.255.255.255`
  - `172.16.0.0` – `172.31.255.255`
  - `192.168.0.0` – `192.168.255.255`
- Everything else is **public**. Examples: `8.8.8.8` (Google DNS), `1.1.1.1` (Cloudflare DNS).
- **Defang public attacker IPs** when sharing: `185.174.137[.]42`.

## Network zones (segmentation)

| Zone | Purpose | Exposure |
|---|---|---|
| DMZ | Internet-facing services: web, proxy, donation/VPN portals | Reachable from the internet |
| Servers | Internal business systems: mail, DNS, databases | Internal |
| Restricted | Crown jewels: domain controller | Internal, minimal access |
| Workstations | Employee computers | Internal; can reach out to the internet |

A **VLAN** groups devices logically. Segmentation limits the blast radius of a compromise.

## DNS

| Record | Maps |
|---|---|
| A | Domain → IPv4 |
| AAAA | Domain → IPv6 |
| MX | Domain → mail server |
| CNAME | Alias → another domain |
| TXT | Text (SPF, DKIM, verification) |
| NS | Domain → name servers |
| PTR | IP → domain (reverse DNS) |

**TTL** = Time To Live, how long (in seconds) a lookup is cached.

## One web request, many logs

1. **DnsEvents**: workstation asks DNS for the domain's IP (UDP 53)
2. **NetworkFlow**: workstation connects to that IP (TCP 443)
3. **ProxyEvents**: proxy logs the full URL and response

Correlate by timestamp: these should be seconds apart.

## HTTP

| Status | Meaning |
|---|---|
| 200 | OK, success |
| 301 / 302 | Redirect (302s to odd domains can mean compromise) |
| 403 | Forbidden (lots of them can mean probing) |
| 404 | Not found |
| 500 | Server error (can mean exploitation attempts) |

| Method | Meaning |
|---|---|
| GET | Retrieve data; parameters show in the URL and logs |
| POST | Submit data (forms, logins, uploads) |
| PUT | Update data |
| DELETE | Remove data |

## C2 (Command and Control)

- **Beaconing:** malware checks in on a schedule. Regular intervals are the giveaway; people browse randomly.
- Small, repeated transfers to the same external host over 443 = classic check-in pattern.
- Lookalike infrastructure: cheap TLDs like `.xyz`, boring names like `update-cdn-service`.
