# IR: Jojo's Hospital

> **About this report:** This is a portfolio write-up of the KC7 cybersecurity training module "Jojo's Hospital: A Ransomware Investigation." The company, people, and logs are simulated. I investigated the logs with KQL queries, and every finding below comes from those query results. Items marked *(story)* come from the module's storyline or the ransom note screenshots, not from the logs. The scope matches the module's investigation; questions it leaves open are listed in section 8, each with the query or data I would use to answer it.
>
> **Skills shown:** KQL log analysis across 7 tables, host-based investigation of process command lines, pivoting through passive DNS to map attacker infrastructure, separating two threat actors in one intrusion, timeline building, Cyber Kill Chain and MITRE ATT\&CK mapping, incident response recommendations, and scoping next steps.

## TL;DR

* **What happened:** A two-stage attack. First, an access broker the story calls SharkFin7 used a fake restaurant ad to get hospital staff to open a malicious document, which launched Cobalt Strike. From employee Anthony Davis's machine they stole network diagrams and a credentials file *(story: they then sold the access)*. Weeks later, a second group, LockByte, logged in with Anthony's account, stole patient records from three server folders, and spread ransomware through a shared folder *(story: group names)*.
* **How I found it:** I started from the ransom note, which was on only one machine. That machine's process history showed the ransomware copy, the patient data theft, and the upload domain. Passive DNS on that domain led to the second group's IPs, their website research, and their login to Anthony's account. A staff report of a fake sponsored ad *(story)* then led to the first victim's machine, Cobalt Strike, and the theft of the network files from Anthony's machine.
* **Impact:** Critical. 6,420 files encrypted on 321 hosts, three archives of patient records exfiltrated, and network diagrams and credentials stolen. Patients were threatened with the release of their records and Social Security numbers *(story)*.
* **Next steps:** Isolate the infected hosts, remove the ransomware from the file share, reset every credential in the stolen file, block both groups' infrastructure, notify affected patients, and find every host that ran Cobalt Strike.

## Case details

| Field             | Entry                                                                                                                                                                                                                                                                                                         |
| :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Case name         | Jojo's Hospital: A Ransomware Investigation                                                                                                                                                                                                                                                                   |
| Date investigated | Oct 6, 2026                                                                                                                                                                                                                                                                                                   |
| Analyst           | Seagram Kern                                                                                                                                                                                                                                                                                                  |
| Severity          | Critical: 6,420 files encrypted on 321 hosts, patient records from three server folders exfiltrated, and network diagrams and a credentials file stolen. Two separate attacker groups were active.                                                                                                           |
| Status            | Open: the last attacker activity in this investigation is the deletion of the patient data zips on 2024-06-17 17:36:47, but nothing in the logs shows any infected host was isolated, the ransomware was removed from the shared folder, or the stolen credentials were reset.                                |
| Data sources used | FileCreationEvents, ProcessEvents, Employees, OutboundNetworkEvents, PassiveDns, InboundNetworkEvents, AuthenticationEvents. From the story only: the ransom note and patient extortion email, the staff report of the sponsored ad, the sale of the credentials on the dark web, and the group names SharkFin7 and LockByte. |

## 1. Summary

On May 1, 2024, a hospital employee clicked a fake ad for Raising Cane's, a restaurant across the street that's popular with staff, at the lookalike domain `raisinkanes[.]com` *(story: sponsored search result)*. The ad led to `Raisin_Kane_Promo_Offer.docx`. When it was opened in Word on `RQJQ-MACHINE`, `cobaltstrike.exe` ran and connected to `93.238.22[.]122` on port 50050. The next day the attacker ran discovery commands on that machine. On May 14, Cobalt Strike also ran on Anthony Davis's machine, `AMFB-MACHINE`. The story says the attacker moved laterally to reach it *(story)*. From there they scanned the network and, on May 16, uploaded network diagrams and a credentials file to `nothing-to-see-here[.]net`.

