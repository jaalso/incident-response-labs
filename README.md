> 🚨Incident-response-labs

All labs conducted in isolated VirtualBox environments or on authorised external targets. 
<br>No unauthorised systems were accessed. All work complies with Swiss law and ethical hacking standards.

---

## 📁 Labs

Forensic investigation · attack simulation · IR reporting · DFIR · Windows forensics

| # | Lab | Tools | Status |
|---|---|---|---|
| 01 | SMB Brute Force Attack & Windows Forensics | CrackMapExec · EZ Tools · EvtxECmd | ✅ Complete |
| 02 | Wayne Corp IR Simulation — Windows DFIR & SOC | KAPE · EZ Tools · impacket · Timeline Explorer | ✅ Complete |

---

### 01 SMB Brute Force Attack & Windows Forensics
Simulated an SMB brute-force attack from Kali Linux against a Windows 10 target, then switch to analyst mode and investigate the attack using Windows forensic artifacts — proving execution,
identifying the attack timeline, and documenting findings in IR report format.
<br>**Tools:** CrackMapExec · Hydra · PECmd · AmcacheParser · AppCompatCacheParser · EvtxECmd · EZ Tools Suite
<br>Target: Target: WIN10TEST $HOSTNAME — SMB port 445

This lab demonstrates the complete SOC analyst workflow:
<br>Attack simulation (Kali) → Artifact collection (WIN10TEST) → Forensic parsing (EZ Tools) → IR report


Attack Chain
| Technique | ID | Tool | Description |
|---|---|---|---|
| Active Scanning | T1595.002 | CrackMapExec | Initial SMB probe — anonymous enum |
| Valid Accounts | T1078.003 | CrackMapExec | Targeting known local account $ATTACKERHOSTNAME |
| Brute Force | T1110.001 | CrackMapExec · Hydra | Rapid wordlist-based password guessing |
| [DEFENSE] | — | Windows Policy | Account lockout — attack blocked (Event 4740) |

Switched to analyst mode on WIN10TEST. Used Eric Zimmerman's EZ Tools suite to parse four Windows forensic artifacts — then correlated them to reconstruct the attack timeline.

