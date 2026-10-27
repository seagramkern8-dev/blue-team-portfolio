# IR: Whiskermania (Payment Shadow Campaign)

> **About this report:** This is a portfolio write-up of the KC7 cybersecurity training module "Whiskermania: An Intro to Network Concepts." The company, people, and logs are simulated. I investigated the logs with KQL queries, and every finding below comes from those query results. Items marked *(story)* come from the module's storyline, not from the logs. The scope matches the module's investigation; questions it leaves open are listed in section 8, each with the query or data I would use to answer it. Section 10 holds extra hunting I did on my own after finishing the module, to answer those open questions.
>
> **Skills shown:** KQL log analysis across 6 tables, network segmentation and IP-range analysis, correlating one event across DNS, network flow, and proxy logs, C2 beaconing detection, a table join to tie IPs to employees, timeline building, Cyber Kill Chain and MITRE ATT\&CK mapping, layered blocking recommendations, and detection rules.

## TL;DR

* **What happened:** An attacker registered a lookalike domain, `whiskersandwonders-hr[.]com`, and sent 9 fake "Employee Benefits Update Required" emails to three Whiskers & Wonders employees with access to donor and adopter data. The employees clicked and malware was installed *(story)*. Their three workstations then contacted a command and control (C2) server at `update-cdn-service[.]xyz` (`185.174.137[.]42`) over HTTPS, checking in at `/api/v1/status` and pulling instructions from `/api/v1/config`. The story names it the Payment Shadow Campaign and says the attacker is after the donation system, five days before the Whiskermania fundraiser *(story)*.
* **How I found it:** I started from an IP on the previous IT admin's post-it note *(story)* and pivoted from DNS to the C2 domain, the three machines that looked it up, their connections and web requests, the phishing emails, and finally, through a join with the Employees table, the three employees.
* **Impact:** High. Three workstations were compromised and talked to C2 for about 5 days, downloading 25 to 98 KB per request. No attacker activity on the donation system is in the logs yet.
* **Next steps:** Block the C2 domain, IP, and URL paths; isolate the three workstations; reset the three users' credentials; protect the DonationPortal before Whiskermania; and find out exactly what the employees clicked and what was installed.

## Case details

| Field             | Entry |
| :---------------- | :---- |
| Case name         | Whiskermania: An Intro to Network Concepts (story campaign name: Payment Shadow Campaign) |
| Date investigated | Oct 25–27, 2026 |
| Analyst           | Seagram Kern |
| Severity          | High: three workstations compromised with active C2 beaconing over 4–5 days, and a financially motivated attacker targeting the donation system before the year's biggest fundraiser *(story)*. |
| Status            | Open: nothing in the logs shows the C2 infrastructure was blocked, the hosts were isolated, or credentials were reset. The proxy allowed every C2 request. Morgan plans to tell the board and possibly law enforcement *(story)*, but that isn't remediation. |
| Data sources used | DeviceInfo, DnsEvents, NetworkFlow, ProxyEvents, Email, Employees. From the story only: Jamie's post-it notes and notebook, the clicks and malware install, the targeting motive, and the campaign name. |
| IOC tracker       | Airtable base "WM" (13 IOCs, 4 people, 7 devices, 3 connections, 5 network zones) |

## 1. Summary

Whiskers & Wonders Cat Sanctuary has 53 devices on 5 network zones: 36 workstations (`10.10.16.x`), 9 servers (`10.10.1.x`), 6 DMZ systems including the DonationPortal (`10.10.0.3`), 1 domain controller in a restricted zone (`10.10.2.x`), and 1 perimeter firewall. The previous IT administrator, Jamie, left suddenly two weeks before Whiskermania, the sanctuary's biggest fundraiser, and left notes asking "Why is 10.10.16.x talking to 185.174.137[.]42?" *(story)*.

On June 2, 2025 at 16:00 UTC, `noreply@whiskersandwonders-hr[.]com` sent "\[EXTERNAL] Important: Employee Benefits Update Required" to Development Officer Alex Rivera and Adoption Coordinators Jessica Huang and David Okonkwo. The real domain is `whiskersandwonders.org`. Alex and Jessica received the same email three more times, on June 3, 4, and 5, always on the hour; David got only the first. The employees clicked and malware was installed *(story)*.