Per the story, SharkFin7 sold Anthony's credentials to the LockByte ransomware group on the dark web *(story)*. On May 20, someone logged into his account from `203.0.113[.]1`, and over the next four weeks the same IPs searched the hospital website for ways to bypass security and for patient records. On June 17, they copied `lockbyte_ransomer.exe` to the hospital file share as `spread_ransomware.exe`, downloaded `patient_data_exporter.exe` from `secure-health-access[.]com`, and began exporting patient records. At 14:49 that day, 6,420 files on 321 hosts were encrypted, and the ransom note `We_Have_Your_Data_Pay_Up.txt` appeared on Anthony's machine. The attacker then finished exporting records from three server folders, uploaded them to `secure-health-access[.]com`, and deleted the local copies. They gave the hospital 72 hours to pay and demanded $10,000 from each patient *(story)*. The incident is **not contained**.

## 2. Starting point

**What started it:** A ransomware attack *(story)*. Files across the hospital were encrypted, and the attackers demanded payment from the hospital and threatened patients.

**First indicator:** The ransom note `We_Have_Your_Data_Pay_Up.txt`, found on only one host: `AMFB-MACHINE`, which belongs to Anthony Davis.

**First hypothesis:** Because the note appeared on one machine out of 321 encrypted hosts, I suspected `AMFB-MACHINE` was where the attacker worked from.

**Result:** Confirmed. Its process history on June 17 showed the ransomware being copied to the file share and patient data being exported and uploaded.

## 3. Timeline

All times are **UTC**. Story rows have no exact times, so they are placed in story order.

| Date & time (UTC)                          | What happened                                                                                                                                                                                                                                       | Source                |
| :----------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------- |
| Before 2024-05-01                          | A staff member reports a sponsored search result for Raising Cane's pointing to `raisinkanes[.]com` instead of `raisingcanes.com`.                                                                                                                 | Story                 |
| 2024-05-01 09:56:40 to 2024-05-14 12:04:24 | 26 requests from 24 internal IPs to `raisinkanes[.]com`, redirecting to `nothing-to-see-here[.]net` and `totally-legit-domain[.]com`.                                                                                                              | OutboundNetworkEvents |
| 2024-05-01 09:56:50                        | First download of `Raisin_Kane_Promo_Offer.docx`, on `RQJQ-MACHINE` (user `evbrowne`), by `chrome.exe`.                                                                                                                                            | FileCreationEvents    |
| 2024-05-01 09:57:16                        | Word opens the docx, and `cobaltstrike.exe` runs one second later. It connects to `93.238.22[.]122` on port 50050 by 10:26:17.                                                                                                                    | ProcessEvents         |
| 2024-05-02 15:45:54 to 2024-05-03 10:30:50 | 6 discovery commands on `RQJQ-MACHINE`: `systeminfo`, `ipconfig /all`, `netstat -an`, `net user`, `net localgroup administrators`, and `net view`.                                                                                                | ProcessEvents         |
| 2024-05-14 12:24:45                        | Cobalt Strike on `AMFB-MACHINE` connects to `93.238.22[.]123` on port 50050. Per the story, the attacker moved laterally to this machine.                                                                                                         | ProcessEvents, Story  |
| 2024-05-15 10:58:02 to 13:48:02            | The same 6 discovery commands on `AMFB-MACHINE`.                                                                                                                                                                                                    | ProcessEvents         |
| 2024-05-16 10:00:05                        | `advanced-ip-scanner.exe /silent` runs on `AMFB-MACHINE` to map the network.                                                                                                                                                                       | ProcessEvents         |
| 2024-05-16 11:25:08 to 13:39:48            | `network_diagrams.pdf` and `credentials.txt` are copied, zipped into `important_network_info.zip`, and uploaded with `curl` to `hxxp://nothing-to-see-here[.]net/upload`.                                                                          | ProcessEvents         |
| After 2024-05-16                           | SharkFin7 sells access to LockByte, and Anthony's credentials are bought on the dark web.                                                                                                                                                          | Story                 |
| 2024-05-20 00:00:00                        | **Second group's access:** login to Anthony Davis's account from `203.0.113[.]1`.                                                                                                                                                                  | AuthenticationEvents  |
| 2024-05-20 00:00:00 to 2024-06-17 13:23:51 | 37 requests to the hospital website from `203.0.113[.]1` and `203.0.113[.]2`, including searches for bypassing security and for patient records.                                                                                                  | InboundNetworkEvents  |
| 2024-06-17 13:35:12                        | `lockbyte_ransomer.exe` is copied to `\\jojos-hospital.org\shared\spread_ransomware.exe`.                                                                                                                                                         | ProcessEvents         |
| 2024-06-17 14:22:29                        | `patient_data_exporter.exe` is downloaded from `secure-health-access[.]com`.                                                                                                                                                                       | OutboundNetworkEvents |
| 2024-06-17 14:23:25                        | The exporter creates `patient_data_1.zip` from `\\jojos-hospital-server\important_data\patient_records`.                                                                                                                                          | ProcessEvents         |
| 2024-06-17 14:49:02                        | Ransom note `We_Have_Your_Data_Pay_Up.txt` is written to `C:\Users\andavis\Documents\` on `AMFB-MACHINE`.                                                                                                                                         | FileCreationEvents    |
| 2024-06-17 14:49:30                        | **Encryption:** 6,420 files on 321 hosts are renamed `.encrypted`, all logged at the same second.                                                                                                                                                 | FileCreationEvents    |
| 2024-06-17 14:56:02 to 15:54:53            | The exporter keeps running after encryption and creates `patient_data_2.zip` and `patient_data_3.zip` from two more patient folders.                                                                                                              | ProcessEvents         |
| 2024-06-17 17:18:57 to 17:36:47            | The three zips are uploaded with `curl` to `hxxp://secure-health-access[.]com/upload/`, then deleted with `cmd.exe /c del C:\Users\andavis\Documents\patient_data_*.zip`.                                                                         | ProcessEvents         |
| After encryption                           | The ransom note gives the hospital 72 hours to pay. Patients receive emails demanding $10,000 each and threatening to release their health history, medical information, and Social Security numbers.                                             | Story (screenshots)   |

