# IR: Rap Beef

> **About this report:** This is a portfolio write-up of the KC7 cybersecurity training module "A Rap Beef: An Intro to Security Investigations." The company, people, and logs are simulated. I investigated the logs with KQL queries, and every finding below comes from those query results. Items marked *(story)* come from the module's storyline, not from the logs. The scope matches the module's investigation; questions it leaves open are listed in section 8, each with the query or data I would use to answer it.
>
> **Skills shown:** KQL log analysis across 6 tables, pivoting from one indicator to the next, phishing investigation, timeline building, Cyber Kill Chain and MITRE ATT\&CK mapping, incident response recommendations, and scoping next steps.

## TL;DR

* **What happened:** An attacker reportedly hired by a rival record label *(story)* researched OWL Records' lead rapper, Dwake, on the company website and answered his password-reset security questions using details from his own diss track *(story)*. The attacker then ran a phishing campaign against the label's rappers. Dwake clicked the link, the attacker logged into his account an hour later, searched his mailbox for scandalous material, and downloaded his Drafts folder as `DwakesDirtySecrets.zip`.
* **How I found it:** I started from an IP in a tip and pivoted to web logs, the phishing domain, the phishing emails, Dwake's click, the attacker's login, and the stolen data.
* **Impact:** High. Dwake's email account was compromised and his drafts were stolen, and his Instagram was taken over *(story)*. 13 phishing emails went to 10 rappers, and 11 of them were marked `CLEAN` by the email filter.
* **Next steps:** Contain Dwake's account, block the attacker's IP, domain, and senders, and check whether the other 9 phishing recipients were compromised.

## Case details

| Field             | Entry                                                                                                                                                                                                                       |
| :---------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Case name         | A Rap Beef: An Intro to Security Investigations                                                                                                                                                                             |
| Date investigated | Sep 20, 2026                                                                                                                                                                                                                |
| Analyst           | Seagram Kern                                                                                                                                                                                                                |
| Severity          | High: the lead rapper's email account was compromised and his Drafts folder was stolen. 9 other phishing recipients have not been checked yet.                                                                              |
| Status            | Open: nothing in the logs shows Dwake's password was reset, his sessions were ended, or the attacker's IP and domain were blocked. The truce between the artists is a story ending, not a remediation.                     |
| Data sources used | Employees, InboundNetworkEvents, PassiveDns, Email, OutboundNetworkEvents, AuthenticationEvents. From the story only: Dwake's verse, the tip about the attacker IP, the Instagram post, the dark web post, the phishing hint, and screenshots of the phishing email and page. |

## 1. Summary

In early April 2024, OWL Records rapper Dwake released a diss track aimed at rival rapper Present. The lyrics revealed his mother's maiden name and his childhood pet's name, which are the answers to his account security questions *(story)*. On April 10, an attacker at `18.66.52[.]227`, reportedly hired by Present's label *(story)*, searched the OWL Records website for Dwake's email address, opened his password reset, and answered his security questions with "Washington" and "Fluffy." According to the story, this gave the attacker his email account, which they used to reset his Instagram password and post an embarrassing image the next day.

The same IP hosted `betterlyrics4u[.]com`, the domain behind a phishing campaign: 13 emails offering "free beats" and a "ghostwriter," sent to 10 of the label's rappers between April 10 and April 25 from two free webmail accounts. On April 15, Dwake received "\[EXTERNAL] RE: Need a ghostwriter for your next hit?", which the email filter marked `CLEAN`. He clicked the link at 12:03:12 and likely entered his credentials on a fake login page. At 13:03:12 the attacker logged into his account successfully from `18.66.52[.]227`. The attacker then searched his mailbox for sensitive material and, on April 30, downloaded his Drafts folder as `DwakesDirtySecrets.zip`. The incident is **not contained**.

## 2. Starting point

**What started it:** A tip *(story)*. After Dwake's diss track went viral, a contact in the cyber underground said Dollar Currency Records, Present's label, had hired a hacker who used a specific IP to look around the OWL Records website in early April.

**First indicator:** The IP address `18.66.52[.]227`, from the tip.

**First hypothesis:** The hacker was gathering information on Dwake through our website to get dirt for Present, because the tip came right after the diss track and named an IP browsing our site.

**Result:** Confirmed. InboundNetworkEvents showed the IP searching for Dwake's email address on April 10 and then answering his security questions on the password reset page.