Less than 8 hours later, at 23:52:31, workstation `10.10.16.14` looked up `update-cdn-service[.]xyz`, which resolves to `185.174.137[.]42`. Four seconds later it connected to that IP on port 443 and requested `/api/v1/status`. It came back the next morning for `/api/v1/config`. `10.10.16.7` followed the same pattern on June 5, two hours after the last phishing email, and `10.10.16.24` on June 7. All 6 requests were GET, all were allowed by the proxy, and each downloaded 25 to 98 KB. Joining the DNS lookups to the Employees table confirmed the three machines belong to the three phishing recipients. The incident is **not contained**.

## 2. Starting point

**What started it:** Jamie's post-it note *(story)*: "`185.174.137[.]42` - WHO IS THIS? Check again," with the IP circled twice, and a second note: "Why is 10.10.16.x talking to 185.174.137[.]42? That's not normal."

**First indicator:** The IP address `185.174.137[.]42`, a public IP outside every private range.

**First hypothesis:** Workstations were connecting to an external server that had no business reason to be contacted. Any machine connecting to it would have looked up its domain first, so DNS logs would show which machines and when.

**Result:** Confirmed. DnsEvents showed `update-cdn-service[.]xyz` resolving to the IP, looked up only by three workstations, each followed seconds later by a connection and a web request to C2 paths.

## 3. Timeline

All times are **UTC**.

| Date & time (UTC) | What happened | Source |
| :---------------- | :------------ | :----- |
| About 2 weeks before the investigation | Jamie, the IT administrator, leaves suddenly, leaving notes about `185.174.137[.]42` and the donation system. | Story |
| 2025-06-02 16:00:00 | Phishing email "\[EXTERNAL] Important: Employee Benefits Update Required" from `noreply@whiskersandwonders-hr[.]com` to Alex Rivera, Jessica Huang, and David Okonkwo. | Email |
| After 2025-06-02 16:00 | The targeted employees click and malware is installed. | Story |
| 2025-06-02 23:52:31 | `10.10.16.14` looks up `update-cdn-service[.]xyz` (A record, resolves to `185.174.137[.]42`), then an AAAA lookup that returns nothing. | DnsEvents |
| 2025-06-02 23:52:35 | **First C2 contact:** `10.10.16.14` connects to `185.174.137[.]42:443` and requests `/api/v1/status` (404, 78,521 bytes received). | NetworkFlow, ProxyEvents |
| 2025-06-03 09:13:40 to 09:13:43 | `10.10.16.14` looks up the domain again and requests `/api/v1/config` (200, 63,133 bytes received). | DnsEvents, ProxyEvents |
| 2025-06-03 12:00:00 | Second phishing email to Alex Rivera and Jessica Huang. | Email |
| 2025-06-04 15:00:00 | Third phishing email to Alex Rivera and Jessica Huang. | Email |
| 2025-06-05 16:00:00 | Fourth and last phishing email to Alex Rivera and Jessica Huang. | Email |
| 2025-06-05 18:12:41 to 18:12:44 | `10.10.16.7` looks up the domain and requests `/api/v1/status` (200, 41,318 bytes received). | DnsEvents, ProxyEvents |
| 2025-06-05 18:39:42 to 18:39:44 | `10.10.16.7` requests `/api/v1/config` (301 redirect, 28,362 bytes received). | DnsEvents, ProxyEvents |
| 2025-06-07 19:11:52 to 19:11:55 | `10.10.16.24` looks up the domain and requests `/api/v1/status` (200, 25,291 bytes received), two days after the last phishing email. | DnsEvents, ProxyEvents |
| 2025-06-07 19:54:54 to 19:54:55 | **Last C2 contact:** `10.10.16.24` requests `/api/v1/config` (200, 97,914 bytes received). | DnsEvents, ProxyEvents |
| Investigation (5 days before Whiskermania) | Morgan plans to tell the board and possibly law enforcement. | Story |

## 4. Evidence

Indicators are listed in the order I pivoted between them.