## 4. Evidence

Indicators are listed in the order I pivoted between them.

| Indicator                                                                                       | Type                   | Where I found it                     | How I found it                                                                                                                                                                                                      |
| :---------------------------------------------------------------------------------------------- | :--------------------- | :----------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `.encrypted` files (6,420 on 321 hosts)                                                         | File extension         | FileCreationEvents                   | `filename endswith ".encrypted"`, then `count` and `distinct hostname` to scope the damage.                                                                                                                        |
| `We_Have_Your_Data_Pay_Up.txt` (SHA256 `97c348e95c8a8aeb8808f76434d73a92bbcb6b4586788365762b22624990b018`) | File                   | FileCreationEvents                   | Searched for the ransom note by name. It appeared on only one host, `AMFB-MACHINE`, at `C:\Users\andavis\Documents\`.                                                                                             |
| Anthony Davis (`andavis`)                                                                       | Employee               | Employees                            | `where hostname == "AMFB-MACHINE"`.                                                                                                                                                                                 |
| `lockbyte_ransomer.exe` → `\\jojos-hospital.org\shared\spread_ransomware.exe`                   | File names / Share path | ProcessEvents                        | Read the 14 process events on `AMFB-MACHINE` for June 17, sorted by time. Putting the ransomware on a shared folder let it reach many machines.                                                                   |
| `patient_data_exporter.exe` and `patient_data_1.zip` to `patient_data_3.zip`                    | Tool / File names      | ProcessEvents                        | Same June 17 query. The `/source` arguments showed three patient record folders under `\\jojos-hospital-server\important_data\`.                                                                                  |
| `secure-health-access[.]com`                                                                    | Domain                 | ProcessEvents, OutboundNetworkEvents | The `curl` upload commands, then `where url has "patient_data_exporter.exe"`, showed the tool was downloaded from the same domain at 14:22:29.                                                                     |
| `cmd.exe /c del C:\Users\andavis\Documents\patient_data_*.zip`                                  | Command                | ProcessEvents                        | Same June 17 query. The attacker deleted the zips after uploading them.                                                                                                                                             |
| `203.0.113[.]1`, `203.0.113[.]2`, `emr-help[.]net`                                              | IPs / Domain           | PassiveDns                           | Pivoted on the upload domain to get its 2 IPs, then on the IPs to find a second domain.                                                                                                                             |
| 37 website requests                                                                             | Web activity           | InboundNetworkEvents                 | `where src_ip in (...)` on the two IPs, then filtered on "bypass" and "patient." The first patient search was `https://jojoshospital.org/search=JoJo%27s+Hospital+patient+records`.                                |
| Login to Anthony's account from `203.0.113[.]1`                                                 | Login                  | AuthenticationEvents                 | Same two IPs. One login, 2024-05-20 00:00:00.                                                                                                                                                                       |
| `raisinkanes[.]com`, `nothing-to-see-here[.]net`, `totally-legit-domain[.]com`                  | Domains                | OutboundNetworkEvents                | Searched for the lookalike domain from the reported ad *(story)*. 26 requests from 24 internal IPs, redirecting to two more attacker domains.                                                                      |
| `Raisin_Kane_Promo_Offer.docx` (SHA256 `bd886046266b909a8ca5f19f16e5606baf73194a70632c81fdc44ef39ba29712`) and `Raisin_Kane_Free_Meal_Voucher.pdf` | Files                  | FileCreationEvents                   | Searched for `.docx` and `.pdf` files. The first docx download was on `RQJQ-MACHINE` at 2024-05-01 09:56:50, by `chrome.exe`.                                                                                     |
| `cobaltstrike.exe` → `93.238.22[.]122:50050`                                                    | Malware / C2           | FileCreationEvents, ProcessEvents    | Looked at files and processes on `RQJQ-MACHINE` after the download. Word opened the docx, and `cobaltstrike.exe` ran and connected to the attacker's IP on port 50050.                                            |
| `systeminfo` and 5 other discovery commands                                                     | Commands               | ProcessEvents                        | `RQJQ-MACHINE` on May 2 and 3. The same 6 ran on `AMFB-MACHINE` on May 15.                                                                                                                                         |
| `cobaltstrike.exe` → `93.238.22[.]123:50050` on `AMFB-MACHINE`                                  | Malware / C2           | ProcessEvents                        | Searched Anthony's machine for `cobaltstrike` or the C2 IP. First connection at 2024-05-14 12:24:45, to a neighboring IP.                                                                                         |
| `advanced-ip-scanner.exe`, `network_diagrams.pdf`, `credentials.txt`, `important_network_info.zip` | Tool / Files           | ProcessEvents                        | `AMFB-MACHINE` command lines from May 13 to 17: a network scan, then copying, zipping, and uploading the files to `nothing-to-see-here[.]net`.                                                                     |

