# IR: CloutHaus Social Media Leads to Compromise

> **About this report:** This is a portfolio write-up of the KC7 cybersecurity training module "CloutHaus: Social Media Leads to Compromise." The company, people, and logs are simulated. I investigated the logs with KQL queries, and every finding below comes from those query results or the IP lookup provided in the module. Items marked *(story)* come from the module's storyline or screenshots, not from the logs. The scope matches the module's investigation; questions it leaves open are listed in section 8, each with the query or data I would use to answer it.
>
> **Skills shown:** KQL log analysis across 6 tables, pivoting from one indicator to the next (phishing email to credential submission to suspicious login to attacker infrastructure to stolen data), shared-infrastructure analysis with passive DNS, IP geolocation, OSINT, timeline building, Cyber Kill Chain and MITRE ATT\&CK mapping, incident response recommendations, and scoping next steps.

## TL;DR

* **What happened:** An attacker had already tried to break into influencer Afomiya Storm's personal email using security-question answers she shared in an Instagram Q\&A, and MFA stopped them *(story)*. After she joined CloutHaus, the attacker researched her on the company site and sent her a fake Dior partnership email. She entered her password on a lookalike login page. Her CloutHaus account had no MFA, so the attacker logged into her company email from China one hour later. Five days later, 21 emails were forwarded from her account to an attacker address, including her passport scan, bank and tax records, and direct deposit form.
* **How I found it:** I found the fake Dior email in her inbox and her password submission to its link, then pulled her logins and found the attacker's login one hour later. Passive DNS tied the sender domain, the phishing page, and the attacker's forwarding address to the same infrastructure, and the attacker IP led to their recon on the CloutHaus site and the forwarded emails.
* **Impact:** High. Her company email was compromised, and her passport, bank, tax, and payment documents were forwarded to the attacker, along with an email titled "gmail login details" for her personal Gmail. Her Instagram was also taken over and used to scam her followers *(story)*.
* **Next steps:** Reset her CloutHaus password, sign out her sessions, and turn on MFA; check her mailbox for forwarding rules; block the attacker's domains and IPs; and check whether any other employee was targeted.

## Case details

| Field             | Entry                                                                                                                                                                                                                                                                                       |
| :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Case name         | CloutHaus: Social Media Leads to Compromise                                                                                                                                                                                                                                                 |
| Completed         | Sep 27, 2026 (KC7 badge date) |
| Analyst           | Seagram Kern                                                                                                                                                                                                                                                                                |
| Severity          | High: company email taken over without MFA, and her passport, bank, tax, and direct deposit documents were forwarded to an attacker address, along with an email titled "gmail login details" for her personal Gmail.                                                                       |
| Status            | Open: nothing in the logs shows her CloutHaus password was reset, her sessions were ended, MFA was turned on, or the attacker's domains and IPs were blocked. Per the story, Afomiya changed her passwords and turned on MFA *(story)*, but it doesn't say whether that included her CloutHaus account. |
| Data sources used | Employees, Email, OutboundNetworkEvents, PassiveDns, AuthenticationEvents, InboundNetworkEvents. External: IP geolocation for `182.45.67.89` and reverse image search. From the story only: her Instagram profile and Q\&A, the personal email reset attempt (screenshot), the Instagram takeover and scam DMs, her apartment photos, and the mailbox break-in. |

## 1. Summary

Afomiya Storm is a fashion and lifestyle influencer in Washington, D.C. In an Instagram Story Q\&A, she answered follower questions that matched her account security questions. An attacker used those answers to try to reset her personal email password, but the code sent to her phone (MFA) stopped them *(story)*.

Afomiya later joined CloutHaus as an Influencer Partner, and her CloutHaus account did not have MFA. Starting at midnight on April 3, 2025, an attacker at `182.45.67.89` ran searches about her on the CloutHaus site, including her home address. That morning she received a fake Dior partnership email from `collabs@dior-partners[.]com`, marked `CLEAN`, linking to `super-brand-offer[.]com/login`. She clicked the link at 11:20:00 and submitted her username and password two seconds later. At 12:20:00 the attacker logged into her company email from `182.45.67.89` (Jinan, China) with an outdated Internet Explorer user agent.