Artifact Overview
| Artifact | Tool | Location | Key Field | Proves |
|---|---|---|---|---|
| Prefetch | PECmd | `C:\Windows\Prefetch\*.pf` | Run count + last 8 timestamps | Program ran |
| AmCache | AmcacheParser | `C:\Windows\AppCompat\Programs\Amcache.hve` | SHA1 hash | Ran + hash for VT |
| Shimcache | AppCompatCacheParser | SYSTEM registry hive | File path + LastModified | Windows saw file |
| Event Logs | EvtxECmd | `C:\Windows\System32\winevt\Logs\` | Event ID + timestamp | Auth + persistence |

<br>✅ Key Finding — Brute Force Attack Reconstructed from Logs
Discovery: Querying Event 4625 (Failed Logon)

> Seeing `"password is wrong"` on $ATTACKERHOSTNAME = the attacker enumerated valid usernames first,
> then targeted this specific account for brute force.

| Event ID | Count | Assessment |
|---|---|---|
| 4624 | 1,024 | Successful logons — normal |
| 4625 | 14,657 | ⚠️ Failed logons — investigate |
| 4648 | 70 | Logon with explicit credentials |
| 4672 | 922 | Admin logons (PowerShell as admin) |
| 4698 | 1 | ⚠️ Scheduled task created — investigate |
| 4740 | 1 | 🚨 Account lockout — the brute force event |
| 4907 | 7,741 | Audit policy changes — normal Windows |

🧰 Tools Used
| Category | Tool | Purpose |
|---|---|---|
| Attack Simulation | CrackMapExec · Hydra | SMB brute force from Kali Linux |
| Prefetch Forensics | PECmd (EZ Tools) | Parse `.pf` files → execution timeline |
| AmCache Forensics | AmcacheParser (EZ Tools) | Parse registry hive → SHA1 hashes |
| Shimcache Forensics | AppCompatCacheParser (EZ Tools) | Parse SYSTEM hive → file presence |
| Event Log Forensics | EvtxECmd (EZ Tools) | Parse Security.evtx → attack timeline |
| Threat Intel | VirusTotal · ThreatFox | SHA1 hash lookups from AmCache |
| Platform | Kali Linux · Windows 10 · VirtualBox | Isolated lab environment |

**credential Brute Force**
<br><img width="925" height="460" alt="image" src="https://github.com/user-attachments/assets/b19625e5-7f3a-4c9d-b91e-e659b8690645" />

**accessing Admin account**
<br><img width="868" height="147" alt="image" src="https://github.com/user-attachments/assets/f00976b0-b150-4fb8-bb21-6753e09f6876" />

**Using Time Explorer to prefecth**
<img width="1055" height="424" alt="image" src="https://github.com/user-attachments/assets/95890e40-5974-4d88-a20c-a639b3471ef1" />

**reading the EvtxECmd output**
<br><img width="1068" height="163" alt="image" src="https://github.com/user-attachments/assets/820328be-2e27-4ab1-bccc-ee25ccca6778" />
**event 4740 burst**
<br><img width="1039" height="242" alt="image" src="https://github.com/user-attachments/assets/c4592c60-44c1-48ac-b413-07ea8cf53cb1" />

---

### 02 IR Simulation — Windows DFIR & SOC Analyst Engagement
Lab type: Full red team → blue team → knowledge consolidation — three operational phases. End-to-end Windows DFIR and SOC analyst engagement spanning offensive operations, defender-side forensic reconstruction, and production of analyst reference materials. Every finding maps to specific MITRE ATT&CK techniques and Windows Event IDs using real engagement data — no synthetic or copied material.
<br>**Tools:** KAPE · EZ Tools Suite · Timeline Explorer · Registry Explorer · impacket-psexec · CrackMapExec · Wazuh
<br>Environment:Kali Linux ($IPATTACKER) → Windows 10 Enterprise WIN10TEST ($IPVICTIM)

Key milestones:
- ✅ Discovered attacker-driven Domain Policy modification (Event 4739) that disabled account lockout mid-attack
- ✅ Collected and parsed 3,303 forensic artifacts (1.6 GB) in 13 minutes using KAPE
- ✅ Reconstructed impacket-psexec lateral movement across five independent forensic sources
- ✅ Identified impacket fingerprint — blank WorkstationName field in Event 4624 distinguishes it from legitimate PsExec


🔴 Phase 1 — Offensive Operations
| Technique | ID | Tool | Evidence |
|---|---|---|---|
| Network Service Discovery | T1046 | nmap · netdiscover | Port 445 open on WIN10TEST |
| Brute Force: Password Guessing | T1110.001 | CrackMapExec · Hydra | 4 bursts · 14,657 Event 4625 entries |
| Impair Defenses: Disable Tools | T1562.001 | net accounts | Event 4739 — LockoutThreshold=0 |
| Valid Accounts: Local Accounts | T1078.003 | Stolen credentials | Event 4624 Type 3 · blank WorkstationName |
| System Services: Service Execution | T1569.002 | impacket-psexec | qSqxGVfD.exe · oTGtPTeq.exe |
| Create or Modify Service | T1543.003 | impacket-psexec | Event 7045 — services ysZf · VSGy |
| Remote Services: SMB Admin Shares | T1021.002 | impacket-psexec | Lateral movement via ADMIN$ |
| Indicator Removal: Clear Event Logs | T1070.001 | wevtutil | Event 1102 — Security log cleared |

🔵 Phase 2 — Defender Investigation
| Tool | Source | Records | Forensic Value |
|---|---|---|---|
| PECmd | `C:\Windows\Prefetch\*.pf` | 141 files | Execution evidence + 8 historical timestamps |
| AmcacheParser | Amcache.hve | 401 entries | SHA1 hashes of every executable seen |
| AppCompatCacheParser | SYSTEM hive | 167 entries | LRU-ordered binary list |
| EvtxECmd | `winevt\Logs\*.evtx` | 30,924 events | Unified parsed event log set |
| RECmd | Multiple hives | DFIRBatch | Persistence key checklist |
| MFTECmd | $MFT | 200K+ entries | File system metadata |


🚨 Key Finding — Event 4739 (Defense Evasion Discovered)
Filtering Timeline Explorer on EventId 4625 revealed 14,657 failed logons.
The question: why did Burst 4 (41 failures) produce no Event 4740 lockout?
Time-range filter between Burst 3 end (13:35:53) and Burst 4 start (13:56:18)
surfaced a single anomalous event — the centrepiece finding of the engagement:
```bash
# Event 4739 — Domain Policy was changed
# TimeCreated:      2026-03-25 13:54:22
# SubjectUserName:  bornia02
# LockoutThreshold: 0    ← DISABLED by attacker
```

KAPE Collection Results:
| KAPE Metric | Result |
|---|---|
| Files collected | 3,303 (1.6 GB) |
| Files deduplicated | 362 |
| Parsing processors run | 18 |
| Output CSVs produced | 30 (305 MB) |
| Total runtime | **13 minutes** |

📚Phase 3 — Knowledge Consolidation:
| Deliverable | Content |
|---|---|
| SMB Pentest + Forensics Report | 24 pages · 4-burst brute force · Event 4739 finding · EZ Tools walkthrough |
| PsExec Lateral Movement Report | 17 pages · 5-source evidence chain · impacket fingerprint |
| SOC Tools Cheat Sheet | 22 tools · MITRE ATT&CK quick-reference · Windows Event ID reference |
| Interactive Drill Widgets (×9) | EvtxECmd · Splunk/SPL · MITRE mapping · Mock Alert Triage · Timeline Explorer · VT+CyberChef · KAPE · Registry Explorer · LOLBAS/PsExec |

📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/portfolio-lab-engagement-report_protected.pdf)**  
> 🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso)


---

## 🧰 Tools Used
| Category | Tools |
|---|---|
| Attack Simulation | CrackMapExec · Hydra · impacket-psexec |
| Collection | KAPE 1.3.0.2 (KapeTriage + !EZParser) |
| Forensic Parsing | PECmd · AmcacheParser · AppCompatCacheParser · EvtxECmd · MFTECmd · RECmd |
| Forensic GUI | Timeline Explorer · Registry Explorer |
| SIEM | Wazuh v4.14.3 · Splunk Enterprise (BOTS v1) |
| Threat Intel | VirusTotal · URLscan.io · Any.RUN · CyberChef |
| Platform | Kali Linux · Windows 10 Enterprise · VirtualBox |

---

## ⚖️ Legal & Ethical Notice

All activities performed in an isolated VirtualBox lab environment against VMs owned by the student.
CrackMapExec and Hydra were used against WIN10TEST — a personal test machine with no external
connectivity. This report is submitted as part of the Swiss Cyber Institute IR Playbooks curriculum.
No unauthorised systems were accessed. All work complies with Swiss law and ethical hacking standards.
