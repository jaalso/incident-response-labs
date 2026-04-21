🚨Incident-response-labs
<br>forensic investigation · attack simulation · IR reporting · DFIR · Windows forensics | SCI

All labs conducted in isolated VirtualBox environments or on authorised external targets.
No unauthorised systems were accessed. All work complies with Swiss law and ethical hacking standards.

---

## 📁 Labs

### Incident Response Lab — SMB Brute Force Attack & Windows Forensics
Simulated an SMB brute-force attack from Kali Linux against a Windows 10 target, then switch to analyst mode and investigate the attack using Windows forensic artifacts — proving execution,
identifying the attack timeline, and documenting findings in IR report format.
<br>**Tools:** CrackMapExec · Hydra · PECmd · AmcacheParser · AppCompatCacheParser · EvtxECmd · EZ Tools Suite
<br>Target: Target: WIN10TEST $HOSTNAME — SMB port 445

This lab demonstrates the complete SOC analyst workflow:
Attack simulation (Kali) → Artifact collection (WIN10TEST) → Forensic parsing (EZ Tools) → IR report


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

- ✅ Key Finding — Brute Force Attack Reconstructed from Logs
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

<br><img width="659" height="357" alt="image" src="https://github.com/user-attachments/assets/62a8110c-ac69-43bd-8239-25ee180a1d53" />
📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/smb-pentest-forensics-report_protected.pdf)**  
> 🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso)

⚖️ Legal & Ethical Notice
All activities performed in an isolated VirtualBox lab environment against VMs owned by the student.
CrackMapExec and Hydra were used against WIN10TEST — a personal test machine with no external
connectivity. This report is submitted as part of the Swiss Cyber Institute IR Playbooks curriculum.
No unauthorised systems were accessed. All work complies with Swiss law and ethical hacking standards.