**Note:** the logs use two spellings of the hospital domain: `jojos-hospital.org` for the file shares and `jojoshospital.org` for the website. This looks like a quirk of the module data, so I kept each spelling as it appears in the logs. Both are the hospital's own domains, so they aren't defanged.

### Key queries

The damage, and where the ransom note landed:

```kusto
FileCreationEvents
| where filename endswith ".encrypted"
| distinct hostname

FileCreationEvents
| where filename == "We_Have_Your_Data_Pay_Up.txt"
```

The second group's work on Anthony's machine:

```kusto
ProcessEvents
| where hostname == "AMFB-MACHINE"
| where timestamp between (datetime(2024-06-17) .. datetime(2024-06-18))
```

Pivoting from the upload domain to the second group's IPs, website research, and login:

```kusto
PassiveDns
| where domain == "secure-health-access.com"
| distinct ip

PassiveDns
| where ip in ("203.0.113.1", "203.0.113.2")

InboundNetworkEvents
| where src_ip in ("203.0.113.1", "203.0.113.2")

AuthenticationEvents
| where src_ip in ("203.0.113.1", "203.0.113.2")
```

The fake ad and the first victim:

```kusto
OutboundNetworkEvents
| where url has "raisinkanes.com"
| project src_ip, url

FileCreationEvents
| where filename == "Raisin_Kane_Promo_Offer.docx"
```

The first group's work on Anthony's machine:

```kusto
ProcessEvents
| where hostname == "AMFB-MACHINE" or username == "andavis"
| where process_commandline has "cobaltstrike" or process_commandline has "93.238.22.122"
| project timestamp, hostname, username, process_commandline
| sort by timestamp asc

ProcessEvents
| where hostname == "AMFB-MACHINE"
| where timestamp between (datetime(2024-05-13) .. datetime(2024-05-17))
```