| Indicator | Type | Where I found it | How I found it |
| :-------- | :--- | :--------------- | :------------- |
| `185.174.137[.]42` | IP address | Post-it note *(story)*, DnsEvents, NetworkFlow | Outside all private ranges (10/8, 172.16/12, 192.168/16), so external. Every C2 connection went to this IP on TCP 443. |
| `update-cdn-service[.]xyz`, `www.update-cdn-service[.]xyz` | Domains | DnsEvents | `resolved_ips has "185.174.137.42"` returned only these two names. A cheap `.xyz` domain named to look like boring infrastructure *(story)*. |
| `10.10.16.14`, `10.10.16.7`, `10.10.16.24` | IP addresses (internal) | DnsEvents, DeviceInfo | The only clients that looked up the C2 domain (6, 3, and 3 lookups). All three are in the workstation zone. |
| `5GKV-DESKTOP`, `2BPN-MACHINE`, `IIHO-DESKTOP` | Hostnames | DeviceInfo | `ip_address in (...)` matched the three IPs to workstations in the `workstations` VLAN, each with RDP enabled. |
| 6 connections to `185.174.137[.]42:443` | Network flows | NetworkFlow | Two per workstation, each 3–4 seconds after the matching DNS lookup. |
| `/api/v1/status`, `/api/v1/config` | URL paths | ProxyEvents | Every host requested `/status` (check-in) and then `/config` (instructions), all GET, all `ALLOWED`. Bytes received: 25,291 to 97,914 per request. |
| 3 user agents | User agents | ProxyEvents | Each host used one user agent for both requests: IE 10 on Windows XP (`10.10.16.14`), Firefox 45 on Windows 8.1 (`10.10.16.7`), Chrome 86 on Windows 10 (`10.10.16.24`). The first two are badly outdated. Under review until compared with each host's normal browsing. |
| `whiskersandwonders-hr[.]com` | Domain (lookalike) | Email | `sender !endswith "@whiskersandwonders.org"` and `sender contains "whiskersandwonders"` found the impersonating domain. |
| `noreply@whiskersandwonders-hr[.]com` | Email address (sender) | Email | Sent all 9 emails, Jun 2 to Jun 5, always on the hour, with the same subject. |
| Alex Rivera, Jessica Huang, David Okonkwo | People | Email, Employees | The 3 phishing recipients. A join of the C2 DNS clients to Employees on `ip_addr` returned the same 3 people: `alrivera` (Development Officer), `jehuang` and `daokonkwo` (Adoption Coordinators). |

### Key queries

Queries show indicators as they appear in the logs (not defanged) so they can be copied and run.

From Jamie's IP to the C2 domain (start of the investigation):

```kusto
DnsEvents
| where resolved_ips has "185.174.137.42"
| distinct query_name
```

Which machines looked it up, and when each started:

```kusto
DnsEvents
| where query_name has "update-cdn-service.xyz"
| summarize
    first_beacon = min(timestamp),
    last_beacon = max(timestamp),
    total_queries = count()
  by client_ip
```

Did they connect, and what did they request:

```kusto
NetworkFlow
| where dest_ip == "185.174.137.42"
| project timestamp, src_ip, dest_ip, dest_port, protocol, bytes
| sort by timestamp asc

ProxyEvents
| where domain == "update-cdn-service.xyz"
| project timestamp, src_ip, url, method, status_code
| order by timestamp asc
```

The phishing campaign:

```kusto
Email
| where sender !endswith "@whiskersandwonders.org"
| where sender contains "whiskersandwonders"
| project timestamp, sender, recipient, subject
```

Tying the beaconing machines to people:

```kusto
DnsEvents
| where query_name contains "update-cdn-service.xyz"
| distinct client_ip
| join kind=inner (Employees | project ip_addr, name, username, role) on $left.client_ip == $right.ip_addr
| project name, username, role
```

## 5. What I ruled out

* **Possible explanation:** `update-cdn-service[.]xyz` is a real content delivery network. **Why I ruled it out:** Only three workstations in the whole network ever looked it up, all three were phishing recipients, and they requested `/api/v1/status` and `/api/v1/config` rather than web content. The story confirms it is not a legitimate CDN.
* **Possible explanation:** The traffic was people browsing. **Why I ruled it out:** Each host made exactly two requests to the same two API paths, and the gaps on two hosts were exact whole minutes (27:00 and 43:00), which points to software, not a person.
* **Possible explanation:** The domain is internal. **Why I ruled it out:** It resolves to a public IP and isn't on the company's `on-prem.local` domain.
* **Possible explanation:** The empty `[]` answer in DnsEvents is a second C2 IP. **Why I ruled it out:** The empty rows are AAAA (IPv6) lookups that returned no address. The only IP the domain ever resolved to is `185.174.137[.]42`.
* **Possible explanation:** The three victims were random. **Why I ruled it out:** All three hold roles with access to donor and adopter data and the donation system, and the story confirms they were chosen for that access.