The attacker kept researching her over the next two days, including her location, her Venmo history, and a fake birthday party to lure her. On April 8, 21 emails were forwarded from her account to `noreply@influencer-deals[.]net` in under 10 minutes. Passive DNS ties that domain, the fake Dior sender, and the phishing page to the same attacker infrastructure. Separately, her Instagram was taken over through a reused password, and the attacker found her apartment through a reverse image search and got into her mailbox from a photo of her keys *(story)*. The incident is **not contained**.

## 2. Starting point

**What started it:** Afomiya's personal accounts had already been targeted using information she shared online *(story)*. After she joined CloutHaus, the module directs a check of her company inbox for brand-deal phishing. It later reports a login to her company email from a different device and country one hour after she clicked a phishing link.

**First indicator:** The fake Dior email in her inbox on 2025-04-03 10:41:00 from `collabs@dior-partners[.]com`, linking to `super-brand-offer[.]com/login`.

**First hypothesis:** She gave her password to a phishing page, because the sender domain isn't Dior's and the Employees table shows MFA disabled on her account.

**Result:** Confirmed. OutboundNetworkEvents showed her password submission at 11:20:02, and AuthenticationEvents showed a successful login from `182.45.67.89` one hour later.

## 3. Timeline

All times are **UTC**. Story rows have no exact times, so they are placed in story order.

| Date & time (UTC)               | What happened                                                                                                                                                                                                                                                                       | Source                |
| :------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------- |
| Before joining CloutHaus        | Afomiya answers follower questions in an Instagram Story Q\&A. Some match her account security questions.                                                                                                                                                                          | Story                 |
| Before joining CloutHaus        | Attacker tries to reset her personal email password with the answers "Lalibela" and "Kidus." The code sent to her phone (MFA) blocks it.                                                                                                                                          | Story (screenshot)    |
| 2024-10-10                      | Afomiya joins CloutHaus as an Influencer Partner. Her account shows MFA disabled.                                                                                                                                                                                                  | Employees             |
| 2025-04-03 00:00:00             | First request from `182.45.67.89` on the CloutHaus site, a search for "Afomiya Storm home address?? (asking for a friend)," nearly 11 hours before the phishing email.                                                                                                             | InboundNetworkEvents  |
| 2025-04-03 10:41:00             | Fake Dior email, "\[EXTERNAL] Exclusive Partnership Opportunity with Dior," from `collabs@dior-partners[.]com` delivered to `afomiya_storm@clouthaus.com`, verdict `CLEAN`, linking to `hxxps://super-brand-offer[.]com/login`. A second copy is logged at the same second with her own address as the sender. | Email                 |
| 2025-04-03 11:12:21 to 12:07:21 | 3 more searches from `182.45.67.89`: her gym and class schedule, her Starbucks order, and whether she has a boyfriend.                                                                                                                                                             | InboundNetworkEvents  |
| 2025-04-03 11:20:00             | Afomiya clicks the link from `10.10.0.3`.                                                                                                                                                                                                                                          | OutboundNetworkEvents |
| 2025-04-03 11:20:02             | Afomiya submits her username (`afstorm`) and password to the fake login page.                                                                                                                                                                                                       | OutboundNetworkEvents |
| 2025-04-03 12:20:00             | **Account compromised:** attacker logs into her company email on `MAIL-SERVER01` from `182.45.67.89` (Jinan, China) with an MSIE 5.0 / Windows NT 5.2 user agent. Result: Successful Login.                                                                                       | AuthenticationEvents  |
| 2025-04-03 12:54:21 to 15:38:21 | 9 more searches, including her brunch spot, her net worth, how to find an influencer's location from an Instagram story, and how to find an apartment from a window view.                                                                                                         | InboundNetworkEvents  |
| 2025-04-04 10:11:29 to 14:38:29 | 12 more searches, including whether her agent has an email, the cost of booking her for a birthday party, reverse image searching an apartment view, and her Venmo history.                                                                                                       | InboundNetworkEvents  |
| 2025-04-04 15:24:29             | Attacker IP requests `clouthaus.com/dashboard/engagement`, an internal-looking page, and the server returns `200` (page served).                                                                                                                                                  | InboundNetworkEvents  |
| 2025-04-07 09:30:48 to 15:15:13 | Logins to her desktop, `OQPA-DESKTOP`, alternate between her internal IP `10.10.0.3` and the external IP `109.52.231.254`, all with her usual Firefox 51 / Windows 10 user agent. Several fail before succeeding, from both IPs.                                                  | AuthenticationEvents  |
| 2025-04-07 12:36:01             | Attacker IP requests `clouthaus.com/resources/social-trends-report` (`200`).                                                                                                                                                                                                      | InboundNetworkEvents  |
| 2025-04-08 15:50:12 to 15:59:54 | 21 emails forwarded from her account to `noreply@influencer-deals[.]net` in under 10 minutes, including her direct deposit form, passport scan, bank statement, tax documents, contracts, invoices, and an email titled "gmail login details." Last attacker activity in this investigation. | Email                 |
| After the phishing              | Her Instagram is taken over through a reused password. The attacker DMs her followers asking for money for fake "investment opportunities" and answers brand deals using her apartment for package deliveries.                                                                   | Story                 |
| After the phishing              | The attacker identifies her building, The Apartments at City Center, by reverse image search of her apartment photos, and gets into her mailbox after she posts a photo of her keys.                                                                                              | Story                 |
| After the phishing              | Afomiya notices suspicious logins, changes her passwords, stops reusing them, and turns on MFA.                                                                                                                                                                                    | Story                 |