## 5. What I ruled out

* **Possible explanation:** Anthony Davis ran the ransomware himself. **Why I ruled it out:** His machine was running Cobalt Strike from May 14, and his account was logged into on May 20 from the external IP `203.0.113[.]1`, one of the two IPs behind `secure-health-access[.]com`, where the patient data was uploaded.
* **Possible explanation:** The restaurant ad visits were normal lunch browsing. **Why I ruled it out:** `raisinkanes[.]com` is a lookalike of `raisingcanes.com`, it redirected to other attacker domains, and on `RQJQ-MACHINE` `cobaltstrike.exe` ran one second after the downloaded docx was opened.
* **Possible explanation:** One attacker did everything. **Why it's unlikely (not fully ruled out):** The two stages use separate infrastructure (`93.238.22[.]122`, `93.238.22[.]123`, `nothing-to-see-here[.]net` vs. `203.0.113[.]1`, `203.0.113[.]2`, `secure-health-access[.]com`, `emr-help[.]net`) and different access methods (malware vs. a direct login with stolen credentials). The story's account of an access broker selling to a ransomware group fits that split *(story)*, but one actor could still switch infrastructure, so the logs alone can't prove two groups.
* **Possible explanation:** The encoded PowerShell commands (`powershell -Nop -ExecutionPolicy bypass -enc ...`) on `AMFB-MACHINE` and `RQJQ-MACHINE` were attacker activity. **Why I ruled it out:** The Base64 decodes to an `Invoke-WmiMethod` call on `CCM_SoftwareUpdatesManager` / `InstallUpdates`, a routine software update task, not an attacker tool.

## 6. Kill Chain and MITRE ATT\&CK mapping

| Stage                 | What the attacker did                                                                                                                                                                                                              | MITRE ATT\&CK                                                                                                                                                                                                                                                                 | Evidence                                                                                                       |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------- |
| Reconnaissance        | No evidence of research before either stage got in. Stage 2's 37 website requests start at the same second as the May 20 login, so they are research after access.                                                                | T1594 Search Victim-Owned Websites (after access)                                                                                                                                                                                                                             | InboundNetworkEvents from `203.0.113[.]1` and `203.0.113[.]2`; AuthenticationEvents 2024-05-20 00:00:00        |
| Weaponization         | Stage 1 built a lookalike restaurant domain, redirect domains, a malicious docx, and a C2 server. Stage 2 prepared ransomware and a patient data export tool.                                                                     | T1583.001 Acquire Infrastructure: Domains; T1588.002 Obtain Capabilities: Tool                                                                                                                                                                                               | `raisinkanes[.]com`, `Raisin_Kane_Promo_Offer.docx`, `cobaltstrike.exe`, `lockbyte_ransomer.exe`, `patient_data_exporter.exe` |
| Delivery              | Stage 1: a fake sponsored ad *(story)* for a lookalike restaurant domain. Stage 2: logged in with Anthony's purchased credentials *(story: purchased)*.                                                                            | T1583.008 Acquire Infrastructure: Malvertising; T1078 Valid Accounts                                                                                                                                                                                                         | OutboundNetworkEvents to `raisinkanes[.]com` (2024-05-01 to 2024-05-14); AuthenticationEvents 2024-05-20       |
| Exploitation          | Staff opened the docx, which launched Cobalt Strike. I saw no software exploit.                                                                                                                                                  | T1204.002 User Execution: Malicious File                                                                                                                                                                                                                                      | ProcessEvents: Word → `cobaltstrike.exe` on `RQJQ-MACHINE`, 2024-05-01 09:57:16                                |
| Installation          | Cobalt Strike ran on `RQJQ-MACHINE` (May 1) and `AMFB-MACHINE` (May 14), which the story says was reached by lateral movement *(story)*. Stage 2 brought the ransomware and the export tool onto `AMFB-MACHINE`.                | T1105 Ingress Tool Transfer                                                                                                                                                                                                                                                   | ProcessEvents; OutboundNetworkEvents 2024-06-17 14:22:29                                                       |
| Command & Control     | Cobalt Strike connected to `93.238.22[.]122` and `93.238.22[.]123` on port 50050. The attackers ran discovery commands on both hosts and a network scan from Anthony's.                                                         | T1571 Non-Standard Port; T1082 System Information Discovery (`systeminfo`); T1016 System Network Configuration Discovery (`ipconfig`); T1049 System Network Connections Discovery (`netstat`); T1087.001 Account Discovery: Local Account (`net user`); T1069.001 Permission Groups Discovery: Local Groups (`net localgroup administrators`); T1018 Remote System Discovery (`net view`); T1046 Network Service Discovery (`advanced-ip-scanner.exe`) | ProcessEvents 2024-05-01 to 2024-05-16                                                                          |
| Actions on Objectives | Stage 1 stole network diagrams and credentials. Stage 2 stole patient records, deleted the evidence, spread ransomware through a file share, and encrypted 6,420 files.                                                          | T1552.001 Unsecured Credentials: Credentials in Files; T1039 Data from Network Shared Drive; T1560 Archive Collected Data; T1048 Exfiltration Over Alternative Protocol; T1070.004 Indicator Removal: File Deletion; T1570 Lateral Tool Transfer; T1486 Data Encrypted for Impact | ProcessEvents 2024-05-16 and 2024-06-17; FileCreationEvents `.encrypted` 2024-06-17 14:49:30                   |