## 6. Kill Chain and MITRE ATT\&CK mapping

| Stage | What the attacker did | MITRE ATT\&CK | Evidence |
| :---- | :-------------------- | :------------ | :------- |
| Reconnaissance | Chose employees with access to the donation system *(story)*. | None confirmed in logs | Story |
| Weaponization | Registered a lookalike HR domain and a C2 domain on a cheap `.xyz` TLD named to look like infrastructure. | T1583.001 Acquire Infrastructure: Domains | Email, DnsEvents |
| Delivery | Sent 9 benefits-themed phishing emails to 3 employees over 4 days. The `[EXTERNAL]` tag didn't stop them. | T1566 Phishing (credential phishing per the story; the link itself isn't in my evidence yet) | Email, 2025-06-02 to 2025-06-05 |
| Exploitation | The employees clicked *(story)*. | T1204 User Execution (story) | Story |
| Installation | Malware was installed on the three workstations *(story)*. | None confirmed in logs | Story |
| Command & Control | Each workstation resolved the C2 domain, connected over HTTPS 443, checked in at `/api/v1/status`, and downloaded data from `/api/v1/config`. | T1071.001 Application Layer Protocol: Web Protocols | DnsEvents, NetworkFlow, ProxyEvents, 2025-06-02 to 2025-06-07 |
| Actions on Objectives | Story says the goal is the donation system and money, before Whiskermania. No attacker activity on the DonationPortal is in the logs yet. | None confirmed in logs | Story |

## 7. Recommendations

| Action | Why (evidence) | Urgency |
| :----- | :------------- | :------ |
| **Contain:** Block `update-cdn-service[.]xyz` (and its `www` subdomain) at DNS. | The malware has to look up the domain before every check-in. | Now |
| **Contain:** Block `185.174.137[.]42` at the firewall. | Every C2 connection went to this IP; it still works if the malware skips DNS. | Now |
| **Contain:** Block `/api/v1/status` and `/api/v1/config` (pattern `/api/v\d+/(status\|config)`) at the proxy. | Still works if the attacker moves to a new domain or IP with the same paths. The proxy allowed all 6 requests. | Now |
| **Contain:** Isolate `5GKV-DESKTOP`, `2BPN-MACHINE`, and `IIHO-DESKTOP`. | All three beaconed to C2 and downloaded data from it. | Now |
| **Contain:** Reset passwords and end sessions for `alrivera`, `jehuang`, and `daokonkwo`. | They were targeted by credential phishing and clicked *(story)*; whether their credentials were used is unknown. | Now |
| **Contain:** Block `whiskersandwonders-hr[.]com` and its sender at the mail gateway. | Source of all 9 phishing emails. | Now |
| **Protect:** Review and lock down the DonationPortal before Whiskermania. | The story names the donation system as the target. The portal is internet-facing in the DMZ with RDP enabled and SMB ports 139/445 open. | Now |
| **Escalate:** Brief leadership, and preserve logs for possible law enforcement. | Active C2 on 3 hosts and a financial motive *(story)*. Morgan plans to notify the board *(story)*. | Now |
| **Detect:** Deploy the three detections from this case: DNS lookups of the C2 domain, NetworkFlow to the C2 IP, and the proxy URL regex. | Each layer caught all three hosts in this investigation. | This week |
| **Detect:** Alert on outdated or unusual user agents and on newly seen `.xyz` domains. | Two hosts used decade-old browser strings, and the C2 domain was on a cheap TLD. | This week |
| **Prevent:** Flag lookalike sender domains that contain the company name. | `whiskersandwonders-hr[.]com` impersonated HR. | This week |
| **Prevent:** Train staff on HR and benefits lures and on `[EXTERNAL]` warnings. | The tag was on every email and the employees still clicked. | Later |

## 8. Open questions

These are questions the module's investigation doesn't answer. Each one lists what I would run or request next.