## 4. Evidence

Indicators are listed in the order I pivoted between them.

| Indicator                                                                  | Type                             | Where I found it      | How I found it                                                                                                                                                                                                                                                                   |
| :------------------------------------------------------------------------- | :------------------------------- | :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `afomiya_storm@clouthaus.com` / `afstorm` / `10.10.0.3` / `OQPA-DESKTOP`   | Email / Username / IP / Hostname | Employees             | Searched Employees for her name to get the identifiers I needed to pivot into the other logs. Her role is Influencer Partner.                                                                                                                                                   |
| `mfa_enabled: False`                                                       | Account setting                  | Employees             | Same Employees record. Explains why a stolen password alone was enough to log in.                                                                                                                                                                                               |
| `collabs@dior-partners[.]com`                                              | Email address (sender)           | Email                 | Filtered her inbox for "Dior" in the subject or links. The sender domain isn't Dior's, and the email was marked `CLEAN`.                                                                                                                                                       |
| `hxxps://super-brand-offer[.]com/login`                                    | URL                              | Email                 | Read the links field of the fake Dior email.                                                                                                                                                                                                                                    |
| `?username=afstorm&password=...` from `10.10.0.3`                          | URL / Credential submission      | OutboundNetworkEvents | Searched outbound traffic for the phishing URL. Found her click at 11:20:00 and her login submission 2 seconds later.                                                                                                                                                          |
| `198.51.100.12`                                                            | IP address                       | PassiveDns            | Looked up the phishing domain to find its IP, then listed every domain on that IP: 3 domains, `super-brand-offer[.]com` (phishing page), `dior-partners[.]com` (fake Dior sender), and `influencer-deals[.]net` (later the forwarding address).                               |
| `182.45.67.89`                                                             | IP address                       | AuthenticationEvents  | Pulled all logins for `afstorm`. Found a successful email login on `MAIL-SERVER01` one hour after the credential submission. Its password hash (`a2feadde...`) matches her own successful logins, so the attacker had her real password.                                     |
| `MSIE 5.0; Windows NT 5.2`                                                 | User agent                       | AuthenticationEvents  | Same login. Her normal user agent is Firefox 51 on Windows 10, so this didn't match her device.                                                                                                                                                                                 |
| Jinan, Shandong, China                                                     | Geolocation                      | IP lookup (external)  | Looked up `182.45.67.89`. Afomiya is based in Washington, D.C. *(story)*.                                                                                                                                                                                                       |
| `influencer-deals[.]net`, `dior-partners[.]com`                            | Domains                          | PassiveDns            | Listed domains on the login IP. Both are also on `198.51.100.12`, which ties the email login to the fake Dior email and the phishing page.                                                                                                                                     |
| 25 searches and 2 page visits from `182.45.67.89`                          | Web activity (recon)             | InboundNetworkEvents  | Filtered inbound traffic on the attacker IP. Searches from 2025-04-03 00:00:00 to 2025-04-04 14:38:29 covered her home address, location, apartment, Venmo, and a fake birthday party. A `has_any` search on payment keywords narrowed the payment app to Venmo.               |
| `noreply@influencer-deals[.]net`                                           | Email address (recipient)        | Email                 | Searched emails sent from her account with forwarding markers (`[FORWARD]`, `Fwd:`, `FW:`) after the 12:20 login.                                                                                                                                                              |
| 21 forwarded emails                                                        | Email subjects                   | Email                 | Same query, narrowed with financial and document keywords. Includes "\[EXTERNAL] \[FORWARD] Afomiya's payment details – direct deposit form," "...passport scan – confidential," "...bank statement – confidential," "...Tax documents – year-end summary," and "gmail login details" for `afomiyavibes@gmail[.]com`. |
| `109.52.231.254`                                                           | IP address                       | AuthenticationEvents  | Same `afstorm` login query. On 2025-04-07 this external IP logged into `OQPA-DESKTOP` (3 failures, then a success at 14:11:23) with her user agent and password hash, between her own logins from `10.10.0.3`. Unexplained; see section 8.                                   |