## 7. Recommendations

| Action                                                                                                                                                                               | Why (evidence)                                                                                                                                                                       | Urgency   |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------- |
| **Contain:** Isolate and reimage `RQJQ-MACHINE` and `AMFB-MACHINE`.                                                                                                                 | Both ran Cobalt Strike, and `AMFB-MACHINE` was also used for the ransomware and the data theft.                                                                                     | Now       |
| **Scope:** Find every host that ran Cobalt Strike or opened the lure, and isolate them too.                                                                                         | 24 internal IPs reached `raisinkanes[.]com`, but the module followed two machines. See section 8, question 2.                                                                       | Now       |
| **Contain:** Delete `spread_ransomware.exe` from `\\jojos-hospital.org\shared` and lock down write access to the share.                                                            | The ransomware was staged there on 2024-06-17 13:35:12, shortly before 321 hosts were encrypted.                                                                                    | Now       |
| **Contain:** Disable Anthony Davis's sessions, then reset his password and every credential in `credentials.txt`.                                                                 | The credentials file was uploaded to the first attacker on May 16, and his account was used from `203.0.113[.]1` four days later.                                                  | Now       |
| **Contain:** Block `93.238.22[.]122`, `93.238.22[.]123`, `203.0.113[.]1`, `203.0.113[.]2`, `raisinkanes[.]com`, `nothing-to-see-here[.]net`, `totally-legit-domain[.]com`, `secure-health-access[.]com`, and `emr-help[.]net`, and add the lure docx hash to endpoint blocklists. | Infrastructure and tools for both stages: lure, C2, research, and both exfiltration sites.                                                                       | Now       |
| **Recover:** Restore encrypted files from backups.                                                                                                                                  | 6,420 files on 321 hosts were encrypted.                                                                                                                                            | Now       |
| **Recover:** Brief legal and compliance on the patient data exposure, notify affected patients, and warn them not to pay the extortion emails.                                     | Three archives of patient records were uploaded to `secure-health-access[.]com`. Patients were told to pay $10,000 or have their records and SSNs released *(story)*.             | Now       |
| **Detect:** Alert on Office applications launching executables from user folders or `C:\ProgramData`, and on outbound connections to port 50050.                                  | `cobaltstrike.exe` ran right after Word opened the docx and used port 50050.                                                                                                       | This week |
| **Detect:** Alert on `curl` uploads from workstations and on executables written to file shares.                                                                                   | Both stages exfiltrated with `curl`, and the ransomware spread through `\\jojos-hospital.org\shared`.                                                                              | This week |
| **Prevent:** Require MFA for logins from outside the network.                                                                                                                      | A stolen password alone let the second group log in from `203.0.113[.]1`.                                                                                                         | This week |
| **Prevent:** Stop storing passwords in plain text files.                                                                                                                           | `credentials.txt` gave the attackers usernames and passwords.                                                                                                                       | Later     |
| **Prevent:** Filter lookalike domains and malicious ads at DNS, and train staff on sponsored-result scams.                                                                        | 24 internal IPs reached `raisinkanes[.]com` from a sponsored search result *(story)*.                                                                                               | Later     |