## 3. Timeline

All times are **UTC**.

| Date & time (UTC)                          | What happened                                                                                                                                                                                                  | Source                |
| :----------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------- |
| Early April 2024                           | Dwake's diss track reveals his mother's maiden name (Washington) and childhood pet's name (Fluffy).                                                                                                           | Story                 |
| Early April 2024                           | A tip says Dollar Currency Records hired a hacker using `18.66.52[.]227` to look around the OWL Records website.                                                                                              | Story                 |
| 2024-04-07 07:50:13                        | `betterlyrics4u[.]com` first seen resolving to `18.66.52[.]227`, 3 days before the attack.                                                                                                                    | PassiveDns            |
| 2024-04-10 00:00:00                        | First request from `18.66.52[.]227` on the OWL Records site, a search for "whats Dwake's email address?"                                                                                                      | InboundNetworkEvents  |
| 2024-04-10 10:50:35 to 10:52:38            | 7 more searches for artist contact info, Dwake's email, and booking info, plus an opinion of his music ("why is Dwake music much soo trasshhhh").                                                             | InboundNetworkEvents  |
| 2024-04-10 10:53:07 to 10:57:03            | Visits to Dwake's artist page and the artist email contacts, booking, events, marketing, press release, and legal contact pages.                                                                              | InboundNetworkEvents  |
| 2024-04-10 10:57:47                        | Opens the password reset page with Dwake's email, `dwake_audrey@owl-records.com`.                                                                                                                             | InboundNetworkEvents  |
| 2024-04-10 11:56:47 to 12:53:47            | Submits a support password-reset request ("Forgot my password") and reaches Dwake's security questions.                                                                                                       | InboundNetworkEvents  |
| 2024-04-10 13:02:47                        | Answers both security questions: mother's maiden name "Washington," first pet's name "Fluffy."                                                                                                                | InboundNetworkEvents  |
| 2024-04-10 to 2024-04-25                   | Phishing campaign: 13 emails linking to `betterlyrics4u[.]com` sent to 10 rappers from `ghostwritersanonymous@protonmail[.]com` and `wemakebeatz@gmail[.]com`. 11 marked `CLEAN`, 2 `BLOCKED`.               | Email                 |
| 2024-04-11 (day after the reset)           | Attacker uses Dwake's email to reset his Instagram password and posts an embarrassing image.                                                                                                                 | Story                 |
| After the Instagram post                   | OWL Records leadership, including Dwake, meets privately for 6 hours and declares the issue settled without saying how.                                                                                       | Story                 |
| The next day                               | A hacker on the dark web threatens to leak damaging information on Present unless he announces his retirement within 30 days. A contact suggests any retaliation would come through phishing.                | Story                 |
| 2024-04-15 11:07:12                        | Dwake receives "\[EXTERNAL] RE: Need a ghostwriter for your next hit?", verdict `CLEAN`, signed "ghostwriter."                                                                                               | Email                 |
| 2024-04-15 12:03:12                        | Dwake (`10.10.0.5`) clicks `hxxp://betterlyrics4u[.]com/share/online/published/enter` and reaches a fake login page with a "login to speak with ghost writer" button.                                         | OutboundNetworkEvents |
| 2024-04-15 13:03:12                        | **Account compromised:** attacker logs into `dwaudrey` from `18.66.52[.]227`. Result: Successful Login.                                                                                                       | AuthenticationEvents  |
| 2024-04-16                                 | Attacker searches Dwake's mailbox for "scandal," "confidential," "leak," "secret," and "sensitive," then views his drafts, meeting notes, private emails, and employee complaints.                             | InboundNetworkEvents  |
| 2024-04-30 14:23:02                        | Attacker downloads Dwake's Drafts folder as `DwakesDirtySecrets.zip`. Last activity on Dwake's account from the attacker IP.                                                                                  | InboundNetworkEvents  |
| Later                                      | Dwake and Present agree to a truce.                                                                                                                                                                           | Story                 |

## 4. Evidence

Indicators are listed in the order I pivoted between them.