### Key queries

Her account details, including MFA status:

```kusto
Employees
| where name contains "Afomiya Storm"
```

The fake Dior email:

```kusto
Email
| where recipient == "afomiya_storm@clouthaus.com"
| where subject contains "Dior" or links contains "Dior"
```

Her click and credential submission:

```kusto
OutboundNetworkEvents
| where url contains "https://super-brand-offer.com/login"
```

Her logins, including the attacker's:

```kusto
AuthenticationEvents
| where username == "afstorm"
```

Attacker infrastructure:

```kusto
PassiveDns
| where ip contains "198.51.100.12"
| distinct domain

PassiveDns
| where ip contains "182.45.67.89"
| distinct domain
```

The attacker's recon on the CloutHaus site:

```kusto
InboundNetworkEvents
| where src_ip contains "182.45.67.89"
| project timestamp, url
```

Everything forwarded to the attacker after the compromise:

```kusto
Email
| where sender == "afomiya_storm@clouthaus.com"
| where recipient == "noreply@influencer-deals.net"
| where subject has_any ("[FORWARD]", "Fwd:", "FW:")
| where timestamp > datetime(2025-04-03 12:20:00)
| project timestamp, recipient, subject
```

## 5. What I ruled out

* **Possible explanation:** Afomiya logged in herself while traveling. **Why I ruled it out:** She normally uses Firefox 51 on Windows 10. The 12:20 login came from Jinan, China with an MSIE 5.0 / Windows NT 5.2 user agent, one hour after she typed her password into the phishing page. She is based in Washington, D.C. *(story)*.
* **Possible explanation:** The Dior email was a real brand offer. **Why I ruled it out:** The sender domain `dior-partners[.]com` isn't Dior's, and passive DNS puts it on the same IPs as the phishing page and the attacker's login.
* **Possible explanation:** Afomiya forwarded the 21 emails herself, for example to an agency address. **Why I ruled it out:** The recipient domain `influencer-deals[.]net` sits on the same attacker IPs, and 21 emails went out in under 10 minutes.
* **Possible explanation:** The 2025-04-07 desktop logins from `109.52.231.254` were a second attacker foothold. **Why it's unlikely (not fully ruled out):** Every attempt used her usual user agent, not the attacker's MSIE 5.0 string, and they alternate with her own logins from `10.10.0.3` throughout the day. They also used her password hash, though, which the attacker had too. See section 8.

## 6. Kill Chain and MITRE ATT\&CK mapping