| # | Question | Why it matters | How I would answer it |
| :- | :------- | :------------- | :-------------------- |
| 1 | What exactly did the employees click, and did they enter credentials? | The story calls it credential phishing. If passwords were stolen, the attacker may also have logged in. | `Email \| where sender contains "whiskersandwonders-hr" \| project link`, then `AuthenticationEvents` for the three usernames after June 2. |
| 2 | What malware was installed? | Needed to clean the hosts and find other infections. | FileCreationEvents and ProcessEvents on the three hosts after each user's click. |
| 3 | What did the hosts download from `/api/v1/config`, and where did the 301 redirect send `10.10.16.7`? | Each host received 25–98 KB, more than a simple check-in. | Proxy logs for the redirect target, or the malware's configuration from the hosts. |
| 4 | Has the attacker touched the DonationPortal? | It's the stated target and Whiskermania is in five days. | `AuthenticationEvents` and `NetworkFlow` for `10.10.0.3`, especially from the three workstations or external IPs. |
| 5 | Are other machines infected but quiet? | A host that connects straight to the IP, or uses another domain, wouldn't show up in the DNS query. | `NetworkFlow \| where dest_ip == "185.174.137.42"` across all sources, and the proxy URL regex across all domains. |
| 6 | Are the outdated user agents malware or the hosts' normal browsers? | Decides whether they're useful detection signatures. | `ProxyEvents \| where src_ip == "<host>" \| summarize count() by user_agent` for each host. |
| 7 | Why does DnsEvents show `dns_server` as `10.1.1.1` when DNS-Server is `10.10.1.7`? | If clients use an unexpected resolver, DNS blocking may not cover them. | Check DeviceInfo and NetworkFlow for traffic to `10.1.1.1` on port 53. |

## 9. Lessons learned

* **Learn the network first.** Knowing that `10.10.16.x` is the workstation zone made it obvious which machines were victims before I knew their names.
* **One action leaves three logs.** Each check-in showed up in DnsEvents, then NetworkFlow, then ProxyEvents, seconds apart. Correlating them proved the activity was the same event.
* **Different tables count differently.** NetworkFlow showed a few hundred bytes per connection, while the proxy showed 25–98 KB received. Checking only one would have understated what the hosts downloaded.
* **Read the question carefully.** "Domain" and "sender address" are different indicators, and "URL path" isn't the domain.
* **Separate proof from timing.** The phishing and beaconing lined up in time, but only the Employees join proved the same three people were involved.
* **Warning labels aren't enough.** Every phishing email carried an `[EXTERNAL]` tag.
* **Escalate early.** Confirmed C2 on several hosts is the point to bring in a lead, not to report "all good."

## 10. Beyond the module: extra hunting

After finishing the module, I kept digging into the open questions from section 8. These findings come from my own queries, not from the module's questions. Answered questions move here from section 8.

### 10.1 Which person owns which workstation?

**Why I checked:** The module's join proved the three beaconing machines belong to the three phishing recipients, but not which machine is whose.

**My query:**

```kusto
DnsEvents
| where query_name contains "update-cdn-service.xyz"
| distinct client_ip
| join kind=inner (Employees | project ip_addr, name, username, role) on $left.client_ip == $right.ip_addr
| project name, username, role, ip_addr
```

**What I found:**

| Person | Username | Role | Workstation | Phishing emails received | First C2 contact (UTC) | Time from last email to first C2 |
| :----- | :------- | :--- | :---------- | :----------------------- | :--------------------- | :------------------------------- |
| David Okonkwo | `daokonkwo` | Adoption Coordinator | `5GKV-DESKTOP` (`10.10.16.14`) | 1 (Jun 2 16:00) | 2025-06-02 23:52:31 | 7 h 52 m |
| Alex Rivera | `alrivera` | Development Officer | `2BPN-MACHINE` (`10.10.16.7`) | 4 (last Jun 5 16:00) | 2025-06-05 18:12:41 | 2 h 12 m |
| Jessica Huang | `jehuang` | Adoption Coordinator | `IIHO-DESKTOP` (`10.10.16.24`) | 4 (last Jun 5 16:00) | 2025-06-07 19:11:52 | 2 days 3 h |

**What it means:**

* **David Okonkwo was the first victim.** He received only one phishing email and his machine was beaconing the same night.
* **The repeat emails went to the two people who hadn't been infected yet.** Alex and Jessica got three follow-ups after June 2; David, already infected, got none. That fits an attacker who stopped emailing whoever had already been compromised, though the logs don't prove the attacker could see that.
* **Alex was infected about two hours after the fourth email**, so the follow-ups likely worked on him.
* **Jessica's machine first beaconed two days after her last email**, so she probably opened an older email late, or her infection is delayed. Which email each person clicked is still open (section 8, question 1).
* Tracker updated: each workstation now has its Owner, and each person their hostname and IP.