| Indicator                                                          | Type                      | Where I found it                    | How I found it                                                                                                                                                                                                                                                         |
| :----------------------------------------------------------------- | :------------------------ | :---------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `18.66.52[.]227`                                                   | IP address                | Tip *(story)*, InboundNetworkEvents | Filtered inbound web traffic on this IP for April 10 and got 19 requests: searches about Dwake, visits to artist contact and booking pages, then his password reset.                                                                                                  |
| `Mozilla/5.0 (compatible; MSIE 6.0; Windows NT 6.2; Trident/5.0)` | User agent                | InboundNetworkEvents                | All 19 requests used this string. Internet Explorer 6 never ran on Windows 8 (NT 6.2), so it's fake, and it makes a useful detection signature.                                                                                                                      |
| `dwake_audrey@owl-records.com`                                     | Email address             | InboundNetworkEvents, Employees     | The attacker's searches asked for Dwake's email, which first appears in their requests in the reset URL at 10:57:47. `Employees \| where name has "Dwake"` confirmed it and gave his username (`dwaudrey`), IP (`10.10.0.5`), and role (Lead Rapper).                |
| `/account/reset-password` and `/account/security-questions`        | URLs                      | InboundNetworkEvents                | Same query. The IP opened Dwake's reset at 10:57:47 and submitted "Washington" and "Fluffy" at 13:02:47. The answers match his diss track *(story)*.                                                                                                                 |
| `betterlyrics4u[.]com`                                             | Domain                    | PassiveDns                          | Looked up domains on the attacker IP. First seen 2024-04-07 07:50:13.                                                                                                                                                                                                  |
| `ghostwritersanonymous@protonmail[.]com`, `wemakebeatz@gmail[.]com` | Email addresses (senders) | Email                               | Searched the `link` column for the domain and found 13 phishing emails. `ghostwritersanonymous` sent most of them. 11 were marked `CLEAN`; the 2 from `wemakebeatz` were `BLOCKED`.                                                                                  |
| 10 phishing recipients                                             | People                    | Email, Employees                    | Matched the recipients to employee records. 9 are Rappers and 1 is the Lead Rapper, Dwake Audrey.                                                                                                                                                                     |
| `hxxp://betterlyrics4u[.]com/share/online/published/enter`         | URL                       | Email                               | The link in Dwake's phishing email, "\[EXTERNAL] RE: Need a ghostwriter for your next hit?" (2024-04-15 11:07:12, verdict `CLEAN`).                                                                                                                                  |
| Dwake's click                                                      | Web request               | OutboundNetworkEvents               | Filtered outbound traffic on the phishing URL and Dwake's IP (`10.10.0.5`). He clicked at 2024-04-15 12:03:12.                                                                                                                                                       |
| Attacker login to `dwaudrey`                                       | Login                     | AuthenticationEvents                | Filtered on Dwake's username and the attacker IP. Successful login at 2024-04-15 13:03:12, one hour after his click.                                                                                                                                                 |
| Mailbox searches and `DwakesDirtySecrets.zip`                      | URLs, file name           | InboundNetworkEvents                | Filtered the attacker IP on `url has "dwaudrey"` from April 12 to May 1 and got 10 results: 5 keyword searches, 4 folder views, and the download of his Drafts folder as `DwakesDirtySecrets.zip` on 2024-04-30 14:23:02.                                             |

### Key queries

Attacker activity on the website (start of the investigation):

```kusto
InboundNetworkEvents
| where timestamp between (datetime("2024-04-10T00:00:00") .. datetime("2024-04-11T00:00:00"))
| where src_ip has "18.66.52.227"
```

Pivot from the attacker IP to the phishing domain:

```kusto
PassiveDns
| where ip == "18.66.52.227"
```

Phishing emails, and who they targeted:

```kusto
let _targets = Email
| where link has "betterlyrics4u.com"
| distinct recipient;
Employees
| where email_addr in (_targets)
```

Dwake's click, then the attacker's login:

```kusto
OutboundNetworkEvents
| where url == "http://betterlyrics4u.com/share/online/published/enter"
| where src_ip == "10.10.0.5"

AuthenticationEvents
| where username == "dwaudrey"
| where src_ip == "18.66.52.227"
```

What the attacker did in Dwake's account:

```kusto
InboundNetworkEvents
| where timestamp between (datetime("2024-04-12T00:00:00") .. datetime("2024-05-01T00:00:00"))
| where url has "dwaudrey"
| where src_ip has "18.66.52.227"
```

## 5. What I ruled out

