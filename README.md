🚨incident-response-labs
Incident Response labs — forensic investigation, attack simulation &amp; IR reporting | SCI

All labs conducted in isolated VirtualBox environments or on authorised external targets.
No unauthorised systems were accessed. All work complies with Swiss law and ethical hacking standards.

---

## 📁 Labs

### Incident Response Lab — SMB Brute Force Attack & Windows Forensics
Performed a full penetration test lifecycle: reconnaissance, scanning, vulnerability identification, exploitation, and post-exploitation. Documented findings in a structured report format.
<br>**Tools:** nmap · Metasploit · Hydra
<br>Target: Metasploitable 2 (192.168.56.XXX)

Attack Chain
<br>T1595.002  Active Scanning     → CrackMapExec initial SMB probe (anonymous enum)
<br>T1078.003  Valid Accounts      → Targeting known local account bornia02
<br>T1110.001  Brute Force         → Rapid wordlist-based password guessing
<br>[DEFENSE]  Account Lockout     → Windows policy blocked further attempts (Event 4740)

Switched to analyst mode on WIN10TEST. Used Eric Zimmerman's EZ Tools suite to parse four Windows forensic artifacts — then correlated them to reconstruct the attack timeline.

Artifact Overview
ArtifactToolLives atKey fieldProvesPrefetchPECmdC:\Windows\Prefetch\*.pfRun count + last 8 timestampsProgram ran (with timing)AmCacheAmcacheParserC:\Windows\AppCompat\Programs\Amcache.hveSHA1 hash + first executionProgram ran with hash for VTShimcacheAppCompatCacheParserSYSTEM registry hiveFile path + LastModified timeWindows saw the file (presence)Event LogsEvt

- ✅ Key Finding — Brute Force Attack Reconstructed from Logs
Discovery: Querying Event 4625 (Failed Logon)

Seeing "password is wrong" on bornia02 = the attacker enumerated valid usernames first,
then targeted this specific account for brute force.
Event 4625 Security Log Metrics (full 28-day log)
Event IDCountAssessment46241,024Successful logons — normal462514,657Failed logons — investigate464870Logon with explicit creds4672922Admin logons (PowerShell as admin)46981Scheduled task created — investigate47401Account lockout — the brute force event49077,741Audit policy changes — normal Windows

🧰 Tools Used
CategoryToolPurposeAttack SimulationCrackMapExec · HydraSMB brute force from Kali LinuxPrefetch ForensicsPECmd (EZ Tools)Parse .pf files → execution timelineAmCache ForensicsAmcacheParser (EZ Tools)Parse registry hive → SHA1 hashesShimcache ForensicsAppCompatCacheParser (EZ Tools)Parse SYSTEM hive → file presenceEvent Log ForensicsEvtxECmd (EZ Tools)Parse Security.evtx → attack timelineThreat IntelVirusTotal · ThreatFoxSHA1 hash lookups from AmCachePlatformKali Linux · Windows 10 · VirtualBoxIsolated lab environment

🔗 How the Artifacts Work Together
Analyst's question: "Did malware X run on this host?"

Step 1: Prefetch    → Did it execute? When? How many times?
                                    ↓
Step 2: AmCache     → What's the SHA1 hash? (even if binary deleted)
                                    ↓
Step 3: vt_lookup   → Is that hash known malware? (VirusTotal/ThreatFox)
                                    ↓
Step 4: Shimcache   → Did Windows see the file? (backup if Prefetch deleted)
                                    ↓
Step 5: EvtxECmd    → What authentication/persistence events surround it?
                                    ↓
Step 6: Timeline Explorer → All CSVs overlaid → single unified attack timeline
The attacker's dilemma: To hide from all three artifacts simultaneously:

Delete Prefetch .pf files ← easy (2 seconds)
Edit live locked AmCache hive ← hard (admin + registry expertise)
Flush 1024+ binaries through Shimcache ← extreme (leaves its own evidence)
Clear Security.evtx ← leaves Event 1102 (log cleared) in System log

No attacker cleans all four without leaving traces.

⚖️ Legal & Ethical Notice
All activities performed in an isolated VirtualBox lab environment against VMs owned by the student.
CrackMapExec and Hydra were used against WIN10TEST — a personal test machine with no external
connectivity. This report is submitted as part of the Swiss Cyber Institute IR Playbooks curriculum.
No unauthorised systems were accessed. All work complies with Swiss law and ethical hacking standards.
