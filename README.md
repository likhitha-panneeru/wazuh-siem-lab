Build Your Own SIEM with Wazuh

A hands-on cybersecurity portfolio project demonstrating real SOC analyst skills — from SIEM deployment to custom detection rules and alert investigation.

# 1 Project Overview

Built a fully functional Wazuh SIEM lab that collects logs from 3 sources, runs 5 custom detection rules, and investigates security alerts end-to-end — all on a single machine using WSL2.

**Role:** SOC Analyst / Blue Team Analyst  
**Tools:** Wazuh 4.14.7, Windows 11, Ubuntu (WSL2), Apache2, Sysmon  
**MITRE ATT&CK Techniques Detected:** T1110 (Brute Force), T1059.001 (PowerShell), T1136.001 (Create Account), T1595 (Active Scanning)

---

# 2 Lab Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    HOST MACHINE                         │
│                  Windows 11 (8GB RAM)                   │
│                                                         │
│  ┌──────────────────┐     ┌───────────────────────────┐ │
│  │  Wazuh Agent      │     │  WSL2 Ubuntu             │ │
│  │  (Windows Logs)   │────▶  Wazuh Server (Manager)   
│  │  + Sysmon         │     │  + Indexer + Dashboard     │  
│  │                   │     │  + Apache2 Web Server      │  
│  └──────────────────┘     └───────────────────────────┘ │
│         │                          │                    │
│    Log Source 1              Log Source 2 & 3           │
│    (Windows Security)   (Linux auth + Apache access)    │
└─────────────────────────────────────────────────────────┘
                          │
                    Wazuh Dashboard
                 https://172.x.x.x
```

#  3 Log Sources Connected

| # | Source | Type | What It Collects |
|---|--------|------|------------------|
| 1 | Windows 11 Host | Wazuh Agent | Security events (4625, 4624), PowerShell, process creation |
| 2 | Ubuntu (WSL2) | Manager Host | auth.log, syslog, system events |
| 3 | Apache2 Web Server | Log File Monitoring | access.log, error.log (HTTP requests, 404s) |

---

# 5 Custom Detection Rules

All rules are in `/var/ossec/etc/rules/local_rules.xml`

| Rule ID | Name | Level | Trigger | MITRE ATT&CK |
|---------|------|-------|---------|--------------|
| 100002 | Multiple Windows Logon Failures | 10 | 5+ failed logons in 120 seconds | T1110 - Brute Force |
| 100003 | Suspicious PowerShell Execution | 12 | Invoke-WebRequest, IEX, EncodedCommand, etc. | T1059.001 - PowerShell |
| 100004 | New User Account Created | 10 | Windows Event ID for account creation | T1136.001 - Local Account |
| 100005 | Repeated SSH Failed Logins | 10 | 5+ SSH failures in 120 seconds | T1110 - Brute Force |
| 100006 | Web 404 Burst (Web Scanning) | 8 | 10+ 404 errors in 60 seconds | T1595 - Active Scanning |

# Rule Validation Results

| Rule ID | Status | Evidence |
|---------|--------|----------|
| 100002 |  Fired | Triggered by simulated brute force via `net use` with wrong credentials |
| 100003 |  Written | Rule deployed, awaiting PowerShell script block logging configuration |
| 100004 |  Fired | Triggered when test user accounts were created |
| 100005 |  Written | Rule deployed, SSH decoder needed tuning for log format |
| 100006 |  Fired | Triggered by rapid 404 requests simulating web scanning |

---

#  Sample Alert Investigation

# Alert: Multiple Windows Logon Failures (Rule 100002)

**Triage Note:**

```
ALERT:      Multiple Windows logon failures detected - possible brute force attack
RULE ID:    100002 (Custom Rule)
SEVERITY:   Level 10 (High)
TIMESTAMP:  Sep 11, 2026 @ 18:06:00 UTC
HOST:       DESKTOP-WIN11 (Agent 001)
ATT&CK:    T1110 - Brute Force / Credential Access

EVIDENCE OBSERVED:
- Windows Event ID 4625 (Logon Failure) fired 5+ times within 120 seconds
- Target username: "hacker" — not a legitimate account on this system
- Source IP: 127.0.0.1 (lab simulation; in production would be attacker IP)
- Logon Type 3 (Network) — indicates remote authentication attempt
- Authentication Package: NTLM
- Failure Reason: Unknown user name or bad password (Status: 0xC000006D)
- Sub Status: 0xC0000064 (user does not exist)

VERDICT:    True Positive — brute force pattern confirmed
            Multiple rapid authentication failures against a non-existent account
            from the same source is consistent with credential guessing.

RECOMMENDED ACTIONS:
1. Block source IP at firewall (in production)
2. Check if any logon succeeded after the failures (Event ID 4624)
3. Review other accounts targeted from the same source
4. Enable account lockout policy if not already configured
5. Monitor for lateral movement from the source IP
```

# What I Learned

- How to deploy and configure a Wazuh SIEM from scratch
- Connecting multiple log sources (Windows, Linux, Web)
- Writing custom XML detection rules with frequency thresholds
- Mapping detections to MITRE ATT&CK techniques
- Investigating alerts and writing analyst triage notes
- Troubleshooting SIEM configuration issues (XML parsing, log collection)
- Understanding Windows Event IDs (4625, 4624) and what they mean

# How to Reproduce

# Prerequisites
- Windows 10/11 with WSL2 enabled
- Ubuntu on WSL2 (with systemd enabled)
- At least 8GB RAM

# Steps
1. Install Wazuh all-in-one on WSL Ubuntu:
   ```bash
   curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i
   ```
2. Install Wazuh agent on Windows host (use the command from Wazuh dashboard → Deploy New Agent)
3. Install Apache2 on WSL and configure log collection in `ossec.conf`
4. Add custom rules to `/var/ossec/etc/rules/local_rules.xml`
5. Generate test events and verify detections fire
6. Investigate alerts and write triage notes