* **Possible explanation:** The April 10 browsing was a fan looking for booking info. **Why I ruled it out:** The same IP went on to open Dwake's password reset and answer his security questions. A fan has no reason to reset his password.
* **Possible explanation:** Dwake reset his own password. **Why I ruled it out:** The reset requests came from `18.66.52[.]227`, not his IP (`10.10.0.5`), right after that IP spent the morning searching for his email address.
* **Possible explanation:** The "ghostwriter" email was a real business offer. **Why I ruled it out:** Its link domain resolves to the attacker's IP and was first seen 3 days before the attack. It came from free webmail, and the same lure went to 10 rappers.
* **Possible explanation:** The April 15 login was Dwake signing in himself. **Why I ruled it out:** It came from the attacker's IP, `18.66.52[.]227`, not his own (`10.10.0.5`), one hour after he clicked the phishing link.
* **Possible explanation:** Dwake downloaded his own Drafts folder. **Why I ruled it out:** The download came from `18.66.52[.]227` and followed the attacker's keyword searches of his mailbox.

## 6. Kill Chain and MITRE ATT\&CK mapping

| Stage                 | What the attacker did                                                                                                                                                                  | MITRE ATT\&CK                                                                                                                               | Evidence                                                                         |
| :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------- |
| Reconnaissance        | Searched the OWL Records site for Dwake's email, contact, and booking info. Took his security-question answers from his diss track *(story)*.                                          | T1594 Search Victim-Owned Websites; T1589.002 Gather Victim Identity Information: Email Addresses; T1593 Search Open Websites/Domains (story) | InboundNetworkEvents, 2024-04-10 00:00:00 to 10:57:03                            |
| Weaponization         | Set up the phishing domain and fake login page, opened free webmail sender accounts, and wrote "ghostwriter" and "free beats" lures aimed at rappers.                                  | T1583.001 Acquire Infrastructure: Domains; T1585.002 Establish Accounts: Email Accounts                                                     | PassiveDns, first seen 2024-04-07 07:50:13                                       |
| Delivery              | Sent 13 phishing emails to 10 rappers. 11, including Dwake's, were marked `CLEAN` and delivered.                                                                                       | T1566.002 Phishing: Spearphishing Link; T1598.003 Phishing for Information: Spearphishing Link                                              | Email, 2024-04-10 to 2024-04-25                                                  |
| Exploitation          | Answered Dwake's security questions on the reset page. Dwake clicked the phishing link and likely entered his credentials, and the attacker used them to log in.                       | T1204.001 User Execution: Malicious Link; T1078 Valid Accounts                                                                              | InboundNetworkEvents 2024-04-10 13:02:47; OutboundNetworkEvents and AuthenticationEvents 2024-04-15 |
| Installation          | No evidence. The attack used stolen credentials, not malware.                                                                                                                         | None                                                                                                                                        | None                                                                             |
| Command & Control     | No evidence.                                                                                                                                                                          | None                                                                                                                                        | None                                                                             |
| Actions on Objectives | Searched Dwake's mailbox for scandalous material and downloaded his Drafts folder as `DwakesDirtySecrets.zip`. Took over his Instagram and posted an embarrassing image *(story)*.     | T1114.002 Email Collection: Remote Email Collection                                                                                         | InboundNetworkEvents, 2024-04-16 and 2024-04-30 14:23:02                         |

## 7. Recommendations