## 8. Open questions

These are questions the module's investigation doesn't answer. Each one lists what I would run or request next.

| # | Question                                                                          | Why it matters                                                                                                                                                                   | How I would answer it                                                                                                                                                                                    |
| :- | :-------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | How did the ransomware run on 321 hosts at once, and why is the ransom note on only one? | The copy to the share is logged, but not how each machine ran it. Without that, the spreading method can't be blocked.                                                          | `ProcessEvents` on all hosts between 14:40 and 14:50 on 2024-06-17, looking for `lockbyte`, `spread_ransomware`, the `\\jojos-hospital.org\shared` path, or a remote execution tool.                    |
| 2 | How many hosts did the first stage compromise?                                    | 24 internal IPs reached the fake ad, but only two machines were investigated. Any other Cobalt Strike host is still an open door.                                               | `ProcessEvents \| where process_commandline has "cobaltstrike" \| summarize first_seen = min(timestamp) by hostname`, and `FileCreationEvents \| where filename has "Raisin_Kane"` by hostname.        |
| 3 | How did Cobalt Strike get onto Anthony's machine?                                 | The story says lateral movement. If Anthony clicked the ad himself, the fix is different: the lure, not internal movement.                                                       | `OutboundNetworkEvents` from Anthony's IP for `raisinkanes`, and `ProcessEvents` on `AMFB-MACHINE` for Word opening a `Raisin_Kane` file before 2024-05-14 12:24:45.                                  |
| 4 | Where did `lockbyte_ransomer.exe` come from?                                      | The download source is more attacker infrastructure to block.                                                                                                                  | `FileCreationEvents \| where filename has "lockbyte"`, then `OutboundNetworkEvents` from Anthony's IP in the minutes before the copy at 13:35:12.                                                       |
| 5 | What did the second group do between the May 20 login and June 17?               | Four weeks of access with only website research in this investigation. They may have touched other machines or accounts.                                                       | `ProcessEvents \| where hostname == "AMFB-MACHINE"` from 2024-05-20 to 2024-06-17, and `AuthenticationEvents` for any other logins from `203.0.113.1` or `203.0.113.2`.                                |
| 6 | Is the first stage's Cobalt Strike access still active, and why do the two hosts use different C2 IPs? | Nothing shows Cobalt Strike was removed, and the neighboring IP `93.238.22[.]123` may belong to a larger range of attacker servers.                                            | `ProcessEvents` and `OutboundNetworkEvents` for `cobaltstrike`, `93.238.22.122`, and `93.238.22.123` after 2024-05-16, plus `PassiveDns` for both IPs.                                                 |
| 7 | Did `credentials.txt` contain Anthony's password?                                 | The story says his credentials were bought on the dark web. If they came from this file, every account in it is exposed too.                                                   | The contents of `credentials.txt` from a backup or forensic image of `AMFB-MACHINE`.                                                                                                                     |

## 9. Lessons learned

* **Start from the rarest artifact.** 321 hosts were encrypted, but the ransom note was on only one. That one host held the whole second stage: the ransomware copy, the export tool, and the upload domain.
* **Pivot through infrastructure.** One upload domain led through passive DNS to two IPs, a second domain, 37 website requests, and the login that gave the second group access.
* **Ransomware is often the last chapter, not the first.** The encryption on June 17 was built on an ad click on May 1 and stolen credentials on May 16.
* **Separate actors by infrastructure and access method.** Different C2 servers, different exfiltration sites, and malware vs. a password login pointed to two stages, which matches the story's access broker and ransomware group.
* **Decode before you judge.** The encoded PowerShell looked malicious, but decoding the Base64 showed a routine software update task, so I left it out of the attack chain.
* **Count mismatches are leads.** 24 IPs reached the fake ad, but the investigation followed two machines. That gap is the first next step, not an assumption.