| Stage                 | What the attacker did                                                                                                                                                                                                         | MITRE ATT\&CK                                                                                                                                       | Evidence                                                                                              |
| :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------- |
| Reconnaissance        | Ran 25 searches about Afomiya on the CloutHaus site, starting nearly 11 hours before the phishing email. Earlier, collected her security-question answers from an Instagram Q\&A *(story)*.                                  | T1594 Search Victim-Owned Websites; T1589 Gather Victim Identity Information; T1593.001 Search Open Websites/Domains: Social Media (story)          | InboundNetworkEvents from `182.45.67.89`, 2025-04-03 00:00:00 to 2025-04-04 14:38:29                  |
| Weaponization         | Set up a fake Dior sender domain, a fake login page, and a forwarding address on shared servers.                                                                                                                             | T1583.001 Acquire Infrastructure: Domains                                                                                                           | PassiveDns: 3 domains on `198.51.100.12`, 2 of them also on `182.45.67.89`                            |
| Delivery              | Sent the fake Dior partnership email with a link to the fake login page.                                                                                                                                                      | T1566.002 Phishing: Spearphishing Link; T1598.003 Phishing for Information: Spearphishing Link                                                      | Email from `collabs@dior-partners[.]com`, 2025-04-03 10:41:00                                         |
| Exploitation          | Afomiya entered her password on the fake page. With MFA disabled, the attacker used it to log into her company email.                                                                                                        | T1204.001 User Execution: Malicious Link; T1078 Valid Accounts                                                                                      | OutboundNetworkEvents 2025-04-03 11:20:02; AuthenticationEvents 2025-04-03 12:20:00; Employees        |
| Installation          | No evidence. The attack used her stolen password, not malware.                                                                                                                                                               | None                                                                                                                                                | None                                                                                                  |
| Command & Control     | No evidence.                                                                                                                                                                                                                  | None                                                                                                                                                | None                                                                                                  |
| Actions on Objectives | Forwarded 21 emails to `noreply@influencer-deals[.]net`, including her passport scan and bank, tax, and payment records. Took over her Instagram to scam followers *(story)*.                                                | T1114.002 Email Collection: Remote Email Collection; T1114.003 Email Forwarding Rule only if a rule was used (unconfirmed)                         | Email, 2025-04-08 15:50:12 to 15:59:54                                                                |

## 7. Recommendations

| Action                                                                                                                             | Why (evidence)                                                                                                                                                         | Urgency   |
| :--------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------- |
| **Contain:** Reset Afomiya's CloutHaus password, sign out all active sessions, and turn on MFA.                                    | The attacker logged in with her stolen password, and `mfa_enabled: False` let it work. A password reset alone doesn't end a session that's already open.            | Now       |
| **Contain:** Check her mailbox for forwarding rules and remove any found.                                                          | 21 emails went to `noreply@influencer-deals[.]net`. Forwarding rules keep working after a password change. See section 8, question 2.                               | Now       |
| **Contain:** Block `182.45.67.89`, `198.51.100.12`, `dior-partners[.]com`, `super-brand-offer[.]com`, and `influencer-deals[.]net`. | Attacker infrastructure for the email, phishing page, login, and forwarding address.                                                                                   | Now       |
| **Contain:** Have Afomiya reset her personal Gmail password and turn on MFA.                                                       | A forwarded email was titled "gmail login details" for `afomiyavibes@gmail[.]com`.                                                                                     | Now       |
| **Contain:** Confirm whether `/dashboard/engagement` should require a login, and restrict it if so.                                | It returned `200` to the attacker IP. See section 8, question 5.                                                                                                       | Now       |
| **Scope:** Check whether any other employee received, clicked, or forwarded anything tied to the attacker's domains.               | The investigation followed Afomiya only, and the attacker's infrastructure could be reused. See section 8, question 1.                                                | Now       |
| **Recover:** Tell Afomiya, HR, and payroll what was exposed, and confirm her direct deposit details haven't been changed.          | Her passport scan, bank statement, tax documents, and direct deposit form were forwarded.                                                                              | Now       |
| **Recover:** Notify legal and affected brand partners that their information was exposed.                                          | Forwarded emails included contracts and invoices.                                                                                                                       | This week |
| **Detect:** Alert on logins from new countries and on outdated user agents.                                                        | The 12:20 login came from China with MSIE 5.0 on Windows NT 5.2.                                                                                                       | This week |
| **Detect:** Alert on bulk forwarding to outside domains.                                                                           | 21 emails were forwarded in under 10 minutes.                                                                                                                          | This week |
| **Prevent:** Require MFA for all employees and check who else has it off.                                                          | MFA blocked the attack on her personal email *(story)*, and its absence on her CloutHaus account let the attacker in.                                                 | This week |
| **Prevent:** Train influencer partners on brand-deal phishing and on oversharing (Q\&A answers, window views, photos of keys).     | She clicked a link in an email tagged `[EXTERNAL]`, and her posts gave away her security answers, building, and keys *(story)*.                                       | Later     |