| Action                                                                                                                         | Why (evidence)                                                                                                                                    | Urgency   |
| :----------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ | :-------- |
| **Contain:** Reset Dwake's password and sign out all of his active sessions.                                                   | The attacker logged in on April 15 and was still taking data on April 30. A password reset alone doesn't end a session that's already open.      | Now       |
| **Contain:** Change Dwake's security questions to answers that aren't public.                                                  | The attacker entered his mother's maiden name and pet's name on his reset page, and both were in his lyrics *(story)*.                            | Now       |
| **Contain:** Block `18.66.52[.]227` and `betterlyrics4u[.]com`, and block both sender addresses.                               | Attacker IP, phishing domain, and the senders of all 13 phishing emails.                                                                          | Now       |
| **Contain:** Help Dwake secure his Instagram and any personal accounts tied to his work email (new passwords, MFA).            | His Instagram was taken over through his email *(story)*.                                                                                         | Now       |
| **Scope:** Check whether the other 9 phishing recipients clicked the link and whether the attacker logged into their accounts. | 13 phishing emails reached 10 rappers, and 11 got past the filter. The investigation so far covered Dwake only. See section 8, question 1.        | Now       |
| **Recover:** Review what was in Dwake's Drafts folder and the emails the attacker viewed, then brief management and legal.     | `DwakesDirtySecrets.zip` and the attacker's searches for "scandal," "confidential," and "leak."                                                   | Now       |
| **Detect:** Alert on a password reset or security-question attempt from an IP that just searched for that user.                | The reset at 13:02:47 came from the IP that spent the morning searching for Dwake's email.                                                       | This week |
| **Detect:** Alert on impossible user agents, logins from unfamiliar external IPs, and bulk mailbox downloads.                  | The attacker used a fake IE 6 on Windows 8 string and downloaded a full mailbox folder.                                                           | This week |
| **Detect:** Alert on clicks to newly seen domains that appear in inbound email links.                                          | `betterlyrics4u[.]com` was first seen 3 days before the campaign.                                                                                 | This week |
| **Prevent:** Turn on MFA and replace security questions with a stronger reset method.                                          | A phished password was enough to log in, and public answers were enough to reach the reset page.                                                 | This week |
| **Prevent:** Tune email filtering for "ghostwriter" and "free beats" lures from free webmail.                                  | 11 of 13 phishing emails were marked `CLEAN`.                                                                                                     | Later     |
| **Prevent:** Train artists on phishing and on oversharing personal details in public.                                          | Dwake clicked the phishing link, and his lyrics gave away his security answers *(story)*.                                                         | Later     |

## 8. Open questions

These are questions the module's investigation doesn't answer. Each one lists what I would run or request next.

| # | Question                                                                 | Why it matters                                                                                                                                                                   | How I would answer it                                                                                                                                                         |
| :- | :----------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | Were the other 9 phishing recipients compromised?                        | 11 phishing emails were delivered to 10 rappers, but only Dwake's click and login were investigated. If others clicked, the scope and severity both grow.                       | `OutboundNetworkEvents \| where url has "betterlyrics4u"` for every click, then `AuthenticationEvents \| where src_ip == "18.66.52.227"` for every login from the attacker IP. |
| 2 | Did the April 10 security-question reset actually give the attacker Dwake's email? | The story says it did. But the reset URLs used the username `dwaubrey`, one letter off from his real username `dwaudrey`, and the attacker still phished him on April 15. | Check Dwake's logins between the reset and the phishing login: `AuthenticationEvents \| where username in ("dwaudrey", "dwaubrey") \| where timestamp between (datetime(2024-04-10) .. datetime(2024-04-15))`. |
| 3 | Is the attacker still in Dwake's account?                                | The zip download on April 30 came 15 days after the login on April 15, which suggests a long-lived session.                                                                      | Search all activity from `18.66.52.227` after April 30 in AuthenticationEvents and InboundNetworkEvents, and confirm his sessions were ended.                                 |
| 4 | Did the attacker use other domains, IPs, or senders?                     | Blocking one IP and one domain won't stop the attacker if they have more infrastructure.                                                                                         | `PassiveDns \| where domain == "betterlyrics4u.com"` for other IPs, and search Email for anything else from the two senders.                                                  |
| 5 | Was the phishing campaign planned before the reset?                      | The first phishing email went out on April 10, the same day as the reset and before the story's dark web post. The domain was live on April 7. This affects the timeline of intent. | Compare the earliest Email timestamps for `betterlyrics4u.com` with the PassiveDns first-seen date and the April 10 web activity.                                            |
| 6 | Did the attacker forward mail or set up mailbox rules in Dwake's account? | Forwarding rules can keep stealing mail after a password reset.                                                                                                                 | Mailbox audit and rule logs (not in this dataset).                                                                                                                           |

## 9. Lessons learned

* **Every finding is the next search term.** The IP led to the domain, the domain led to the emails, and the emails led to Dwake's click and the attacker's login.
* **Read the URLs.** The attacker's search queries spelled out what they wanted, including Dwake's email address.
* **Public information breaks security questions.** Two lines in a song were enough to answer Dwake's reset questions.
* **A `CLEAN` verdict doesn't mean safe.** 11 of 13 phishing emails got past the filter, including the one that worked.
* **Check the user agent.** An impossible browser string ties the attacker's activity together and makes an easy detection rule.
* **Follow the attacker past the login.** The data theft happened 15 days after the account was compromised.
* **Scope before you close.** 13 phishing emails went out, but this investigation followed one. Checking the other recipients is the first next step.