## 8. Open questions

These are questions the module's investigation doesn't answer. Each one lists what I would run or request next.

| # | Question                                                                    | Why it matters                                                                                                                                                         | How I would answer it                                                                                                                                                                                                       |
| :- | :-------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | Was any other employee targeted?                                            | The investigation followed Afomiya only. If others received or clicked the same lures, the scope grows.                                                                | `Email \| where sender has_any ("dior-partners.com", "super-brand-offer.com", "influencer-deals.net") or links has_any (...) or recipient has "influencer-deals.net"`, plus `OutboundNetworkEvents \| where url has_any (...)` for the same domains. |
| 2 | How were the April 8 forwards sent?                                         | The only attacker login is on April 3, five days earlier. The attacker either kept that session open or set up a forwarding rule, and a rule survives a password reset. | Check AuthenticationEvents for any other logins from the attacker IPs between April 3 and April 8, then mailbox rule and session logs (not in this dataset).                                                               |
| 3 | Did any security alert fire?                                                | A login from China with a 25-year-old browser and a burst of 21 forwards should have raised an alert.                                                                 | `SecurityAlerts \| where description has_any ("afstorm", "afomiya", "182.45.67.89", "OQPA-DESKTOP")`.                                                                                                                     |
| 4 | Who logged into `OQPA-DESKTOP` from `109.52.231.254` on April 7?            | It's an external IP the day before the forwarding. It used her user agent, but also her password hash, which the attacker had too.                                    | Geolocate the IP, check VPN or remote-access logs, and ask Afomiya whether she worked remotely that day.                                                                                                                   |
| 5 | What did the attacker see on `/dashboard/engagement`?                       | It returned `200` to the attacker IP, so the page was served without a login to the site from that IP.                                                                | The page's access settings and content, or web application logs showing what it returned.                                                                                                                                  |
| 6 | When was the attacker infrastructure set up?                                | How far in advance the domains went live shows how planned the attack was, and earlier records may reveal other targets.                                              | `PassiveDns \| where domain has_any ("dior-partners.com", "super-brand-offer.com", "influencer-deals.net") \| summarize first_seen = min(timestamp) by domain, ip`.                                                       |
| 7 | Why is a second copy of the Dior email logged with Afomiya as the sender?   | If the attacker spoofed her address, the mail filter may be trusting internal-looking senders.                                                                       | Pull both rows in full and compare the sender, reply-to, verdict, and headers.                                                                                                                                             |
| 8 | Did Afomiya's account cleanup include her CloutHaus account?                | The story says she changed passwords and turned on MFA, but Employees still shows MFA disabled.                                                                      | Check password-change events and current MFA status for `afstorm`, and confirm with Afomiya.                                                                                                                               |

## 9. Lessons learned

* **MFA decided both attacks.** It blocked the reset on her personal email *(story)*, and its absence on her CloutHaus account let one stolen password through.
* **Check the account settings early.** The Employees record showed `mfa_enabled: False` before I found the login, which explained why the phishing worked.
* **Follow the credential, not just the click.** The URL in OutboundNetworkEvents carried her username and password, which proved she submitted them and gave an exact time to search her logins from.
* **Pivot through passive DNS.** Three different-looking domains (sender, phishing page, forwarding address) sat on shared IPs, which tied the email, the login, and the theft to one attacker.
* **Look at what left, not just who logged in.** The login was one event. The 21 forwarded emails five days later showed the real impact.
* **Oversharing is attack surface.** Q\&A answers, window views, and a photo of her keys each gave the attacker a way in *(story)*, and the attacker's site searches show them working through those same steps.
* **Don't force every odd event into the story.** The `109.52.231.254` desktop logins don't match the attacker's pattern, but they aren't explained either, so I left them as an open question.
